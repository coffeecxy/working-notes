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
