ruby的一些笔记
===

ruby是一门动态语言，其特性和python是差不多的，都是上世纪九十年代出现的语言。

其最成功的地方是其`ROR(Ruby on Rails)`web编程框架。

作为语言本身，其受到了`Smalltalk`的影响，是一门`OO`程度很高的语言，这一点特别是对于`C++`,`java`来说的。

下面是我学习`ruby`时觉得重要的东西。

## 类

## new和initialize(对象的初始化问题)
在所有的OO语言中，对对象进行初始化的函数叫做构造函数。在C++,JAVA中，使用构造函数的方法是类似的，其目的就是保证一个对象在使用之前总是会被正确的初始化（这一点在C中就做不到，所以C中对于每一个结构体都要显示的调用函数进行初始化）。

在C++/java中，构造函数要完成两件事情。
 1. 为对象 instance 分配需要的内存
 2. 调用构造函数将这部分内存初始化，返回这个对象

对于第一步，因为C++/Java是静态语言，所以一个类的对象要使用多少的内存是知道的，所以只需要分配就可以了。

要特别注意的是，在C++/JAVA中，上面的两步是**连着进行的，是不可分割的**，也就是说，不管是以怎么样的方式创建一个instance，那么上面的两个步骤总是会进行完，这个也是C++/java设计的时候的一种考虑。

在ruby/python中，对于对象的初始化是不同的。表现出来就是，其需要两个函数来完成这个功能，第一个是new，第二个是initialize


```ruby
class MyClass
  def initialize(aStr)
    @avar = aStr
  end
end

ob = MyClass.new("hello world")
```

首先，是如上的代码，这个代码是和C++中十分类似的，`MyClass`有一个`initialize`，类似于C++中的构造函数。然后要新建一个`MyClass`的instance的时候，调用其`new`函数。

需要解释的是

 MyClass.new这个语句是一个函数调用，而不是C++中的关键字的使用（C++中，new是关键字，那么其要完成的事情编译器就会完成了，是不能改变的），`MyClass`是一个类，new是这个类的一个Class method（也就是C++中的static method，类方法）。在ruby中，类也是一个object，而其是由Class这个类创建出来的，在Class这个类定的定义中，其定义了一个new方法，所以Class的实例MyClass，才会有new方法，这个new方法完成的事情是
 * 在内存中新建一个Myclass的instance 
 * 如果MyClass有initialize方法的话，调用initialize方法，调用的时候传入的参数就是new的参数.

一般情况下，我们都会使用这种默认的方式。但是，我们可以在MyClass中实现自己的new方法，这个new方法可以覆盖Class中定义的new方法。

但是一个思考就是，为什么要这样呢，上面的默认的new方法有什么问题么，为什么还要开放这个接口，让程序员可以实现不一样的new呢？

这儿有点说不清楚想表达什么了，放一放？

TODO:

### `.class`和`.superclass`
这两个方法名字看起来很像，但是其完成的事情是不一样的。

`instance.class`返回的是`instance`所属于的类。

```ruby
class MyClass

end

class MyOtherClass < MyClass

end

instance = MyClass.new

instance.class == MyClass
```
上面的代码返回`true`,因为instance是被MyClass new出来的。

还要注意的是，在ruby中
```ruby
AnyClass.class == Class
```
总是成立的，因为在ruby中，所有的类都是Class类的实例。

`AClass.superclass`返回的是ruby中一个Class的父类，比如上面的代码中
`MyOtherClass.superclass == MyClass`是成立的。

需要注意的是，如果`a.superclass`中的`a`是一个instance，而不是一个类，那么是会报错的，因为只有类（这种对象）才是有superclass这个属性。

**实例(instance)和类(class)都是对象(object)，实例是类使用new方法创建出来的，所有的类中Class类是特殊的，其他的类都是由Class类new出来的**


**所有的类构成一个树状的继承结构，在最上面的是BasicObject，然后是Object（在较早的版本中，就是Object）**