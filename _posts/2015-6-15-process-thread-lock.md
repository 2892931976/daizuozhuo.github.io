---
layout: post
title: Process, Thread, Locks
---

为准备校招，今天复习了一下线程进程这些基本概念。
###process
拥有独立的adress table, 进程之间的变量和数据不共享，需要通过进程间通信来交流，如pipe, file, socket, signal.

####查看进程

```
ps -e -o pid,ppid,command
-e :every process
-o :output format
```
####杀死进程
kill pid: send SIGTERM signal to process

kill -KILL pid

SIGTERM 和 SIGKILL 的区别：
都是为了终止进程，但是SIGTERM可以被进程处理并无视掉，而SIGKILL总是会杀死该进程。

####创建进程create process

C library: `system(), fork(), exec() `.

####调度进程process scheduling

使用nice调节进程的优先度，niceness越高，优先度越低，如：`nice -n 10 sort input.txt > output.txt`
renice可以改变进程的优先度

####僵尸进程zombie process

一个进程结束了，但是他的父进程没有等待(调用wait / waitpid)他， 那么他将变成一个僵尸进程。 但是如果该进程的父进程已经先结束了，那么该进程就不会变成僵尸进程， 因为每个进程结束的时候，系统都会扫描当前系统中所运行的所有进程， 看有没有哪个进程是刚刚结束的这个进程的子进程，如果是的话，就由Init 来接管他，成为他的父进程
避免僵尸进程：
通过wait和waitpid等函数等待子进程结束，或者用signal函数为SIGCHLD安装handler，因为子进程结束后， 父进程会收到该信号，可以在handler中调用wait回收。

###thread 

thread存在进程之中并且分享进程的资源，多个thread可以在一个process中且共享heap space，线程拥有自己的stack和register.

###死锁deadlocks

1. Mutual Exclusion
2. Hold and Wait
3. No Preemption
4. Circular Wait

###并发编程的三种方法
进程， I/O多路复用， 线程
