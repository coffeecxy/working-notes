# ubuntu-java



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
