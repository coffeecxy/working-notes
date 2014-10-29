jdbc mysql
========
只要是一个优点规模的web应用，那么后台都会跑着一个数据库，不然数据的存储根本做不下来。

在其他的语言中，对于数据库应该怎么访问，一般各个数据库厂商，组织都会有一套自己的API，虽然都大同小异，但是还是不太一样的，如果想写出数据库独立的代码，基本是做不到的。Java在这方面做得很好，在EE中有一个标准，为`JDBC API`，这些API规定了数据库提供给Java应该要提供哪些函数，然后要实现那些类。所以说各个数据库厂商都会这样向上层提供接口和类，这样就可以写出与数据库无关的代码。

JDBC本身定义的时候主要是给关系型数据库使用的，比如说mysql,postgres这些，所以对于现在的非关系型数据库，比如说mongodb，使用JDBC的话，其定义的一些函数接口就不太适用了。

### 使用JDBC的一般流程
* 需要使用`Class.forName("com.mysql.jdbc.Driver");`载入要使用的数据库的JDBC driver，当然要完成这一步需要将driver的jar文件加到JAVAPATH里面去（在IDEA中就是将其加入到library中）

* 然后使用`con = DriverManager.getConnection("jdbc:mysql://localhost/test?" + "user=test&password=test");`来连接到数据库，一般来说，要连接到一个数据库，需要提供数据库的host，port，要使用的数据库的名字，用户名和密码，对于不同的数据库，`getConnection`要使用的格式是不同的，当然从其driver给出的文档中是肯定可以找到这个格式的。上面给出的是MySQL的格式。
* 连接到数据库后，就需要读取进行读写操作。就像使用数据库提供的client程序来运行SQL语句一样。但是在JDBC中，首先需要一个`Statement`对象才可以。`Statement`对象不能直接实例化，只能运行`sta = con.createStatement();`得到这个Statement。
* 运行SQL语句，`rs = sta.executeQuery("SELECT NAME,PHONE FROM EMPLOYEES");`。当使用client直接运行SQL语句的时候，得到的结果会直接显示在console中，使用`executeQuery`得到的是一个`ResultSet`对象，我们需要从这个对象中提取出需要的结果。
* 从`ResultSet`中提取出需要的结果。返回的结果可能有很多行，在`ResultSet`内部有一个类似指针的东西，其最开始在第一行的前面，使用`rs.next()`可以将其顺序的指向后面的行，其返回的是Boolean，如果后面没有行了，返回的是FALSE，否则为TRUE。在`ResultSet`上调用getXXX可以得到这一行的相应列的值。getXXX有两类，一类接收的参数是一个string，表示这一列的label，另一类接收的参数是int，表示该列的index，index是从1开始的。

### 使用一个connection
使用上面说的流程会有一个问题就是对于这个servlet处理的每一次request，都会创建一个connection，而实际上，创建connection的过程是比较慢的，而且一般来说，数据库允许创建的connection的数目是有限的，所以对于访问量很大的网址，这样处理是不好的。

在JDBC标准中，规定了`Connection`是线程安全的，所以我们可以将`Connection`定义为该servlet的一个成员域。
```java
private Connection con = null;
```
	
然后在`init()`函数中对其进行初始化。

```java
@Override
   public void init() throws ServletException {
       try {
           //载入driver
           Class.forName("com.mysql.jdbc.Driver");
           //得到一个connection
           con = DriverManager.getConnection("jdbc:mysql://localhost/test?" + "user=test&password=test");
       } catch (SQLException e) {
           throw new UnavailableException("Couldn't get db connection");
       } catch (ClassNotFoundException e) {
           throw new UnavailableException("Couldn't load database driver");
       }
   }

```	

这样在这个servlet处理每个request的时候（可能是多线程的），就都可以使用这个`Connection`了

当然需要在`destory()`中`close`这个`Connection`。

```java
@Override
public void destroy() {
    // Clean up.
    try {
        if (con != null) con.close();
    } catch (SQLException ignored) {
    }
}
```

### transaction
在前面说的所有的query操作使用的都是JDBC默认commit方式，也就是auto commit，意思是执行完了一条SQL之后，结果马上就被数据库写到了硬盘里面，也就是每个query函数都是立即执行的。

如果我们运行的都是`SELECT`语句，这个是没有问题的，但是比如我们做的是一个银行系统，现在一个账户想转账到另外一个账户，完成这个转账需要两步
* 从第一个账户中扣除一定金额
* 给另外一个账户加入一定的金额
如果这两步都安全的完成了，那么就好了，但是如果第一步完成了，但是做第二步的时候出现了问题，那么肯定是**不行的**。我们想要到达的效果是，这两步要么都执行了，要么都没有执行，因为只有这样，数据库才是处于**有效**的状态。

这个就是transaction的概念。就是将几条SQL语句打包成一个整体，让其看起来就像是一条语句，如果所有都执行成功了，那么就算commit了，如果有任何一条没有执行成功，那么就需要被`rollback`。

JDBC默认是auto commit的，也就是不是transaction的，执行

	// Turn on transactions
      con.setAutoCommit(false);
那么就可以变成transaction的了，因为关掉了auto commit之后，我们必须调用`con.commit();`才可以将前面的所有SQL语句的结果写入硬盘。而如果有任何错误的话，都可以在`catch`语句中使用`con.rollback();`还原到上一次commit的地方。

**当使用了transaction之后，connect就不能被多个request共用了**,因为如果共用的话，`con.commit();`随时可能被调用，根本就达不到transaction的效果。

那么使用transaction的话，又回到了开始的问题了，每个request都必须使用一个`Connection`，那么就根本没有这么多`Connection`可以使用。对于一般的中型的数据库来说，可以同时使用的`Connection`的数目是`100`个左右，而对于小型的数据库，比如`Microsoft Accesss`，可以同时有的连接只有10个以内。

#### connection pool
解决这个问题的一个办法就是使用connection pool,就是在一个servlet初始化的时候，建立一定数量的`Connection`，当来了一个request的时候，从这个pool中找出一个没有被使用的`Connection`给其使用，处理完这个request之后，将这个`Connection`还回到这个pool中，这是一个比较高效的使用connection的方式。但是要自己实现起来还是比较复杂，书上有一个很简单版本的，但是不能用在工程中，如果真的要使用的话可以用第三方库。
