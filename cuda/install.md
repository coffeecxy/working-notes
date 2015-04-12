## 安装软件

要使用cuda c进行编程,需要安装cuda toolkit和ms visual studio这两个软件. 因为安装cuda toolkit的时候会向vs中安装插件,所以要先安装vs,再安装cuda toolkit.

### visual studio 2010/2012/2013

cuda toolkit在开发的时候,一般对于vs 2010后面的版本都是支持了的.
但是vs2010（不是sp1）在64位系统上面链接会有问题,我安装了比较新的vs 2013.

vs的安装包是iso文件，可以使用daemon tools来挂载，就不要解压了。

安装完了vs 2013之后,可以安装一个visual assistx,这个插件可以让vs开发C/C++代码更加方便.

### cuda tookit
在NVIDIA的官网下载最新版本的cuda toolkit,我下载的7.0.
对其进行安装,安装的时候就选择默认选项就行了,其会将所有的东西装在C盘,但是C盘够大的话,也没有什么问题.

cuda toolkit安装的时候,会对系统的环境变量做一些更改.

    CUDA_PATH=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0
    CUDA_PATH_V6_5=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v6.5
    CUDA_PATH_V7_0=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0

其会设置cuda toolkit的路径,其中`CUDA_PATH`是我们安装的7.0的根目录,因为以前安装过6.5,所以有`CUDA_PATH_V6_5`这个变量,`CUDA_PATH_V7_0`也是安装的7.0的目录.
这三个环境变量可以方便外部工具找到cuda toolkit.

    Path=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\bin;C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\libnvvp;

其还会将cuda toolkit中的各个可执行文件的路径加入到path的前面,这样就算系统中安装了老版本的cuda toolkit,仍然会使用刚刚安装好的.


安装好了之后,可以使用vs打开cuda toolkit安装的cuda samples这个解决方案.然后运行一下其中的一些工程,如果可以运行,那么说明安装成功了.

