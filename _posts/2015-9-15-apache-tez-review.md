---
layout: post
title: Apache Tez Paper Review
---
##Summary:
This paper introduces Apache Tez, an open source project optimized for customized data-processing applications running in Hadoop. It models data processing as a data flow graph, so projects in the Apache Hadoop ecosystem can meet requirements for human-interactive response times and extreme throughput at petabyte scale.

##Strengths:
* The Hadoop ecosystem applications have some common features. Apache Tez provides a common implementation for these features to reduce repeated work and fragmentation.

* Allows user to build arbitrary data processing flows as DAG, so it has more flexibility and customizability than MapReduce.

There are some workloads, such as Machine Learning, which do not fit well into the MapReduce paradigm. Since the paper says Apache Tez can model arbitrary data flow, it can meet a broad spectrum of use cases without forcing people to go out of their way to make things work. And the current Hadoop applications like MapReduce, Hive, Pig, Spark, Flink can all build on top of it and their performance can be further improved. As a result, the new Hadoop applications specialized in various area can quickly build with Apache Tez API.

* Offers dynamic performance optimization based on runtime context information 
Runtime configuration gives users the ability to do more optimization. According to the runtime context, users can manipulate the Graph, define vertex processor, choose the best schedule strategy, and define how data is read from data sources.

##Weakness:
The experiments are not clear enough. When comparing Spark and the Tez-based implementations of Spark, it says Tez based Spark performs one time faster. However, Tez only supports Java now and Spark is implemented with Scala. How is the Tez based Spark implemented? In addition, it is expensive for Spark to switch to another API. One time faster is not so persuasive.

##Conclusion:
Tez is a distributed execution framework that works on computations represented as dataflow graphs. It maps naturally to higher-level declarative languages like Hive, Pig, Cascading, etc. This is useful for developing new Hadoop applications.
