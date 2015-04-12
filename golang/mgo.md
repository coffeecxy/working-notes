mgo
====

mongodb是一个nosql，在[mongodb介绍](../mongodb/index.md)中，给出了mongodb的一般性的介绍。
对于mysql这类关系型数据库的使用我们就知道了，一方面，我们可以直接使用sql语句来操作数据库，同时，mysql本身还提供了很多的各个语言的driver，通过这些driver，我们就可以使用mysql了。
在mongodb中，也可以实现类似的功能。
mongodb官方给出了通用的语言的driver，比如C/C++/JAVA之类的，但是其没有给出golang的。
不过在其官网上面列出了社区驱动的golang驱动[mgo](http://godoc.org/labix.org/v2/mgo)，而且其以后应该也会变成官方支持的。

## mgo的一般的用法

首先，连接到数据库，其中的`url`用来指定`mongod`这个服务程序监听的地址，如果成功了的话，那么`err`就是`nil`。没有错误的时候，那么我们就会使用其返回的`session`，用这个`session`去操作数据库。

	session, err := mgo.Dial(url)

先使用`session.DB`选择到我们要操作的数据库，其返回的是一个`Database`，然后使用`Database.C`其选择到一个`collection`，这样就可以在这个`collection`上面进行操作了。
	
	c := session.DB(database).C(collection)
	err := c.Find(query).One(&result)


## Database
Database表示一个mongodb的数据库

###	`func (db *Database) C(name string) *Collection`

这个函数返回一个`*Collection`，表示mongodb中的一个`collection`
>这个是我们用得最多的函数

### `func (db *Database) CollectionNames() (names []string, err error)`
得到这个Database上的所有的collection

### `func (db *Database) DropDatabase() error`
将这个数据库删除了。


当然，里面的所有collection也就会被删除了。

## Collection
#### `func (c *Collection) Count() (n int, err error)`
返回这个collection中的document的数量

#### `func (c *Collection) DropCollection() error`
删除这个collection

### Find
	func (c *Collection) Find(query interface{}) *Query
创建出一个`Query`，注意到`query`参数是`interface{}`，表示可以是任何的数据类型，当然，其需要能够被`bson`给解析掉。
如果给出的是一个`nil`，那么所有的document都会被返回来。

得到这个`Query`之后，可以使用其函数来得到我们想要的结果。

## Query

### Sort 
	func (q *Query) Sort(fields ...string) *Query
将返回的document按照指定的`fields`排序，如果其前面加上了一个`-`,那么表示按照其相反的顺序排序。

比如

	query1 := collection.Find(nil).Sort("firstname", "lastname")
### Limit
	func (q *Query) Limit(n int) *Query
设置返回的document的数量的最大值。

### All
	func (q *Query) All(result interface{}) error

将所有的