debian的安装和使用
===

本来使用的是centos，centos的稳定性肯定是不用说的，但是但是centos的包是rpm的，虽然也是很好的，但是也想尝试一一下其他的发行版，以前用过ubuntu，但是现在其使用的桌面和大家的都不一样了，就不想用了。

所以就试一下debian，这个是ubuntu的上游，稳定性还是可以的。而且其使用的是gnome的桌面，和centos一样，同时其使用的是deb的安装包，包含的包的数量还是很多的。

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
	$sudo dpkg -i *.deb
这样就安装好了。

### 配置
安装完成之后要配置
	$ cd /etc/fonts/infinality
	$ sudo ./infctl.sh setstyle 
现在会让你选择使用那种style，看了一下，果断使用osx,因为其对字体的渲染做得最好。

重启电脑，会发现字体一点都不发虚了，十分好看。
