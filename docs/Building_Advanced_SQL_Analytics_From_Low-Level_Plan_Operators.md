# Low-Level Plan Operators

## 目标：解决什么问题  

### 查询语句中多个统计函数的执行效率问题

分析查询中，经常出现如sum，count，avg等诸如此类的统计计算，在SQL中主要体现如： group-by Agg/Agg， Window， GroupingSet此类。
目前业界的实现，对于不同的计算逻辑，往往会采用独立的算子来进行实现，如presto目前，通过HashAggregationOperator，WindowOperator，GroupIdsOperator等来完成上面算子的实现，
算子与算子间采用统一的交互方式，tuple by tuple （umbra），presto采用page by page。  

文中指出，这种方式的实现存在如下两个问题：  
- 代码逻辑未复用，如Sort-Agg和Window等均需要sort；    
- 数据未复用，算子间的状态需要多次物化；   

第一个问题，起初我还比较疑惑，因为本身presto目前类似如PagesIndex这种数据结构依然是可以在sort，window等算子内复用，理论上不存在这类问题，但是如果从论文的背景系统 Umbra来看的话，应该是在于code-gen的code block，以算子为粒度的话，如果需要排序的话，compile会展开对应的逻辑，导致代码块无法复用。  
第二个问题，算子物化结果的复用。这个我理解本身含有两层含义，   

1. 物化的数据结构可以被多次使用，典型 如 相同group-by key下的多个Agg functions，本身是采用了相同的HashMap结构进行合并到一个算子计算，这类复用大部分系统都有做到；  
2. 物化数据后的结果复用，从文中的例子来看，更多是物化数据的属性（如 order，partitioned）以及数据本身的share，如物化属性方面：如order属性下，避免额外的hash-based agg；

#### 目前常见算子的物化代价

目前大部分SQL引擎的系统，无论是采用vectorized or compiled，所有计算逻辑均是采用标准的算子接口实现，算子间的交互均是采用统一的方式，要不tuple by tuple or vector by vector（mini-batch），这样带来的问题就是算子间需要重新根据自身的计算特点进行数据重新物化。

我们来看下：
- Sort，基于目前indirect sort算法 物化代价：本身数据无需拷贝，主要是sorted address；
- topN，heap实现，物化代价：重新构建heap；
- HashJoin，Multi-HashMap，物化代价：Multi-HashMap的索引构建，本身数据也是追加的方式； 
- HashAgg，HashMap，物化代价：重新构建一个HashMap，相比hashJoin，本身数据集会合并；
- Window， Sort实现，物化代价：Sorted Address构建；

以上主要是算子本身input需要做的物化， 另外一方面的物化开销，主要就是本身在输出过程中。


## 方案：具体怎么做的  

### 核心思路

1. 进一步对现有算子进行抽象，提出Low-Level Plan Operator；
2. 由Plan Tree => DAG，Producer to N-Consumer

#### Low-Level Plan Operator

核心思路在于对于sort，partition，merge等主要负责数据的物化，改变数据属性的逻辑，抽象为Transform；partition/sort/merge/combine/scan；  
对于实际负责基于物化结果数据进行计算的，抽象为Compute；Window/Hash-Agg/Order-Agg；

并且算子间的交互不再局限于一种形式，同时存在tuple-by-tuple， 或者 物化后的buffer；  



#### Plan Tree => DAG



### 核心难点

### 实现考虑

## 效果：结果怎么样  

## 总结   

