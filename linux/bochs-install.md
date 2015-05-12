# bochs-install

## 在ubuntu14.04中安装

在sourceforge下载bochs的2.4.5的源代码.解压这gz文件,cd到这目录下面.

    sudo ./configure --enable-debugger --enable-disasm

运行完成后会生成一个Makefile文件,打开这Makefile文件,编译bochs的时候,需要使用pthread这个库,二`./configure`生成的Makefiel中连接的参数中默认是没有这些选项的,在Makefile中的LIBS字段中添加

    -lz -lrt -lm -lpthread

然后运行

    sudo make && sudo make install

这样就安装好了.
