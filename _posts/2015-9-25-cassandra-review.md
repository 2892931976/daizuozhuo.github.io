---
layout: post
title: Cassandra Paper Review 
---

这是读完 A. Lakshman, P. Malik, ‘‘Cassandra: a Decentralized Structured Storage System,’’ ACM SIGOPS Operating Systems Review, 2010. 的总结.
##Summary
This paper presents a distributed structured storage system named Cassandra which meet reliability and scalability. Cassandra is developed to solve Facebook Inbox Search problem.

##Strengths
* Scale incrementally

In order to scale incrementally, Cassandra uses consistent hashing to partition the data over a set of nodes. Nodes in the system compose a ring. Each node responsible for the region in the ring between it and its predecessor node on the ring. When a node leave or join the cluster, it will only affect it immediate neighbors so minimize the amount of data to be copied.
Achieve high availability and durability by replication
Cassandra uses Zookeeper to select leader which in charge of the data replication. ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. It is based on Paxos algorithm.
 
* Membership protocol

Cassandra uses Gossip style membership protocol. In gossip protocol, each node sends out some data to a set of other nodes. Data propagates through the system node by node like a virus. Eventually data propagates to every node in the system. A modified version of the Accrual Failure Detection is employed to detect failure.

* Local persistence and fast write performance

When performing a write operation, Cassandra does not write it into a local disk directly. Instead, it writes the commit log to local disk first and stores it in the memory. 

##Weakness
There are not many innovations in Cassandra. Most of its component are built on the existing algorithms and projects, like consistent hashing, ZooKeeper. And the experiment part is not sufficient. The paper only shows its performance in Facebook inbox search scenario. It is better to compare it to some current structured data storage system like MongoDB and MySQL. At last, the paper may add more implementation details of Cassandra.

##Conclusion
This paper introduces Cassandra which is reliable and scalable. The experiment shows it has high throughput and low latency. The results can be more persuasive if more comparison tests are added.
