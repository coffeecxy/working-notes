java 8的知识
====

Java语言是一门很成熟的语言，其语言本身的更新速度是很慢的，当然相对于C/C++这种用标准规定的语言来说，其更新速度还是快一点。
所以很多在其他语言中出现了的语法在Java中都会出现得很慢。
相对于Python，ruby这类语言来说，其就是慢了很多。

### lambda
Java 8在语言本身的更新上来说，其**终于**有了lambda函数是其一个巨大的进步。
虽然使用interface可以实现lambda函数的功能，但是使用lambda函数可以很大的减少代码量，当然，实际上写这种类型的interface的时候，IDE都会生成这些代码的。

在Java中，有一类interface，其中只包含了一个函数声明。
比如在GUI程序中经常使用的`ActionListener`这个interface

```
package java.awt.event;
import java.util.EventListener;
public interface ActionListener extends EventListener { 
	public void actionPerformed(ActionEvent e);
}
```

在使用的时候一般都是使用anonymous class，然后直接生成一个对象使用。
```JAVA
JButton testButton = new JButton("Test Button");
testButton.addActionListener(new ActionListener(){
	@Override 
	public void actionPerformed(ActionEvent ae) {
	    System.out.println("Click Detected by Anon Class");
	  }
});
```

因为如果不这样做的话就必须创建一个类来implement`ActionListener`，然后新建这个类的实例，然后将这个实例作为`testButton.addActionListener`的参数。

如果我们需要很多个`ActionListener`，每个做的事情是不一样的，不使用anonymous class的话，就会要实现很多个类，很麻烦。

使用了anonymous class，还是有一个问题，代码的可读性很差，因为代码中有太多的冗余部分，很多的都是样板代码。

lambda函数就是针对这种情况，其可以直接将上面的代码变成如下这样的简单
```
JButton testButton = new JButton("Test Button");
testButton.addActionListener((ActionEvent ae) -> {
	    System.out.println("Click Detected by Anon Class");
	  });
```

lambda函数的格式为
![](lambda.png)

其中`->`前面的为函数的输入参数，后面的为函数体，而函数的返回参数就是根据函数体`return`的值的类型。
这个JS中的函数定义类似，因为在JS中的函数也是不会指定返回类型的。

lambda函数的写法虽然比较简单，但是实际上其还是一个anonymous class的实例的实现。搞清楚这一点十分重要。

其他的东西可以看[oracle提供的文档](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html)
