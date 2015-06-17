---
layout: post
title: unlimited channel buffer in Go 

---
channel buffer可以事先分配大小，但是这些是需要占用内存的，事先分配几G内存给一个channel很浪费资源的，所以怎样创建一个无限的channel buffer呢？比较naive的写法就是把东西放进一个队列里，然后时刻检查队列的大小，比如说：

```
for {
	if len(queue) != 0 {
    	data = queue.front()
    	out <- data
	} else {
    	time.sleep(time.Duration(1) * time.Microsecond)
	}
}
```
但是这样写是有问题的，就是当queue的中新加了元素时，并不能及时的检查出来，对性能损害极大。问题的关键在于如何在加入queue时通知out呢？如果直接通知out是会在out处阻塞的。正确写法如下：

```
for {
     if len(queue) != 0 {
          data := queue.front()
          select {
               case out <- data:
                    queue.pop()
               case job = <- in:
                    queue.push(job)
     } else {
          queue.push(<- in)
}
```
select可以同时等待多个channel,所以当有数据进来或者出去时，都可以触发相应的操作。
