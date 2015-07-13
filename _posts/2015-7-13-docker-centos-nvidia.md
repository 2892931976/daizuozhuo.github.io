---
layout: post
title: 在centos docker中安装nvidia驱动
---
因为计算需要用到GPU加速, 今天尝试在centos 机器的Docker里安装了GTX980驱动, 记录一下详细安装过程. 首先安装Docker和镜像:

```
sudo yum install docker
sudo systemctl start docker
sudo docker pull centos
```
然后去[nvidia官方网站](http://www.nvidia.com/download/driverResults.aspx/86390/en-us)下载合适的linux驱动放在当前文件夹内.
在默认情况下Docker是不能访问任何device的, 为了能在Docker里访问显卡,必须加上--privileged=true的选项:

```
sudo docker run --privileged=true -i -t -v $PWD:/data centos /bin/bash
```
`-v`将当前文件夹mount到容器内部的/data目录里这样就可以安装nvidia驱动:

```
yum install gcc gcc-c++ kmod mesa-libGL-devel mesa-libGLU-devel libGLEW glew-devel freeglut-devel
sh NVIDIA-Linux-x86_64-346.47.run -a -N --ui=none --no-kernel-module
```
这样就安装好了,可以退出来保存一下, `sudo docker ps`得到container ID, 然后`sudo docker commit $containerID daizuozhuo/nvidia`.

为了能够在容器里面打开显示器,我们还需要在启动时指定DISPLAY:

```
sudo docker run --privileged=true -ti -v $PWD:/data -e DISPLAY=:0 -v /tmp/.X11-unix:/tmp/.X11-unix daizuozhuo/nvidia /bin/bash
```

这样就可以在Docker里跑用到GPU的程序了.