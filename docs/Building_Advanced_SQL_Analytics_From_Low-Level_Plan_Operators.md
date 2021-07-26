# Low-Level Plan Operators

## 目标：解决什么问题  

### 查询语句中多个统计函数的执行效率问题

分析查询中，经常出现如sum，count，avg等诸如此类的统计计算，在SQL中主要体现如： group-by Agg/Agg， Window， GroupingSet此类。
目前业界的实现，对于不同的计算逻辑，往往会采用独立的算子来进行实现，如presto目前，通过HashAggregationOperator，WindowOperator，GroupIdsOperator等来完成上面算子的实现，
算子与算子间采用统一的交互方式，tuple by tuple （umbra），presto采用page by page。  

文中指出，这种方式的实现存在如下两个问题：  
- 代码逻辑未复用，如Sort-Agg和Window等均需要sort；    
- 数据未复用，算子间的状态需要多次物化；   

第一个问题我其实比较困惑，理论上系统抽象做的好一些的话，基础的数据结构算法逻辑统一抽象在公共实现中，应该不存在代码冗余，因为不同的算子里更多的是对数据属性的需求，以及访问的区别，而实际排序逻辑这些均会复用，如presto；

第二个问题，从整片文章的举例来看的话，更多的是算子结果的share；  

例如，groupingSet的例子，先计算最长的组合，再基于结果计算其他的，最后把所有结果union all；
另外一类是同时有sort和 hashAgg，数据可以被share同时计算两种；
还有一类就是ordAgg不同序的时候，可以多次share；

```
个人的一些理解，这里面的中间物化结果复用，其实有两类：  
1. 对于一次hash or sort，多个表达式的计算，这种本身是通过把数据属性需求一致的表达式放在一起计算，通过一次物化，这一类大部分系统应该都支持；
2. 就是算子结果的share； 这一类目前主要是类似CTE，在单个node内计划的cte，大部分系统依然还是树形的结构，一个operator node对应下游一个operator node；
```

#### 目前常见算子的物化代价

目前大部分SQL引擎的系统，无论是采用vectorized or compiled，所有计算逻辑均是采用标准的算子接口实现，算子间的交互均是采用统一的方式，要不tuple by tuple or vector by vector（mini-batch），这样带来的问题就是算子间需要重新根据自身的计算特点进行数据重新物化。

我们来看下：
- Sort，基于目前indirect sort算法 物化代价：本身数据无需拷贝，主要是sorted address；
- topN，heap实现，物化代价：重新构建heap；
- HashJoin，Multi-HashMap，物化代价：Multi-HashMap的索引构建，本身数据也是追加的方式； 
- HashAgg，HashMap，物化代价：重新构建一个HashMap，相比hashJoin，本身数据集会合并；
- Window， Sort实现，物化代价：Sorted Address构建；

目前Presto里面的实现，各个有状态的算子，主要还是indirect实现，数据单独追加存放（物化开销很低，不涉及内存拷贝），物化主要体现在构建索引上（indirect sort构建有序的索引），但是在输出的阶段，为了保证数据的属性，需要随机的访存，存在写入的物化代价。   
如sort，本身需要保证输出结果有序，则需要额外物化数据本身，如hashAgg，需要保证group中的key 和 state对齐，需要额外物化；

## 方案：具体怎么做的  

### 核心思路

1. 进一步对现有算子进行抽象，提出Low-Level Plan Operator；
2. 由Plan Tree => DAG，Producer to N-Consumer

#### Low-Level Plan Operator

核心思路在于对于sort，partition，merge等主要负责数据的物化，改变数据属性的逻辑，抽象为Transform；partition/sort/merge/combine/scan；  
对于实际负责基于物化结果数据进行计算的，抽象为Compute；Window/Hash-Agg/Order-Agg； 

![img](https://github.com/leoYY/papers/blob/main/img/low-level_plan_operators.png)  

算子间的交互不再局限于一种形式，同时存在tuple-by-tuple， 或者 物化后的buffer，如Sort的input是Buffer，可以减少输出的物化开销。  

可以看到，对于Order-Agg，Window等算子，不再内部进行数据的物化，完全依赖外部物化好的结果，input类型为buffer。  

```
对于low-level plan operator的抽象，从整个抽出来的transform主要还是应用于sort相关，这里面从他的例子中，从他的例子中更多看到的是 在相同partition后的数据情况下，sort间是有依赖的，可以复用具体的数据buffer；
```

#### Plan Tree => DAG

![img](https://github.com/leoYY/papers/blob/main/img/LOLEPOPS-DAG.png)   

整个Plan Tree => DAG的转变，从整个图中可以看到，自左向右，
首先SQL采用关系代数的方式表达， 然后对具体的计算表达式拆解成独立的节点。  
每个计算表达式由 ARG/KEY/ORD 三个维度表示。

例子中，对于median 需要采用sort-AGG实现，因此 ORD = a key = d（group by列）， arg = a(实际计算参数)；  

avg这里面做了拆分，拆分成了sum；count，对于一些更复杂的数据计算公式中，需要统计总数可以进行表达式级别的复用。   

另外就是ANY，这里面的ANY更多的是想进行去重，无实际含义，主要是用于算 Sum-Distinct的 转换成了group by d，c 后进行计算。  

![img](https://github.com/leoYY/papers/blob/main/img/LOLEPOP_DAG_algo.png)  

文中将整个算法过程也总结归纳成5步骤：  
1.  增加combine节点，进行多路的结果关联，相同组的数据合并；  
2.  根据表达式公式的特点，进行计算node的生成；  
3.  根据data properties demand，添加transform node进行数据buffer的准备处理；  
4.  处理DAG图的上下游；  
5.  DAG图的优化；

关于DAG图的优化方面，一方面对于一些不必要的计算优化，如有包含关系的sort key，以及多余的combine(groupingSets中)，进行图复杂度的简化rewrite；  

另外一方面，对于算子算法本身的优化，如indirect or inplace sort，sort策略等选择，不过本身选择的策略并没有详细讨论。    

```
这块我的理解， 如图上的例子，如果median(a), median(c) 的话，本身sort均是基于key + arg，那么对于第一次sort可以采用in-place sort，在复用结果的时候，尽可能保持cache友好。
```

#### 一些更复杂的例子

文中接下来给了5个更具体复杂一些的例子，

核心主要是：   

Eample 1的扩展groupingSet，通过首先计算最长keys，然后结果计算其他keys，计算结果复用，同时由于三个hashAgg的结果不存在join上的情况，combine同时可以优化成union all；  
Eample 2的更多是对于sort or hash agg方式的选择；  
Eample 3，比较特殊的在于进一步合并了topN算子，对于本身window计算物化后的有序数组，进一步sort 提供limit，__这里面我认为topN的算法，可以先进行分割，取topN后，则对N排序，会更快一些__   
Eample 4/5，更多是组合window 与 order-Agg来实现比较复杂的统计逻辑；  

最终给出一个API接口，可以比较简单的描述这种依赖关系的统计计算；  

```
def planMSSD ( arg , key , ord ):
    f = WindowFrame ( Rows , CurrentRow , Following (1))
    lead = plan ( LEAD , arg , key , ord , f )
    ssd = plan ( power ( sub ( lead , arg ) , 2))
    sum = plan ( SUM , ssd , key )
    cnt = plan ( COUNT , ssd , key )
    res = plan ( div ( sum , nullif ( sub ( cnt , 1) , 0)))
    return res
```

这里也可以看到，表达式核心由arg，key，ord来标识；

### 实现考虑

#### DAG in Producer-Consumer Pattern

umbra 典型的compiled系统，采用的producer/consumer的模型。
相比原有producer/consumer模式最大的区别主要体现在：
1. 结果复用下，parent node 会依次调用多个children的consumer；
2. 同时对于transform节点，input为buffer的情况下，不需要consumer；

话外：  
对于compiled模型下，可以看到join的过程就是forloop下lookup并继续进行下一步计算，这块跟vectorized思路是完全相反的。  

#### 内存结构 Tuple Buffer；

这里关于Tuple Buffer介绍主要考虑几点：  
1. 通过chunk list的方式，无法预估数据的情况下，减少reallocate开销；  
2. 虽然是列存系统，但是对于物化结果采用行存的方式，对于 in-place sort有比较好的效果；  

对于indirect sort，采用premultation vector，原理上与prefix-Sort有类似，不过文中考虑的更多是在于实际计算过程中的比较，因此会存全key的方式。这样带来了构建vector更大的开销，但是提升计算的cache友好，这部分倒没有具体介绍相关的模型考虑。  

最终通过iterator的模式，来屏蔽访问的具体内存结构。 

#### 算子的实现

Window的采用Segment Trees计算，对同一物化结果进行反复计算，具体这部分实现参考[Efficient Processing of Window Functions in Analytical SQL Queries](https://dl.acm.org/doi/abs/10.14778/2794367.2794375)   

另外对于HashAgg，会存在partial-Agg的逻辑，partial阶段会根据下游并发的分桶进行partial-agg，每个写一个chunk中，最后会concat整个chunk发给下一个线程进行计算。  
这里面主要涉及到 umbra的线程模型，从HashAgg的逻辑来看，会存在类似localExchange的方式进行线程间的通信，文中使用Shuffle来描述这个过程。  

对于partition算子，会存在显示的shuffle调用。对于需要in-place modified情况，会compact chunk lists，该做法尽可能减少后续算子的代价。    

在整片文章中，多次提到thread-local buffer，感觉更多是对于算子单线程内管理的内存buffer，在需要partition的时候，存在shuffle给其他线程的情况。  

## 效果：结果怎么样  

#### 效果方案

整个文章的case设计还是比较精细的，每个case均有明确的优化点来对比HyPer。

## 总结   

文章中比较新的地方就是对于物化结果的复用，通过把实际物化的算子单独抽象成Transform，而计算算子直接复用物化结果进行计算。  
本文主要还是介绍了这套LOLEPOP的基础框架，简略的介绍了如何在umbra中的应用，其中一些细节的代价考虑，如in-place vs indirect， 这些方面没做过多介绍。  
给我更多的启发在于，对于类似groupjoin的方式类似，物化结构的复用，groupjoin本身也是通过对join 以及 group hash结构的复用类似的情况；

