spring的一些配置
========

### 打开spring输出的DEBUG信息
为了输出spring在做的每一步的详细，需要使用`log4j`，首先在`pom.xml`文件中添加

    <!--log4j-->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
	
这样IDEA就可以直接从maven中下载apache log4j包了。

然后需要配置log4j

新建`\src\main\resources\log4j.properties`文件，向文件中写入如下的内容，

	log4j.rootLogger=DEBUG, A1
	log4j.appender.A1=org.apache.log4j.ConsoleAppender
	log4j.appender.A1.layout=org.apache.log4j.PatternLayout
	log4j.appender.A1.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n

上面的内容使用了`properties`文件的语法，具体的信息要看log4j的文档。
需要特别注意的是第一句`log4j.rootLogger=DEBUG, A1`，其表示要输出的信息的级别，默认的是`INFO`，就是在IDEA中可以看到的信息前面都有`INFO`,这里将其改成了`DEBUG`，所以就可以看到所有的DEBUG信息了。

## 2014年11月19日12:31:23
### `xxx-servlet.xml`和`@Configuration`
Spring mvc需要配置
