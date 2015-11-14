---
layout: post
title:	Borg Paper Review
---

本文读完Google的论文"Large-scale cluster managemnet at Google with Borg"的总结. Borg是谷歌内部使用了十多年的集群管理系统,是Kubernetes的前身.
##Summary
This paper introduces Google’s large scale cluster management system named Borg. It explains how Borg is designed from both user perspective and developers perspective.

##User perspective
Jobs can be classified to long-running services and batch jobs. Usually long-running services are high priority jobs and batch jobs are low priority jobs. High priority job can preempt low priority job. 

Borg presents an abstraction “cell” to differentiate cluster, which physically defined by high-performance network. A cluster is usually divided to several cells to avoid single point of failure. The user can only see cells instead of cluster. 

When a job is submitted, the user has to specify its name, owner, the number of tasks, and constraints. Each task maps to a set of Linux processes running in a container on a machine. Users can operate jobs through RPC.

Borg reserves a set of resources on a machine as alloc (allocation) for future tasks, to retain resources between stopping a task and starting it again. This can reduce a lot of overhead for resources allocation and reclamation.

##Developer's perspective: The architecture

Borgmaster is the logically centralized controller. It consists of two processes, the main Borgmaster process and a separate scheduler. Borgmaster is replicated five time for scalability and availability. When the elected master fails, Borg will select a new master from the five replicas with the help of Chubby.

Scheduling algorithm has two parts: feasibility checking and scoring. Feasibility checking is to find machines meet the task’s constraints. Scoring is to select the most suitable one from all feasible machine. Borg uses a hybrid of worst fit and best fit scoring algorithm which can reduce the amount of stranded resources.

Borg solves the scalability issue from both Borgmaster and scheduler way. In Borgmaster, the elected master spreads its workload to the master replicas. In terms of Borg scheduler, it caches machine score, use equivalence classes to reduce feasibility checking for task with identical constraints, and use relaxed randomization to avoid calculate feasibility and scores for every machine.  

##Lessons
The paper has already summary the good part and the bad part of Borg so they can learn to design the new cluster management system Kubernetes. 

##Conclusion
This paper revealed lots of technology details on designing a large scale cluster management system. And more importantly, it summarizes the good part and bad part of the Borg from their experiences over the past decade, which is especially useful for other companies to improve their existing in use cluster management system.
