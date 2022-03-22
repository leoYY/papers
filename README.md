# papers
use to keep some papers 

follow  
[Thomas Neumann](https://scholar.google.de/citations?hl=zh_CN&user=xSDfDpsAAAAJ&view_op=list_works&sortby=pubdate)  
[Marcin Żukowski](https://scholar.google.com/citations?hl=zh-CN&user=F-TSpooAAAAJ&view_op=list_works&sortby=pubdate)  
[Andy Pavlo](https://scholar.google.com/citations?hl=zh-CN&user=u1UDm4wAAAAJ&view_op=list_works&sortby=pubdate)  
[Peter Boncz](https://scholar.google.com/citations?hl=zh-CN&user=DCIZE1kAAAAJ&view_op=list_works&sortby=pubdate)  
[Hannes Mühleisen](https://scholar.google.com/citations?hl=zh-CN&user=lBF-mDAAAAAJ)  
[Matei Zaharia](https://scholar.google.com/citations?user=I1EvjZsAAAAJ)  
[Stefan Manegold](https://scholar.google.com/citations?hl=en&user=BauIS1IAAAAJ&view_op=list_works&sortby=pubdate)  

[Optimizer:Alekh Jindal](https://scholar.google.com/citations?hl=zh-CN&user=f5vdXEQAAAAJ&view_op=list_works&sortby=pubdate)  


# Query Optimizer

## Cost model

[Orca: A Modular Query Optimizer Architecture for Big Data](https://15721.courses.cs.cmu.edu/spring2016/papers/p337-soliman.pdf)  
[优化器cost模型验证 Testing the accuracy of query optimizers](https://databasescience.files.wordpress.com/2013/01/taqo.pdf)  

## Estimation  
[Are We Ready For Learned Cardinality Estimation?](https://arxiv.org/pdf/2012.06743.pdf)   https://dataprep.ai/    

[优化器利用window解关联：WinMagic : Subquery Elimination Using Window Aggregation ](https://thelackthereof.org/docs/library/cs/database/Zuzarte,%20Calisto%20et%20al:%20Winmagic%20-%20Subquery%20Elimination%20Using%20Window%20Aggregation.pdf)  


# Query Execution   

## Adaptive Execution   

- [ ] [Adaptive and Big Data Scale Parallel Execution in Oracle](http://www.vldb.org/pvldb/vol6/p1102-bellamkonda.pdf)  

### Micro Adaptive  
- [ ] [loop-adaptive execution in weld](https://homepages.cwi.nl/~boncz/msc/2018-RichardGankema.pdf)  
- [ ] [micro adaptivity in vectorwise ](https://homepages.cwi.nl/~boncz/msc/2018-RichardGankema.pdf)  

## Query Execution model
- [ ] Balancing Vectorized Query Execution with Bandwidth-Optimized Storage
- [ ] [Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age](https://15721.courses.cs.cmu.edu/spring2016/papers/p743-leis.pdf)  

## Operator Algo

- [x] [Building Advanced SQL Analytics From Low-Level Plan Operators](https://db.in.tum.de/~kohn/papers/lolepops-sigmod21.pdf)   
- [x] [Accelerating Queries with Group-By and Join by Groupjoin](http://www.vldb.org/pvldb/vol4/p843-moerkotte.pdf)  
- [x] [A Practical Approach to GroupJoin and Nested Aggregates](https://vldb.org/pvldb/vol14/p2383-fent.pdf)       

### Aggregate Operator

- [ ] [Adaptive Aggregation on Chip Multiprocessors](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.440.8800&rep=rep1&type=pdf)    
- [ ] [Cache-Efficient Aggregation: Hashing Is Sorting](https://dl.acm.org/doi/pdf/10.1145/2723372.2747644)   
- [ ] [Accelerating Aggregation using Intra-cycle Parallelism](https://www.semanticscholar.org/paper/Accelerating-aggregation-using-intra-cycle-Feng-Lo/4f3ba18bad4307a241378731b4c96a6899b56669)   
- [ ] [High Throughput Heavy Hitter Aggregation for Modern SIMD Processors](http://www.cs.columbia.edu/~orestis/damon13.pdf)   


### Window Operator 
- [x] [Efficient Processing of Window Functions in Analytical SQL Queries](https://dl.acm.org/doi/pdf/10.14778/2794367.2794375)   
- [ ] [Optimization of Analytic Window Functions](http://vldb.org/pvldb/vol5/p1244_yucao_vldb2012.pdf)    
- [ ] [Incremental Computation of Common Windowed Holistic Aggregates](https://research.tableau.com/sites/default/files/p1221-wesley.pdf)   
- [ ] [Analytic Functions in Oracle 8i](http://infolab.stanford.edu/infoseminar/archive/SpringY2000/speakers/agupta/paper.pdf)   
- [ ] [Hammer Slide: Work- and CPU-efficient Streaming Window Aggregation](http://www.adms-conf.org/2018-camera-ready/SIMDWindowPaper_ADMS'18.pdf)    
- [ ] [Sliding-Window Aggregation Algorithms](http://hirzels.com/martin/papers/encyc18-sliding-window.pdf)   
- [ ] [Slider: Incremental Sliding Window Analytics](https://dl.acm.org/doi/abs/10.1145/2663165.2663334)   

### Join Operator
- [x] [To Partition, or Not to Partition, That is the Join Question in a Real System](https://dl.acm.org/doi/abs/10.1145/3448016.3452831)  
- [ ] [Main-Memory Hash Joins on Multi-Core CPUs: Tuning to the Underlying Hardware](https://15721.courses.cs.cmu.edu/spring2017/papers/18-hashjoins/balkesen-icde2013.pdf)    
- [ ] [Towards Multi-way Join Aware Optimizer in SAP HANA](http://www.vldb.org/pvldb/vol13/p3019-wi.pdf)   
- [ ] [massively parallel sort-merge joins in main memory multi-core database systems](https://15721.courses.cs.cmu.edu/spring2018/papers/20-sortmergejoins/p1064-albutiu.pdf)   
- [ ] [Main-Memory Hash Joins on Modern Processor Architectures](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=6778794)   
- [ ] [Multi-Way Hash Join Effectiveness](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.706.6219&rep=rep1&type=pdf)   
- [ ] [Efficient Query Processing with Optimistically Compressed Hash Tables & Strings in the USSR](https://ir.cwi.nl/pub/30644/30644.pdf)  

### Compiled
- [ ] [Adaptive Code Generation for Data-Intensive Analytics](http://vldb.org/pvldb/vol14/p929-zhang.pdf)    

### Sort
- [ ] [BlockQuicksort: Avoiding Branch Mispredictions in Quicksort](https://drops.dagstuhl.de/opus/volltexte/2016/6389/pdf/LIPIcs-ESA-2016-38.pdf)  
- [ ] [Merge Path - A Visually Intuitive Approach to Parallel Merging](https://arxiv.org/pdf/1406.2628.pdf)  

### No Classify
- [ ] [Blink: Not Your Father’s Database!](https://link.springer.com/chapter/10.1007/978-3-642-33500-6_1)  
- [ ] [DB2 with BLU Acceleration: So Much More than Just a Column Store](https://researcher.watson.ibm.com/researcher/files/us-ipandis/vldb13db2blu.pdf)    
- [ ] [Adaptive and Big Data Scale Parallel Execution in Oracle](http://www.vldb.org/pvldb/vol6/p1102-bellamkonda.pdf)   

# Resource Scheduler
- [ ] [非中心化调度 Sparrow: Distributed, Low Latency Scheduling](https://cs.stanford.edu/~matei/papers/2013/sosp_sparrow.pdf)  

# Self-driving

- [ ] [Self-Driving Database Management Systems](https://www.pdl.cmu.edu/PDL-FTP/Database/p42-pavlo-cidr17.pdf)  

# Computer architecture
- [ ] [What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)   
- [ ] [A Top-Down Method for Performance Analysis and Counters Architecture](https://ieeexplore.ieee.org/document/6844459)    
- [ ] [TLSF: a New Dynamic Memory Allocator for Real-Time Systems∗](http://www.gii.upv.es/tlsf/files/ecrts04_tlsf.pdf)    
