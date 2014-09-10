用户管理命令
========
和sql中一样，mongodb中也有用户的概念，但是两者是不太相同的。

在sql中，user是一个顶级的概念，也就是说，在我们要操作数据库之前，我们就必须先以某个用户登陆进入，不然是连接不上数据库的。同时，每个用户都有自己的权限，这些权限包括对所有数据库的可见，可读，可写之类的操作能否执行。

也就是说，在sql中，用户以及其操作权限是最开始就要求了的。

但是在mongodb中，我们发现其默认是不需要使用用户登陆的，而且其用户是关联到某一个database上面的（当然可以给它其他database的权限），而不是将用户作为一等公民，在连接的时候就需要提供。


### 添加用户

`db.createUser(user, writeConcern)`

向当前的db添加一个用户，意思是如果当前的db是`test`，添加的用户名字是`cxy`，那么添加的用户就是`cxy@test`

`user, writeConcern`都是document.

其中user的格式如下，其中user,pwd,roles都是必须给出来的，customData是可以不给的。

	{ user: "<name>",
	  pwd: "<cleartext password>",
	  customData: { <any information> },
	  roles: [
	    { role: "<role>", db: "<database>" } | "<role>",
	    ...
	  ]
	}
其中roles表示这个用户的权限，其中第一种格式表示在db上面的权限为role，第二种格式没有给出db，那么就表示在当前的db上面的权限。

	"roles" : [ { role: "clusterAdmin", db: "admin" },
	                             { role: "readAnyDatabase", db: "admin" },
	                             "readWrite"
	                             ] },
								
比如上面的roles，结果就是这个用户在admin上有clusterAdmin和readAnyDatabase权限，在当前db上面有readWrite权限。

`admin`这个database是一个特殊的database，其是用来管理所有的用户的权限的一个database，这个和SQL中的那几个系统自带的database的意思是一样的（就是user,password那几个database）。

### 删除用户
`db.dropUser(username, writeConcern)`

同样的，要删除用户之前，要先选择到一个database。

username是一个string，表示要删除的用户。

	use products
	db.dropUser("accountAdmin01", {w: "majority", wtimeout: 5000})
这样就删除了accountAdmin01@products这个用户。

### 删除所有用户
`db.dropAllUsers(writeConcern)`

上面的语句删除了该db上的一个特定用户，而dropAllUsers可以删除其上的所有用户。

	db.dropAllUsers( {w: "majority", wtimeout: 5000} )
	
### 更新用户信息
#### 更新用户的所有信息
`db.updateUser(username, update, writeConcern)`

从上面createUser的语法可以看出，一个user的信息为pwd,customData,roles这三个，使用updateUser就可以更新username（string）的这三个信息中的任何一个，update(document)中的值会完全替换已有的值。

**使用`updateUser`是不好的，最好使用下面的单独更新该用户某个信息的函数**
#### 更新pwd
`db.changeUserPassword(username, password)`

这个函数很直观，就是更新username(string)的password(string)

	db.changeUserPassword("reporting", "SOh3TbYhx8ypJPxmt1oOfL")
											
#### 给用户增加权限

`db.grantRolesToUser(username, roles, writeConcern)`

username是我们想要更新权限的用户名，roles是一个array，里面包含了我们要更新的权限

比如原来用户的权限为

	"roles" : [
	    { "role" : "assetsReader",
	      "db" : "assets"
	    }
	]
	
执行操作

	use products
	db.grantRolesToUser(
	   "accountUser01",
	   [ "readWrite" , { role: "read", db: "stock" } ],
	   { w: "majority" , wtimeout: 4000 }
	)

其权限变为

	"roles" : [
    { "role" : "assetsReader",
      "db" : "assets"
    },
    { "role" : "read",
      "db" : "stock"
    },
    { "role" : "readWrite",
      "db" : "products"
    }
	]

也就是这个函数会将用户的权限更新，如果该权限已经出现过了，那么没有改变，不然的话会添加到rules这个array中。

#### 给用户减少权限
`db.revokeRolesFromUser( "<username>", [ <roles> ], { <writeConcern> } )`

和grantRolesToUser相反，这个函数让用户的权限变少，其参数和grantRolesToUser完全相同，


紧接着上面的例子，

	use products
	db.revokeRolesFromUser( "accountUser01",
	                        [ { role: "read", db: "stock" }, "readWrite" ],
	                        { w: "majority" }
	                      )

下面的两个权限就从accountUser01的roles中删除了。

	 { "role" : "read",
      "db" : "stock"
    },
    { "role" : "readWrite",
      "db" : "products"
    }

其权限变成

"roles" : [
    { "role" : "assetsReader",
      "db" : "assets"
    }
]

---
从上面可以看出当一个user要去操作一个database的时候，mongodb首先会检查这个user的权限是不是可以的，如果不可以，那么那个操作是不能进行的。
