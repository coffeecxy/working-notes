debian的安装和使用
===

本来使用的是centos，centos的稳定性肯定是不用说的，但是但是centos的包是rpm的，虽然也是很好的，但是也想尝试一一下其他的发行版，以前用过ubuntu，但是现在其使用的桌面和大家的都不一样了，就不想用了。

所以就试一下debian，这个是ubuntu的上游，稳定性还是可以的。而且其使用的是gnome的桌面，和centos一样，同时其使用的是deb的安装包，包含的包的数量还是很多的。

debian的稳定性确实很高，我的电脑是双核的，如果在windows下面，如果两个cpu都跑到了100%左右，那么基本就不能用了，但是在debian下面，我编译haskell的文件，来那个cpu基本都到了100%，但是操作的时候基本都还是没有什么影响。

# 安装
安装就不具体说了，直接将ISO文件用ultraiso写到U盘就是了，最好在安装之前将硬盘留出相应的空间，比如我留了30GB，这样安装的时候，有一个选项就是将系统安装到剩余的空间中，就不用分盘了。

## 字体渲染
安装完成之后的第一感觉就是这个字体渲染太恼火了，看着是发虚的，这一点就和ubuntu差了，不过我们可以安装`infinality`,安装了之后就和ubuntu看起来一样的好看了。

`infinality`现在是一个fedora的包，所以不能直接使用，但是有一个中国人已经将其做成了deb包，

	$ git clone https://github.com/chenxiaolong/Debian-Packages.git
	$ cd Debian-Packages/

因为要编译，所以安装需要的工具
	$ sudo apt-get install build-essential devscripts
然后到这两个目录下面看看其依赖是不是满足了
	$ cd freetype-infinality/
	$ dpkg-checkbuilddeps
	$ cd ../fontconfig-infinality/
	$ dpkg-checkbuilddeps
如果没有任何输出，就说明已经满足了，不然的话吧相应的包给`apt-get install`就可以了。

到来那个个目录下面去build
	$ cd ../freetype-infinality/
	$ ./build.sh
	$ cd ../fontconfig-infinality/
	$ ./build.sh
如果提示要权限的话就直接给sudo或者是让root来运行就是了。

好像其中一个会提示有一个错误，不过是生成了deb包的，不用理这个错误。

**特别注意，现在要把这两个deb包放到一个文件夹里面去，因为它们会相互依赖的**

然后在该目录下面执行
```
$sudo dpkg -i *.deb
```

这样就安装好了。

### 配置
安装完成之后要配置
	$ cd /etc/fonts/infinality
	$ sudo ./infctl.sh setstyle
现在会让你选择使用那种style，看了一下，果断使用osx,因为其对字体的渲染做得最好。

重启电脑，会发现字体一点都不发虚了，十分好看。

## 中文输入，ibus的使用
在debian中输入中文有好几个输入框架可以选择，ibus和fcitx是两个用得比较多的。ibus相对fcitx更老，但是还是可以使用的，基于fcitx，现在也有了搜狗输入法，但是它做的时候是基于fcitx的。我使用的是ibus，所以安装了google拼音，虽然没有那么好用，但是还是可以了。

打开synaptic软件管理程序，从里面安装ibus-googlepinyin,这样ibus也会自动按照了。

为了让ibus开机自动启动
	sudo gedit /etc/X11/Xsession.d/95xinput
新建并编辑这个文件，

	export GTK_IM_MODULE=ibus
	export XMODIFIERS=@im=ibus
	export QT_IM_MODULE=ibus
	ibus-daemon&

使用`ibus-setup`来设置ibus.

使用ibus的时候的一个问题是有时候我们会看不到输入框了，而且在panel上面也看不到输入法的指示了，这个时候需要重启ibus，如下

	killall ibus-daemon
	ibus-daemon -d

## 使用zsh
zsh是一个新的sh，一般我们都使用的是bash，但是bash其实是一个很老的shell了，zsh是一个新的shell，其是兼容bash，所以现在使用的还是很多的。

	$sudo apt-get install zsh
就可以安装好了

# 阻止系统的beep
在我使用了xfce之后，发现一个事情就是系统不停的beep,很是烦人。

具体就是在shel中，其会不断的beep，然后就是在nautilus中，只要按一下就会响。


打开`/etc/inputrc`文件，这个文件是设置shell的属性的，其又是在/etc中，所以所有用户都有用。

	set bell-style none
将上面的那句话反注释了。这样就在shell中就不会叫了。


编辑`~/.bashrc`文件，加入下面的内容。

	if [ -n "$DISPLAY" ]; then
	  xset b off
	fi

注意使用的是xset，所以会将xorg中的beep给关掉了。


# debian中的网络配置
当我们安装好了debian的时候，我们会发现网络直接就可以使用了。

后来我想使用一些xfce这个更加简单的桌面，所以就安装了,但是后面还遇到了一些问题，第一个就是发现网络用不了了。

在以前都是使用ifupdown来配置网络，使用的方法就是编辑`/etc/network/interface`这个文件，使用相应的语法来编辑。

但是后面有了network-manager这个包，其可以做到让网络开箱可用。

我安装好了xfce之后，想着gnome的东西我都不使用了，所以就给全部remove了，导致network-manager也被卸载了。

执行
	sudo apt-get install network-manager
	sudo apt-get install network-manager-gnome 
这样可以安装这个daemon和前端的管理器。

要特别注意的是，如果要使用network-manager的话，就不要使用ifupdown了，也就是把`/etc/network/interface`中只是配置loopback，而不要其配置任何的物理网口。
