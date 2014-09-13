nodejs的使用
====

nodejs是基于google V8这个js引擎做的一个用于server的项目。

### windows下面的安装
直接到官网下载相应的msi文件。

安装在d:\nodejs下面。

### 配置
安装完了之后需要进行一些配置。

#### 配置npm

在`D:\nodejs\node_modules\npm`下面有一个npmrc文件，将里面的内容改为如下

	prefix=D:\nodejs\node_modules\npm
	registry = http://registry.cnpmjs.org

其中的`prefix`指定了global方式下载的package存放的位置。`registry`指名了下载package地方，这儿使用了国内的镜像。

### 修改nodevars.bat文件
因为我们上面修改了`prefix`，所以要修改一下nodevars.bat文件。将其中的PATH修改为如下。

	set PATH=%~dp0\node_modules\npm;%~dp0;%PATH%