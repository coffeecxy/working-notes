go tool
===

## go build

go build [-o output] [-i] [build flags] [packages]

编译指定的package,这些依赖的package也会被编译,但是不会安装编译生成的包.

go真正编译的时候使用的平台相关的命令,在我的amd64上面,它们为`6a,6c,6g,6l`.
其中`6a`是plan 9上面的汇编器. `6c`是plan 9上面的C编译器.
这两个工具使用的并不多.

`6g`是对go文件的编译器,其输入一个go文件夹,输出一个.6文件.
`6l`是`6g`对应的连接器,其将.6文件连接到一起生成一个.a文件或者是一个可执行文件(ELF文件)

-o选项指定输出的文件的名字,如果不指定的话,这个名字会被推断出来. 也就是对于一个package `p`,
如果其不是`main`的话,会生成`p.a`,对于main,其名字就是这个package的目录的名字.

-i选项会导致这个包的依赖项会被安装.

有如下的一些build flags

* -a 强制要求将所有up to date的包也从新编译
* -n 输出命令但是不运行这些命令
* -p n 并行运行的编译个数,默认值是CPU的个数
* -race 开启冲突检测
* -v 编译的时候输出包的名字
* -x 输出编译的时候使用的命令,比如说 6g 6l
* -ccflags 'arg list' 传递给6c这个编译器的参数,就是编译c代码的时候会使用的参数
* -compiler name 选择使用那个编译器,可以选择gccgo或者是gc(现在一般都是使用的gc,也就是6g/6l)
* -gccflags 'arg list' 如果使用的gccgo,那么这些就是其编译时候的参数
* -gcflags 'arg list' 使用6g的时候的参数
* -ldflags 'arg list' 使用6l的时候的参数

其中的arg list是用空格分隔的参数
	

