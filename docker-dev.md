
## docker安装和镜像设置

在[官方文档](http://docs.docker.com/installation/ubuntulinux/)中详细的介绍了在
ubuntu 14.04中安装docker的方式.

主要就是使用

	wget -qO- https://get.docker.com/ | sh
从docker的官网上面下载一个shell文件然后使用这个shell文件来安装docker,这样可以保证安装好的docker是现在的
最新的版本.

安装完成之后,运行

	$ docker pull ubuntu


在国内,daoCloud这个公司提供了一个docker hub的镜像. 使用这个镜像可以明显的加速docker的下载速度.

[这个链接](https://dashboard.daocloud.io/mirror)中介绍了怎么使用daoCloud提供的
镜像来加速我们的镜像下载过程.

其实际进行的操作就是修改`/etc/default/docker`文件,让docker这个daemon启动的时候指定其要使用的下载来源.

修改之后再重启docker daemon

	$ sudo service docker restart

这样实际启动的时候运行的命令就是

	docker --registry-mirror=http://bf2350bc.m.daocloud.io

## docker开发环境的建立

docker的核心开发人员在开发docker的时候,使用了像编译器开发人员一样的模式,就是用一个
稳定版本的docker来开发下一个版本的docker.

### fork一个docker的源代码

首先需要在github上fork并且clone一个docker的源代码下来.

	cd $HOME/code
	git clone https://github.com/coffeecxy/docker.git docker
这个过程可能会比较久,因为docker的源代码很大,有差不多50M.

### 编译Dockerfile

上面我们已经安装了一个稳定版本的docker了.所以可以用来编译我们的开发环境的镜像了.

切换路径到clone下来的docker目录下

	cd docker
	make
	
这样会开始编译一个叫做`docker:master`的镜像. 会发现这样是不能成功的生成这个镜像的. 需要修改
Dockerfile文件.

在按照下面一节描述的修改之后,重写运行`make`命令,这次会运行很久,因为会下载很多包,然后运行各种从源代码
生成可执行文件的过程.

### 修改原始的Dockerfile

1.在开始安装基本的软件之前,更换源

```
# CHANGE: 在国内需要使用另外一个源,ubuntu官方源太慢了
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse" >/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse" >>/etc/apt/sources.list
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse" >>/etc/apt/sources.list
```
因为这个Dockerfile使用的是ubuntu 14.04开始构建的,其默认的软件源再国内会很慢,
换成阿里云的源.

2.安装golang的时候,默认下载源代码是从

	curl -sSL https://golang.org/dl/go${GO_VERSION}.src.tar.gz

同样的,golang.org是被墙了的,需要换成国内的

	curl -sSL http://golangtc.com/static/go/go${GO_VERSION}.src.tar.gz

3.编译golang的工具的时候,是先将当前平台的工具连构造出来,然后又构造了所有支持的其他
平台的工具连,如果不想要这些工具连的话,可以注释掉.

4.安装完了golang之后,又会安装以前版本的gofmt和go cover,还会使用gem下载一个ruby的包,
这些网址都是被墙了,所以注释掉.

5.安装busybox和hello-world两个镜像的地方可以注释掉,因为现在使用的是docker hub,下载
起来会相当慢.

总之,Dockerfile文件并不复杂,如果发现不是必须的东西,而且因为网络的原因还下载不了,那么
就只能跳过一些步骤了.
但是其中对golang的安装和docker源代码的下载是必须的,因为docker是使用go写的,而且我们是
基于现在的源代码来修改docker的.

> 因为这个修改还是比较麻烦的,我将自己修改过的Dockerfile又push到了自己的github上面.

### 编译过程的原理分析

在这个路径下面,有一个Makefile,docker使用make来完成生成镜像的工作.
其在build的时候,使用的是这个目录下面的Dockerfil文件. 
运行命令

	~/code/docker$ sudo make --debug
make的时候使用的是默认的配置.

	default: binary

	binary: build
		$(DOCKER_RUN_DOCKER) hack/make.sh binary
		
	build: bundles
		docker build -t "$(DOCKER_IMAGE)" .

默认是要编译一个binary这个target,其依赖于build,再依赖于bundles.
其中`$(DOCKER_IMAGE)`是`docker:master`

`$(DOCKER_RUN_DOCKER)`的值是

	docker run --rm -it --privileged -e BUILDFLAGS -e DOCKER_CLIENTONLY -e DOCKER_EXECDRIVER -e DOCKER_GRAPHDRIVER -e TESTDIRS -e TESTFLAGS -e TIMEOUT -v "/home/cxy/code/docker/bundles:/go/src/github.com/docker/docker/bundles" "docker:master" 

其意思为,在成功的编译好了`docker:master`这个镜像之后,要启动这个镜像,在其中运行
	
	hack/make.sh binary
要特别注意的是其中的`-v`选项,这儿只是将docker目录下的bundles给映射了. 因为编译生成的docker可执行文件
会被放在docker/bundles这个文件夹下面.

实际上,在开发的时候,我们需要将整个docker目录映射到容器中去,为了方便,使用了如下的重命名.这些重命名
写在了`.bashrc`文件中

	# 方便对docker进行开发的时候使用的别名
	# 启动容器
	alias docker-daemon='docker run -it --privileged --name=docker-master  -v "/home/cxy/go/src/github.com/docker/docker:/go/src/github.com/docker/docker" \
	 "docker:master" /bin/bash'
	# 删除容器
	alias docker-remove='docker rm -f docker-master'
	# 重启容器
	alias docker-restart='docker restart docker-master'
	# 从另外一个终端中进入容器
	alias docker-shell='docker exec -it docker-master /bin/bash'
	


	


