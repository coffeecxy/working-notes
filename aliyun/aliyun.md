zoom testing aliyun ubuntu server
====
购买了一个最基本配置的云服务器。

阿里云送了一个系统盘，是20G的，而Ubuntu系统只占用了1个多G，所以先还是可以不用买数据盘了。

测试zoom的服务器的id是 i-281r8ypnv
名称改成了 zoom-testing

安装的系统是阿里云提供的 ubuntu 14.04 lts

公网ip是  121.42.154.86

连接终端管理器要输入的密码是 789223
这个终端管理器就相当于我们真实的主机前面放的显示器，如果系统真的出现了很大的问题，那么可以从这儿使用单用户登录的模式来解决。

## 系统启动之后的基本配置

默认情况下，阿里云服务器是开放了root用户的，下面的操作新建一个用户，然后将root用户lock了。
这样通过ssh到服务器上面也不能以root用户登陆了。

新建一个用户管理的用户

	adduser cxy	
	
将用户放到sudo中

	usermod -aG sudo cxy

将root用户的密码删除了，然后将其Lock了。
	
	passwd -dl root
	passwd -aS
	


### 重置root密码

有可能我们操作系统的时候出现了问题，那么就可以在阿里云提供的控制台中提供的重置密码操作，这个操作是将root密码重置了。
重置了密码之后，要记住**重新启动服务器**，然后就又可以用root登陆了。
	
## 部署服务器运行环境

### Java和tomcat环境

更新一个apt-get

	sudo apt-get update

更新一下系统

	sudo apt-get upgrade
	
安装java jre，当然也可以安装jdk，因为我不在上面开发，所以只安装jre就可以了gn

	sudo apt-get install openjdk-7-jre
	
安装tomcat7,安装的时候也可以安装其他的文档之类的，这儿就不安装了
	
	sudo apt-get install tomcat7
	
安装完成之后，tomcat7会启动。安装一个curl，来看看是不是启动起来了

	sudo apt-get install curl
	curl http://localhost:8080

发现有返回东西，说明启动起来了。

### 绑定域名

首先，通过浏览器访问

	http://121.42.154.86:8080/
如果可结果为tomcat的欢迎页面，那么说明tomcat工作了。

我们的域名是在万网申请的，为`chouxiaya.com`，在万网的域名解析月面将这个服务器的公网ip添加为A记录，那么就可以解析了。

	http://chouxiaya.com:8080/
	
访问上的网址，也可以得tomcat的欢迎页面了。

对tomcat的安装就是这样了，需要更多的tomcat的知识[tomcat7](../java/tomcat/tomcat.md)
