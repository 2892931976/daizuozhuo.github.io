---
layout: post
title: Spark Paper Review
---
##Summary:
This paper presented an In-memory cluster competing system called Spark, which is claimed faster than the current distributed computing system like Hadoop by an order of magnitude. Spark is fast because it stores intermedia results in memory rather in stable storage.  


##Strong:

* General purpose RDD
It proposes a new abstraction called resilient distributed datasets(RDDs) which can be either stored in memory or partitioned across machines. RDD has an important property called lineage which means it has enough information about about how it was derived from other datasets to compute the partitions in stable storage.

* Fault tolerance
Based on the assumptions that recompute an intermedia partial results is faster then backup the whole result in stable data. Spark traces the data transformation logs so that it can recover results when data is lost.

* Goode performance
In the evaluation part, it compares Spark’s performance to Hadoop and HadoopBinMem in terms of real applications. The results show Spark can accelerate data mining tasks successfully.

##Weak:
The data’s transformation actions are limited. Since Spark is aimed to data intensive work like text processing text, so it may doesn’t perform well for other tasks. 
For example, compute sift features for a large set of images will takes several days, and we definitely need to store the intermedia results during the computation since it is too expensive to recompute it. In this case, Spark cannot accelerate this task. 

##Conclusion:
The paper implemented an in-memory computing framework Spark which proves much faster than current computing frameworks for data intensive jobs. The traditional computing frameworks store intermedia results on stable store. Spark, however stores transformation logs so that reduce lots of IO cost. In addition, the RDD can freely move between stable store and memory.  I think this system will be very useful in modern applications.

