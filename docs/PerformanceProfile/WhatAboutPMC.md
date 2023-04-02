# PMC数据 CPU Profile  
## 概述  
本文核心在于基于《Denis Bakhvalov - Performance Analysis and Tuning on Modern CPUs》这本书中的介绍，重新梳理相关知识。整体围绕一个问题，如何基于PMC数据做CPU性能分析。  
通过把这本书中介绍的信息以自己的理解重新做一遍解构。  
后续文章逐个以问题出发，来逐渐把PMC数据的CPU profile相关技术介绍出来。  

Performance Monitoring Unit 可以理解为CPU中的组件，如Branch Predictor Unit负责分之预测。
Performance Monitoring Counter则是负责实际存储Performance Events数据的寄存器，类似的还有LBR，PEBS。
Performance Counter，48bit长度，属于Model Specific Registers（MSR），不同CPU模型会有区别。

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

参考*Interrupt Event-Base Sampling[https://easyperf.net/blog/2018/06/08/Advanced-profiling-topics-PEBS-and-LBR]*

2. PEBS如名字中event-based，在事件发生时通过独立CPU硬件单元记录发生事件的指令等信息记录到一段缓冲区中。

## 2. 如何设置PMC采集具体Events

## 3. PEBS
