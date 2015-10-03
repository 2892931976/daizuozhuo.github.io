---
layout: post
title: Tachyon Paper Review
---

##Summary
This paper presents a Java implemented memory speed storage called Tachyon.  Tachyon read and write files to the cluster shared memory to eliminate writing bottleneck, and provides fault-tolerance through lineage.

##Strengths
* Provide fault-tolerance through lineage instead of data replication.

Providing fault-tolerance through replicate data is redundant and limited by hard disk bandwidth. With lineage, Tachyon provides both fault-tolerance and high throughput.

* Bound the recomputation cost

To bound the recomputation cost, Tachyon continuously checkpointing files asynchronously in the background. It designed a simple Algorithm called Edge to schedule checkpoints. First, Edge only checkpoints the latest data by checkpoint the leaves of DAG. Second, Edge put more efforts on checkpointing hot files. Third, Edge only evicts checked files inside memory.

* Allocate resources for computation based on job priorities.

Each job has its own priorities. When data is lost, if it is needed by a higher priority job, then tachyon increase its priority to recompute it.

##Weaknesses

Tachyon continuously checkpointing files into the distributed file system like HDFS in the background. This will use a large part of network traffic and affect the memory data sharing performance between nodes.

When memory is not enough, Tachyon has to checkpoint files beyond the threshold, otherwise it will lost these files. In this case, it is more like a cache system rather than a file system.

As a file system, Tachyon lacks of a common file API. To use Tachyon, users have to write programs using it Java API. For example, a compiled program named “translate” can be used like this “translate A B”. It simply read file A and write file B. The Tachyon official forum responds that this is not working in Tachyon Now.

##Conclusion
Tachyon provides a memory speed storage abstraction, which is more flexible than Spark for general applications. 
