# 交叉编译msgpack包

因为要出数据,我们要使用msgpack包来完成.
需要从源代码开始进行交叉编译.

首先,下载源代码.

    wget http://msgpack.org/releases/cpp/msgpack-0.5.5.tar.gz

开始编译. 写一个shell

    #! /bin/sh
    CSTOOL_DIR=f:/bisai
    ./configure  --host=arm-linux-gnueabihf --prefix=${CSTOOL_DIR}
    make
    make install

de1-soc使用的开发环境在window中的是cygwin. 最开始的时候我一直使用的是`/cygdrive/f/bisai`来作为`CSTOOL_DIR`. 发现在`make install`的时候总是会说`/cygdrive/f/bisai/lib/libmsgpack.a`找不到. 我看了半天,一直认为路强是完全正确的. 最后才发现,在上面的make步骤中,其显示的都是`d:/xxx/yyy`的路径的形式. 所以改成这种形式之后就可以了.

编译完成之后,会在`f:/bisai/lib`下面有10个文件,其中5个是使用c++的时候要连接的,5个是使用C的时候要连接的,我只使用C的,所以就是后面的5个(`libmsgpackc.xx`).

这几个文件中,我们使用的是动态连接,所以要使用的是那几个.so的文件.
但是这几个so文件其实只有一个是真实的,其他两个都是连接上的.

    libmsgpackc.so ->libmsgpackc.so.2.0.0
    libmsgpackc.so.2 ->libmsgpackc.so.2.0.0

使用这几个文件来进行连接的的时候,`ld`会提示错误,说`libmsgpackc.so`的格式不对,先我也看了很久,还使用了`file`命令来看了这些文件,发现  `libmsgpackc.so.2.0.0`是一个ARM上的动态连接库.
按照上面的链接关系的话,`libmsgpackc.so`也应该是对的才对.

后面就该了一下. 将`libmsgpackc.so.2.0.0`的名字改为`libmsgpackc.so`,重新链接,就可以了.

使用sftp将这个`libmsgpackc.so`传到板子上的`/usr/lib`中. 然后运行我们基于这个库生成的可执行文件`msgpackt`(也通过sftp传上来的). 发现

    ./msgpackt: error while loading shared libraries: libmsgpackc.so.2: cannot open shared object file: No such file or directory

可以看出,在板子上,需要的是`libmsgpackc.so.2`这个名字.

    cd /usr/lib
    ln -s libmsgpackc.so libmsgpackc.so.2
重新运行`msgpackt`,出了正确的结果.

最后跑通了发现要做的事情也不是很多,但是都需要自己慢慢的摸索出来.




