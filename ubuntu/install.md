使用需要注意的事情
====

## 字体渲染
在ubuntu中,字体的渲染本来还是可以的,但是还是有一些问题,所以我们一般还是会使用其他的一些增强渲染技术来达到更加理想的字体渲染,这样看起来会舒服很多.

一般的,我们都会使用infinality这技术.

[https://github.com/achaphiv/ppa-fonts/blob/master/ppa/README.md](这个连接)下面详细的介绍了怎么进行安装.

安装的时候需要使用PPA

	sudo add-apt-repository ppa:no1wantdthisname/ppa
	sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get install fontconfig-infinality

安装完成之后需要选择渲染的模式:

	cd /etc/fonts/infinality/
	sudo bash infctl.sh setstyle
	# modify conf.d/*

有好几种模式,我选择的是`osx2`,也就是第`5个`.

在上面的链接给出的方法中,其还说了改变`/etc/profile.d/infinality-settings.sh`中的`USE_STYLE`变量,但是我发现不用改变也是可以的.直接使用其给出的`DEFAULT`值就可以了.

而第三个是改变gnome下面的应用的hinting/antialiasing的,因为我使用的是kde,所以就不用设置了.

完成之后,注销并重新登录就可以了.

## 安装搜狗拼音输入法
现在搜狗拼音输入法只有ubuntu下面才有，这个也是我使用ubuntu的一个原因。



在安装之前,要将系统中现在存在的输入法全部删除了,不然安装好了之后一般都会有一些问题.

	sudo apt-get remove ibus
	sudo apt-get remove fcitx*
一般使用的输入法就是ibus或者是fcitx,要注意删除ibus的时候不要使用`ibus*`,因为其会将不需要删除的东西
给山除了.

现在搜狗支持的是ubuntu 12.04和14.04这两个LTS版本,直接到搜狗的官网去下载相应的deb安装包.
使用
	
	gdebi sogoupinyin_1.2.0.0042_amd64.deb
	
来安装这个deb文件,因为`gdebi`可以安装其依赖的package.

安装完成之后再重启,一般就可以了.


## orcale jdk安装
在ubuntu server中，可以直接使用`apt-get`的方式来安装`openjdk`。
`openjdk`和orcale jdk都是对java的实现，而且也都差不多，但是我开发程序的时候是在win上开发的，使用的是orcale jdk，所以还是希望在应用部署的时候使用的还是orcale jdk，这样出现错误的可能就会小一些。

orcale官方只提供了rpm安装包，Ubuntu要使用的deb的安装包，不过ppa提供了这个支持。

首先，如果安装了openjdk，那么使用如下的默认将Openjdk卸载了。

	sudo apt-get purge openjdk*

要使用PPA，需要使用`apt-add-repository`命令，在Ubuntu sever中其默认是没有的，使用如下的命令安装。

	sudo apt-get install python-software-properties
	sudo apt-get install software-properties-common

添加orcale jdk的ppa，然后更新一下本地的apt cache，在安装orcale jdk 8.

	sudo add-apt-repository ppa:webupd8team/java
	sudo apt-get update
	sudo apt-get install oracle-java8-installer
	
## tomcat7使用Orcale jdk

在安装了orcale jdk之后，如果启动tomcat7，其会提示`JAVA_HOME`没有设置，在`/etc/init.d/tomcat7`文件中，将`JDK_DIRS`中改为如下的


	JDK_DIRS="/usr/lib/jvm/default-java ${OPENJDKS} /usr/lib/jvm/java-6-openjdk /usr/lib/jvm/java-6-sun /usr/lib/jvm/java-7-oracle /usr/lib/jvm/java-8-oracle"
	
也就是这个启动脚本默认没有搜索orcale jdk 8的路径，加上之后就可以了。
