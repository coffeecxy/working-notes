javascript
====

javascript这门语言比较灵活，如果我们只是使用其好的一面，会发现其能是十分强大的。

在ES6标准中，提出了generator的概念。 其核心的思想就是这个函数可以进入，退出几次，而不是传统的函数只能进入，退出一次。 这个语法在其他的比较高级的语言，比如python,ruby中是早就有了的。

	function* name([param[, param[, ... param]]]) {
	   statements
	}

其语法如上。
* name 这个generator function的名字
* param 这个generator function接收的参数
* statements 运行在其中的语句

generator可以进入退出很多次，在退出的时候，其运行状态会被保存，在下一次进入的时候又恢复。

调用一个generator函数不会导致这个函数马上执行，而是会返回一个iterator. 每次调用这个iterator的next()函数，generator就会运行，直到遇到第一个yield语句，yield语句后面的expression指定了这一次要返回的值，或者是遇到了一个`yield*`语句，这样会交给另外一个generator去运行。 `next()`函数返回的是一个object，其`value`属性包含了从generator中yield回来的值，其`done`属性表示这个generator是不是完全运行完了。

	function* idMaker(){
	    var index = 0;
	    while(true)
	        yield index++;
	}
	
	var gen = idMaker();
	
	console.log(gen.next().value); // 0
	console.log(gen.next().value); // 1
	console.log(gen.next().value); // 2
	
比如上面的代码，`var gen = idMaker();`并不会导致这个函数直接运行。 后面每次调用`gen.next()`都会进入这个函数，运行到yield的地方，然后返回yield后面的表达式的值，在这里，第一次`yield index++`会得到0。 第二次进入后，从`while(true)`开始运行，又会运行到`yield index++`结束，这次返回一个1.

	function* anotherGenerator(i) {
	  yield i + 1;
	  yield i + 2;
	  yield i + 3;
	}
	function* generator(i){
	  yield i;
	  yield* anotherGenerator(i);
	  yield i + 10;
	}
	
	var gen = generator(10);
	
	console.log(gen.next().value); // 10
	console.log(gen.next().value); // 11
	console.log(gen.next().value); // 12
	console.log(gen.next().value); // 13
	console.log(gen.next().value); // 20
	
上面是`yield*`的例子。 同样的，第一次`yield`会将10返回来。 第二次遇到的是`yield*`语句，所以其会到`anotherGenerator`中去找下一个`yield`语句或者是return语句。

上面的例子只是给出了一般的使用方法。

	function* ticketGenerator(){
	    for(var i=0; true; i++){
	        var reset = yield i;
	        if(reset) {i = -1;}
	    }
	}
	
	var tick = ticketGenerator();
	console.log(tick.next().value)
	console.log(tick.next(true).value)
	
实际上，next是可以接受参数的。

上面的工作过程如下。
* `var tick = ticketGenerator();`创建一个generator，在主程序中执行。
* tick.next()让程序进入了`ticketGenerator`函数，一直运行到`yield i`,返回的value是0，done是false
* 程序又返回到主程序中，`.value`输出当前的值
* tick.next(true)又让程序进入了`ticketGenerator`，在`ticketGenerator`中，上次退出时的状态被恢复，主程序通过next传递过来的true赋值给`var reset`，然后将`i=-1`，`i++`,又执行到`yield i`退出，返回value是0，done是false.
* 主程序中`.value`输出值
* 。。。。。。

可以看出，generator可以向主程序传出值，主程序可以向generator传入值。

现在，generator的一个用得很多的地方就是处理回调嵌套