# PMC数据 CPU Profile  
## 概述  
本文核心在于基于《Denis Bakhvalov - Performance Analysis and Tuning on Modern CPUs》这本书中的介绍，重新梳理相关知识。整体围绕一个问题，如何基于PMU做CPU性能分析。  
通过把这本书中介绍的信息以自己的理解重新做一遍解构。  
后续文章逐个以问题出发，来逐渐把PMC数据的CPU profile相关技术介绍出来。  

### 相关名词  

Performance Monitoring Unit 可以理解为CPU中负责性能检测的组件，如Branch Predictor Unit负责分之预测一样。  
Performance Monitoring Counter基于计数触发分析性能事件的方法。  
Performance Counter，48bit长度，属于Model Specific Registers（MSR），不同CPU模型会有区别。  
PEBS Performance event-based sampling， 基于事件触发的分析性能方法。  
LBR Last branch recording，记录分支指令机制。  

### Counter vs Sampling
围绕Performance Profile，主要分为Counter 和 Sampling两种方式。  
Counter 通过配置PMC记录指定HW Event，读取指定时间范围内的Event发生次数。  

Sampling则是通过EBS（event-base sampling）出发PMI（performance monitoring interrupt）记录IP寄存器的指令地址。

这里我理解Counter主要类似Trace，每一次Event都会被记录下来，而sampling类似之前我们profile frame的通过多次采样来分析主要瓶颈。 

书中关于这部分主要的区别在于counter用于分区程序负载特征，而sampling用于寻找hotspot。这里感觉负载特征的分析也主要是根据HW event热点情况来确定负载属于内存瓶颈，CPU瓶颈等，而sampling则是根据采集instruction 采样来确定代码瓶颈点。  

## 1. Event数据是如何采集的？
Counter 主要可以通过perf stat来记录。 整体采集过程相对简单，通过PMC归零，并运行程序后读取这段时间记录的PMC值的方式。  

Sampling 主要是通过perf record 来记录。

这里面会分为两种：  
1. PMC的Sampling则是通过PMC来记录cycles，设置N cycles后触发PMI，Handle PMI主要逻辑：  
  1.1 disable counting；  
  1.2 记录instruction；   
  1.3 重制counter； 

参考*[Interrupt Event-Base Sampling](https://easyperf.net/blog/2018/06/08/Advanced-profiling-topics-PEBS-and-LBR)*

2. PEBS如名字中event-based，在事件发生时通过独立CPU硬件单元记录发生事件的指令等信息记录到一段缓冲区中。 PEBS的缓冲区是有限缓冲区，当缓冲区满后会触发PMI来通知系统进行保存，同时缓冲期会随着新数据的写入覆盖老数据，保持缓冲区都是最新的内容。

可以看到两种方式中，PEBS会先记录在通过PMI这样可以保证记录数据的准确，避免PMI处理过程中出现Skid的问题。  

## 2. 如何设置采集具体Events实际操作    

同样分为Counter 和 Sampling 两部分来看：  
1. Counter部分，从上面采集原理我们知道主要是借助PMC来获取记录时间计数的寄存器值。
```
   perf stat -e event...
```  
2. Sampling部分
```
   perf record -e event...
```
对于perf record 默认不采用PEBS机制， 如果需要使用PEBS则需要类似:p, :pp的方式指定event。  

参考 *[man perf-list](https://man7.org/linux/man-pages/man1/perf-list.1.html#EVENT_MODIFIERS)*

### 如何指定Event
可以理解perf list中预定了一部分事件，对于这种事件，我们可以方便的使用事件名来进行指定
如
```
perf stat -e cache-miss...
```
但是对于某些情况perf中并未预先定义事件，这个时候CPU本身支持的情况下，就需要通过事件码与掩码的方式来指定监听事件，
如
```
   perf stat -e cpu/event=0x3c,umask=0x0/,cpu/event=0xc5,umask=0x0/ ...
```
其中事件码与掩码主要是表示事件码描述一类事件，而掩码具体描述该类事件中特定事件，如L2缓存读取事件与写入事件就属于相同事件码与不同掩码。  

## 3. 分析方法 ---- TMA
// TODO
# 再说Branch
// TODO LBR，BTS

# intel PT
// TODO
