---
layout: post
title: 用etcd做服务发现及Go代码示例 
---
[之前](/kubernetes-and-zrpc/)说到要让我们的系统zrpc可以动态调整集群大小. 那么首先就要支持服务发现, 就是说当一个新的节点启动时,可以将自己的信息注册给master, 让master把它加入到集群里, 关闭之后也可以把自己从集群中删除. 既然是用Go语言, 我们采用etcd来做服务发现, 在我们这个情况下其实就是一个membership protocol, 用来维护集群成员的信息. 实现时发现线在网上的etcd Go API的教程都是采用`go-etcd`这个已经被废弃的库, 官方推荐使用`etcd/client`, 用法跟以前的稍有不同, 我就写一个简短的示例好了.

整个代码的思路很简单, worker启动时向etcd注册自己的信息,并设置一个过期时间TTL,每隔一段时间更新这个TTL,如果该worker挂掉了,这个TTL就会expire. master则监听`workers/`这个etcd directory, 根据检测到的不同action来增加, 更新, 或删除worker.

首先我们要建立一个etcd client:

```Go
func NewMaster(endpoints []string) {
	cfg := client.Config{
		Endpoints:               endpoints,
		Transport:               client.DefaultTransport,
		HeaderTimeoutPerRequest: time.Second,
	}

	etcdClient, err := client.New(cfg)
	if err != nil {
		log.Fatal("Error: cannot connec to etcd:", err)
	}
	master := &Master{
		members: make(map[string]*Member),
		KeysAPI: client.NewKeysAPI(etcdClient),
	}
	go master.WatchWorkers()
	return master
}
```
这里我们先建立一个etcd client, 然后把它的key API放进master里面,这样我们以后只需要通过这个API来跟etcd进行交互. Endpoints是指etcd服务器们的地址, 如"http://192.168.0.1:2379"等. `go master.WatchWorkers()` 这一行启动一个Go routine来监控节点的情况. 下面是WatchWorkers的代码:

```Go
func (m *Master) WatchWorkers() {
	api := m.KeysAPI
	watcher := api.Watcher("workers/", &client.WatcherOptions{
		Recursive: true,
	})
	for {
		res, err := watcher.Next(context.Background())
		if err != nil {
			log.Println("Error watch workers:", err)
			break
		}
		if res.Action == "expire" {
			member, ok := m.members[res.Node.Key]
			if ok {
				member.InGroup = false
			}
		} else if res.Action == "set" || res.Action == "update"{
			info := &WorkerInfo{}
			err := json.Unmarshal([]byte(res.Node.Value), info)
			if err != nil {
				log.Print(err)
			}
			if _, ok := m.members[info.Name]; ok {
				m.UpdateWorker(info)
			} else {
				m.AddWorker(info)
			}
		} else if res.Action == "delete" {
			delete(m.members, res.Node.Key)
		}
	}

}
```
WatcherOptions里recursive指的是要监听这个文件夹下面所有节点的变化, 而不是这个文件夹的变化. 当返回`expire`的时候, 该节点不一定挂掉, 有可能只是网络状况不好, 因此我们只将它暂时设置成不在集群里, 等当它返回`update`时在设置回来. 只有返回`delete`才明确表示将它删除.

worker这边也跟master类似, 保存一个etcd KeysAPI, 通过它与etcd交互.然后用heartbeat来保持自己的状态.


```Go
func NewWorker(name, IP string, endpoints []string) *Worker {
	cfg := client.Config{
		Endpoints:               endpoints,
		Transport:               client.DefaultTransport,
		HeaderTimeoutPerRequest: time.Second,
	}

	etcdClient, err := client.New(cfg)
	if err != nil {
		log.Fatal("Error: cannot connec to etcd:", err)
	}

	w := &Worker{
		Name: name,
		IP: IP,
		KeysAPI: client.NewKeysAPI(etcdClient)
		
	}
	go w.HeartBeat()
	return w
}

func (w *Worker) HeartBeat() {
	api := w.KeysAPI

	for {
		info := &WorkerInfo{
			Name: w.Name,
			IP:   w.IP,
			CPU:  runtime.NumCPU(),
		}

		key := "workers/" + w.Name
		value, _ := json.Marshal(info)

		_, err := api.Set(context.Background(), key, string(value), &client.SetOptions{
			TTL: time.Second * 10,
		})
		if err != nil {
			log.Println("Error update workerInfo:", err)
		}
		time.Sleep(time.Second * 3)
	}
}
```
完整的代码可以在在github [etcd-service-discovery](http://github.com/daizuozhuo/etcd-service-discovery)上下载.
