mongodb的一些基本知识
========

mongodb是一种nosql，但是其是在所有的nosql中和sql数据库最接近的。其由两个大的程序组成，一个是mongod.exe，这是一个daemon，在前面说了怎么安装这个服务。

第二个是mongo.exe这个程序，这是一个client，通过在这个client可以连接到daemon上，然后通过输入命令，让daemon去操作存储在本地磁盘上面的数据库文件。mongodb使用BSON来存储，这是JSON以二进制表示的形式，实际上，使用者可以认为mongo使用的就是JSON，而JSON是js的Object数据类型的一个子集，这种数据结构的表达力十分强。

因为mongo使用的JSON，所以mongo.exe这个client实际上就是一个js解释器，通过在里面输入合法的js代码，就可以操作mongodb数据库了。


## 存储的结构
在SQL数据库中，其存储的分级为 database,table,row。也就是说，为了解决一个问题，我们会使用一个数据库，这个问题的不同的对象会使用一个table，每个对象的一个实例会使用这个table中的一行。

类似的，对mongodb，使用database,collection,document的分级。为了解决一个问题，我们会使用一个数据库，这个问题的不同的对象会使用一个collection，对于每一个这个对象的实例，使用一个document。

但是这两者不是完全等价的，在sql中，一个table中的每一行的域是完全相同的，只是其值不同，但是在mongo中，一个collection中的每一个document存储的东西可以完全的不相同（比如说对于`test.users`这个collection，其第一个document存储的可以是`{name:cxy}`，第二个document存储的可以是`{age:30}`）如果要在SQL中达到这个效果，那么就要让table有两个域，name和age，然后第一个row的age为空，第二个row的name为空。还有就是在mongodb中，document可以全套，而且SQL中不行，要达到相同的效果，必须使用另外一个table。

总的来说，mongodb比SQL在存储上更加自由，很多使用SQL不能实现的功能在mongodb中都能实现。

## 一些概念
### namespace
在mongodb中有namespace的概念。一个namespace为database.collection。

比如在acrm这个database中有users这个collection，那么acrm.users就是一个namespace。

要注意的是，collection的名字是允许有.的。

比如admin.system.users这个系统自带的collection。其名字为system.users。虽然这个collection看起来好像是system下面嵌套了一个users，但是实际上是名字就是system.users，也就是其namespace是flat（扁平）的。