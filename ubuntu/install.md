ubuntu安装
====

## 安装kubuntu操作系统
ubuntu发型版根据使用的桌面环境的不同,分了好几个子发行版.默认的发行版使用的是unity桌面,我自己笔记喜欢kde桌面,所以下载了kubuntu的iso文件.

使用uui(universal usb installer)这软件将这个ISO文件写到准备好的u盘中,我们也可以使用其他的安装方式,比如使用硬盘安装,使用grub4dos等等,使用uui刻录的方式方便简单.

重启电脑之后进入u盘启动,为了使得电脑从u盘启动,有些主机只需要在重启的时候按着`f12`之类的按键,有些电脑需要修改bios.后面就可以进入到安装界面了.如果要在电脑上面安装双系统,一般就是win7+ubuntu,那么要注意先用分区工具让硬盘留出一块空的空间给ubuntu使用.

安装界面中基本比较傻瓜.特别要注意的是分区的那一步,因为我们想要使用双系统,所以一定要选择手动分区.分区的时候可以使用最简单的分区方法,只使用`/`和`swap`分区.其中`swap`分区一般要2G就够了,剩下的都给`/`分区,文件系统一般都使用ext[234],现在一般都是ext4.

安装完成之后会重启电脑,重启之后会发现使用的是grub2引导程序了,而且我们以前的win7也会在grub2中,所以双系统就安装好了.

## 更改系统源&升级软件包
重启进入到ubuntu之后,首先需要更改软件源,因为在国内如果使用默认的软件源的话会相当慢.

修改`/etc/apt/sources.list`文件的内容如下

    deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

我们使用了国内阿里云的源,其中的`trusty`是14.04这lts版本的代号,如果使用的是其他版本,那么改成相应的代号就行了.然后运行

    sudo apt-get update && sudo apt-get upgrade

首先更新本地的包列表,然后更新所有的软件包.

## 安装win7下面的字体
ubuntu安装之后,其中的字体,特别是中文字体是很少的,因为我们安装的双系统,在win7下面是有很多我们常用的字体,我们可以将这些字体复制到ubuntu中再安装好.

首先使用Dolphin这文件管理器挂载win7的系统盘,也就是在dolphin中点击win7系统盘对应的分区.

    cd /media/cxy/725C49B95C4978BD/Windows/Fonts
转到win7的字体存放目录下面去.

    sudo mkdir /usr/share/font/win7
    sudo cp * /usr/share/font/win7

在ubuntu的字体目录下新建win7目录,然后将win7下面的所有的字体文件都复制到ubuntu的相应目录下.

    cd /usr/share/font/win7
    sudo chmod 644 *
    sudo mkfontscale && sudo mkfontdir && sudo fc-cache -fv

要特别注意的是,要将这些字体的属性改为644,这样其他的用户才可以正确的读取到这个字体,也就是使用这个字体,不然会出现问题.

mkfontsacle和mkfontdir会win7目录下面建立索引文件,这样在应用要搜索字体的时候,速度会快了很多.fc-cache命令会对当前系统中安装的所有字体文件做一个cache,这样其他的应用就知道有这些字体可以使用了.

## 修改字体渲染

ubuntu的字体的渲染相对于debian来说还是好了很多的,但是还是有一些问题,所以我们一般还是会使用其他的
一些增强渲染技术来达到更加理想的字体渲染,这样看起来会舒服很多.

一般的,我们都会使用infinality这技术.[github上的infinality](https://github.com/achaphiv/ppa-fonts/)是一个比较好的字体渲染方案.注意这个软件只对ubuntu 14.04有用.实际上,在这个github中,作者同时给出了在ubuntu 14.04中一般的字体渲染和对openjdk中的swing应用的字体渲染的优化技术.

### 一般的字体渲染
安装的时候需要使用PPA

	sudo add-apt-repository ppa:no1wantdthisname/ppa && \
	sudo apt-get update && \
	sudo apt-get upgrade && \
	sudo apt-get install fontconfig-infinality

安装完成之后需要选择渲染的模式:

	cd /etc/fonts/infinality/
	sudo bash infctl.sh setstyle
	# modify conf.d/*

输出为

	Select a style:
    1) debug       3) linux       5) osx2        7) win98
    2) infinality  4) osx         6) win7        8) winxp
    #?

有好几种字体渲染模式可以,我选择的是`osx2`,也就是第`5个`. 这种模式是模拟osx中的字体渲染方式.

在上面的链接给出的方法中,其还说了改变`/etc/profile.d/infinality-settings.sh`中的
`USE_STYLE`变量,但是我发现不用改变也是可以的.直接使用其给出的`DEFAULT`值就可以了.

而第三个是改变gnome下面的应用的hinting/antialiasing的,因为我使用的是kde,所以就不用设置了.

完成之后,注销并重新登录就可以了.

### openjdk中的字体渲染

在linux中,如果要使用java的话,我们一般都是使用openjdk,openjdk(其实orcale的jdk也是一样的)在ubuntu中的字体渲染有问题,特别是那种swing应用中的字体,给人的感觉是很虚的.因为我要使用clion,其使用的是swing来做的gui,所以如果直接使用openjdk的话,界面很难看.


	sudo add-apt-repository ppa:no1wantdthisname/openjdk-fontfix && \
	sudo apt-get update && \
	sudo apt-get install openjdk-7-jdk

同样的使用了上面的那个ppa,注意后面安装的时候,会发现安装比较慢,因为不是使用的ubuntu的
官方源,而是使用的这个ppa了.

### 安装clion
在linux中以前没有一个好的C/C++ IDE,不过现在intelij出了一个clion,现在已经是1.0版本了,在其EAP的时候使用过,发现还是很好用的.

clion需要使用jre来运行,官方提供的clion包中,自己是包含了一个jre的,其使用的是clion.sh这个
脚本来启动clion,在里面有一个部分就是找到系统中一个JDK,如果没有设置一些环境变量的话,就会
使用自带的那个jre,所以在寻找JDK代码的后面加上下面的代码,这样就能保证总是使用我们上面安装的
打了字体patch的JDK.

	# 使用我们自己安装的对字体渲染进行了patch的jdk
	JDK="/usr/lib/jvm/java-7-openjdk-amd64"

还需要修改`clion64.vmoptions`这个文件,里面的内容为启动java是使用的选项,保证文件中包含了下面的两行.

    -Dawt.useSystemAAFontSettings=on
    -Dsun.java2d.xrender=true

重新启动之后,会发现clion的字体漂亮了很多.

## 安装搜狗拼音输入法

现在搜狗拼音输入法只有ubuntu下面才有，这个也是我使用ubuntu的一个原因。

搜狗输入法使用的是fcitx输入法框架,但是其又对这个框架做了一些改变,所以安装之前,需要将这个框架的所有东西都卸载了.

	sudo apt-get remove fcitx*

现在搜狗支持的是ubuntu 12.04和14.04这两个LTS版本,直接到搜狗的官网去下载相应的deb安装包.使用

	gdebi sogoupinyin_1.2.0.0042_amd64.deb

来安装这个deb文件.这里我没有使用dpkg直接安装,因为gdebi可以自动的搜索这个deb包依赖的包.

安装完成之后再重启,一般就可以了.

## 安装adobe pdf reader
对于pdf阅读器,在windows下面有很多很好的,ubuntu的各个发行版也都有自己的默认的pdf阅读器,但是大部分功能都是比较差的.

[这个blog](http://ubuntuhandbook.org/index.php/2014/04/install-adobe-reader-ubuntu-1404/)中给出在64位的14.04中安装adobe pdf reader的方法.

1. 下载pdf reader的deb包.
2. 使用gdebi来安装这文件.会发现要安装很多的依赖,因为这是一个32位的程序,所以所有32位的基本的包都会被安装. 安装完成之后会发现还是启动不了,还需要安装以下的包.


    sudo apt-get install libgtk2.0-0:i386 libnss3-1d:i386 libnspr4-0d:i386 lib32nss-mdns* libxml2:i386 libxslt1.1:i386 libstdc++6:i386
完成之后就可以成功启动pdf reader了.

## 使用zsh而不是bash
ubuntu默认使用的shell是bash. bash是一个用得比较多的shell,但是用起来并不舒服.

现在大家使用得非常多的一个shell是zsh. 在加上oh-my-zsh提供的一堆的theme和plugin. zsh用起来就十分舒服了.

首先,安装zsh.

    sudo apt-get install zsh

oh-my-zsh的主页[github上](https://github.com/robbyrussell/oh-my-zsh),其安装的过程也是十分的简单.

    wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh

安装完成之后使用

    chsh
在提示输入了密码之后输入`/bin/zsh`,然后注销再重新登录,就会使用`zsh`作为login shell了.

在`$HOME`下面有一个.zshrc文件.oh-my-zsh已经往里面写了一些东西了. 我修改了其中的两个内容,一个是开启了自己要使用的plugin,然后修改了一下使用的主题.

    plugins=(git git-flow  github)
    ZSH_THEME="dstufft"
