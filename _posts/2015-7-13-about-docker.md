---
layout: post
title: 深夜关于docker的一些思考
---
Docker最近很火,但是实际上在哪里能够用到呢? 在做分布式系统的过程中,我一直都在思考, 在什么样的情况下我要使用docker. 这段时间搭建了一个六台服务器的分布式计算系统, 文件系统使用的是Gluster, 调度系统是自己写的[Admiral](https://github.com/daizuozhuo/admiral), 开发测试时一直用的ansible来操纵集群, 觉得docker并没有什么用处. 知道今天组里让我再配一个集群时我突然想到,这不就是需要用到docker的情况吗? 如果我事先将这些东西全部打包好, 那岂不是直接将镜像拷贝过去就好了. 下面将记录一下使用docker过程中的点点滴滴.

##为什么要使用docker?
需要用到虚拟机的地方就可以使用docker, docker的优势就是比虚拟机快.

docker不适合做开发, 因为没有GUI不能用IDE.

docker可以用来编译和测试程序, 像jenkins那样.

docker可以当做生产环境, 可以很方便的增加服务器.

可以用kubernetes 管理docker.

用etcd管理配置搭建一个docker集群

##Docker三个概念：
###镜像 Image
镜像是一个只读的模板
可以用 docker save, docker load来导出导入镜像

dcoker images: 列出当前的镜像

docker build -t $(tag) $(dockerfile directory) :从当前文件夹的Dockerfile建立镜像

###容器 Container
容器是从镜像创建出来的实例，用来运行应用

docker ps :列出正在运行的容器

###仓库 Repository
类似于git的仓库


