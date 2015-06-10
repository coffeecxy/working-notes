交叉编译
===

使用golang的一个好处是其可以十分方便的进行交叉编译. 因为我经常在windows下面进行开发,但是部署一般都是在Linux服务器上面.

要进行交叉编译,首先需要在Windows上面安装gcc编译工具. 一般在windows上面安装gcc编译工具有两个选择,一个是cygwin,一个mingw,我先试了cygwin,发现编译的时候会有错误. 然后安装了mingw,发现就没有错误了.所以推荐安装mingw. mingw有32和64的版本.我的是64位的机子,所以安装了64位的版本.

Mingw的下载地址为
[http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/5.1.0/threads-posix/seh/](http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/5.1.0/threads-posix/seh/). 下载之后直接解压就可以了.

然后将`PATH`设置一下,并保证`gcc`命令是可用的.

打开`cmd`,`cd /d d:\go\src`.我是讲go安装在`d:\go`的. 运行下面的命令.

	set CGO_ENABLED=0
	set GOARCH=amd64
	set GOOS=linux
	call make.bat --no-clean
	
	set GOARCH=386
	set GOOS=linux
	call make.bat --no-clean
	
	set GOARCH=386
	set GOOS=windows
	call make.bat --no-clean

这样在我的64位的Windows下面就可以编译32位/64位的给Windows/Linux使用的程序了.