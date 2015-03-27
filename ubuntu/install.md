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
