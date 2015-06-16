---
layout: post
title: Leader Election 选举算法
---
觉得之前几篇文章有点水，这样还怎么证明自己是搞分布式的呀，今天来一点干货，讲一讲分布式系统中必不可少的选举算法。
leader 就是一堆服务器中的协调者，某一个时刻只能有一个leader且所有服务器都承认这个leader. leader election就是在一组进程中，选举一个leader且让该组的进程都同意这个leader.

假设有N个process, 每个process都有个可以比较的ID，可以提出选举。
leader election算法要满足两点：

* safety: 每一个进程要么不知道结果，要么知道正确的结果（不会选错leader）。
* liveness: 选举算法总会结束，每一个进程都知道了结果。

###Ring leader election 环算法
processes组成了一个环，第i个 process p_i 可以和 p_(i+1)modN通信。

如果进程i发现原来的leader挂了，它就会发起选举，发送一个包含自己ID a_i “Election”的信息。然后当某个进程j收到了这个信息的时候，会将这个a_i与自己的ID a_j比，如果a_i>a_j, 继续转发这条消息，如果a_i<a_j且它之前没有转发过消息,就把a_i取代掉然后把消息发送出去。如果发现收到的标识符和自己的一样，代表自己成为leader，然后把自己当选的信息发送出去。

最坏情况分析：一个进程发起选举，如果它的上一个进程具有最大标识符，则选举消息到达上一个进程需要（N—1）次传递，然后消息又要N次才能宣布它当选，最后需要N个消息告诉大家它当选了，所以需要（3N-1）个消息。最好情况就是发起选举的进程就是leader，只需要（2N）个消息。如果有多个进程同时发起选举，那么只有具有最大ID的进程会完成选举。

但是环算法实用价值很少，因为在选举过程中如果有进程崩溃，选举就无法完成，不符合liveness的要求。

###实际生产中的Leader Election
用Paxos-like (Paxos是解决一致性问题的一种方法) 来选举。
####Google Chubby
A system for locking

每个process最多选举一次，得到最多选票且大于某一数值的process当选。
####Apache Zookeeper
Centralized service for maintaining configuration information

Paxos的变种Zab(Zookeeper Atomic Broadcast)。必须始终保持有leader。

每个进程都向ZK文件系统中写入自己的ID,只要保证写入是原子性的，就把最大ID的进程选为leader.


每个进程都监视着正好比自己ID高的进程，如果那个进程是leader然后挂掉了，那么它就成为新的leader.

###Bully Algorithm 霸道算法
假设系统是同步的，所有进程都可以互相通信。当某一进程发现leader挂掉，如果自己ID最大，就宣布自己是leader，否则向ID比自己高的进程发起选举。发起选举如果没有收到回应，就宣布自己是leader，收到了回应就等待比自己ID大的进程宣布结果。
当一个进程收到从ID比自己低的进程发来的选举消息时，就向更高的进程发起选举。

最坏时间复杂度分析：只需5个消息传递时间

1. 最小ID进程发起选举
2. 第二大ID进程回应
3. 第二大ID进程向最大ID进程发起选举
4. 最大ID进程无回应，timtout
5. 第二大ID进程宣布当选