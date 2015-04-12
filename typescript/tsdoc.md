typescript handbook
===

## basic types
为了程序有用，我们需要定义一些基本的数据类型：numbers,strings,structures,boolean.
在typescript中，支持的数据类型和js中是类似的，只是多了一个enumeration数据类型。

### Boolean
最简单的数据类型就是true/false值，在ts和js中都是boolean数据类型。

	var isDone: boolean = false;

### Number
和javascript中一样，在TS中的所以数都是浮点数，他们的数据类型都是number

	var height: number = 6;

### String
另外一个基本的数据类型是用于处理文本数据的。
和在其他的语言中一样，这种数据类型是string.
和JS一样，TS中使用`"`或者是`'`来包围在一个string两边。

	var name: string = "bob";
	name = 'smith';

### Array
和JS一样，TS中也可以使用数组。
数组可以有两种表达形式。
第一种在基本数据类型后面加上`[]`，第二种使用generic array，用`Array<elemType>`

	var list:number[] = [1, 2, 3];
	var list:Array<number> = [1, 2, 3];
	
### Enum
TS比JS多了的数据类型是enum，这个在C#中是有的，其实际上是用来表示number的，但是提供了更加友好的方式。

	enum Color {Red, Green, Blue};
	var c: Color = Color.Green;
	
和C中一样，默认的其实值是0，可以手动的改变这个起始值，这样后面的值就跟着加1了。

	enum Color {Red = 1, Green, Blue};
	var c: Color = Color.Green;

也可以给每一个元素都设置值。

	enum Color {Red = 1, Green = 2, Blue = 4};
	var c: Color = Color.Green;

enum提供了一个功能，其也可以从数值值映射到这个值的name.

	enum Color {Red = 1, Green, Blue};
	var colorName: string = Color[2];
	
	alert(colorName);
	
比如上面，我们现在有值2，但是不知道其在`Color`中的名字是什么，我们可以直接使用`Color[2]`来得到这个名字。

>enum的实现就是一个数组，比如上面的`Color`，其被编译成JS后如下，就是每个元素的名字和值相互映射。 得到一个元素的值和名字也都是从数组中取值。

```
enum Color {Red=1, Green, Blue}
var greenNum: number = Color.Green;
var greenName: string = Color[1];
```

```
var Color;
(function (Color) {
    Color[Color["Red"] = 1] = "Red";
    Color[Color["Green"] = 2] = "Green";
    Color[Color["Blue"] = 3] = "Blue";
})(Color || (Color = {}));
var greenNum = 2 /* Green */;
var greenName = Color[1];
```


### Any
我们可能使用在写代码的时候不知道是什么类型的数据。
这些数据可能是运行的时候动态生成的，也可能是使用第三方库时没有的。
在这种情况下，我们会想要在编译是去掉对这些数据的类型检测。
为了完成这个事情，可以给这些数据`Any`数据类型。

```
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

`Any`数据类型在使用当前已经存在的JS库的时候很有用，因为这样可以让TSC忽略调用这些第三方库的时候的输入输出参数的数据类型。

`Any`也可以用在知道数据的一些类型，但是其他类型有的类型不知道的情况下。比如说，对于一个array，如果要使用多个数据类型，那么只能使用`Any`。

	var list:any[] = [1, true, "free"];

	list[1] = 100;

>Any可以看成是Java中的Object这种类型，比如上面的代码中，`true`是`Boolean`类型的，其可以放到需要`Any`的地方，同时`100`是number的，其也可以放到`Any`的地方。

>JS本身是动态数据类型的，任何变量可以在任何时刻绑定到任何数据类型的。

### void
和`any`相对的数据类型应该就是`void`了，在主流的编程语言中都有这种数据类型。
其一般只会用在函数的返回类型处，表示这个函数不返回值。

```
function warnUser(): void {
    alert("This is my warning message");
}
```

## interface
>在Java中有interface，但是TS中的interface和JAVA中的还不太一样。

在TS中的一个原则是，对于数据类型的检查只是检查到其`shape`上。
有时候这个被叫做`duck typeing` or `structural subtyping`。
在TS中，interface用来做这个事情，这是一种强大的用来约定TS代码和其他代码的方式。

### 第一个interface
这是一个简单的样列。

```
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

类型检查的时候会检查调用`printLabel`函数的参数的类型，其不要求类型完全匹配，而是要求其只有一个参数，这个参数必须有一个`label`属性，这个属性的类型是`string`。注意我们提供的参数实际上还有其他的属性，但是编译器值要求给出的实参`最少`要满足的要求。

可以重写上面的代码，现在我们定义一个interface来描述一个类型`需要`有一个名字为`label`的`string`型的属性。

```
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`LabelledValue`interface是用来描述这个`需要`的一个名字。它仍然是表示需要有一个名字为`label`的`string`型的属性。
要注意的是我们不需要显式的指名`myObj`是实现了这个interface的，而在Java中需要显式的指出来。
在TS中，只要`myObj`有相应的属性就行了。

要注意的是编译器不会关系interface中各个属性出现的顺序，其只关心有哪些类型的属性需要出现。

>TS中这种松耦合处理interface是十分必要的，因为我们使用的时候可能要用第三方的JS库，使用这些库中的数据的时候，我们不能要求这些类型implement了一个interface。

### Optional Properties
一个interface中的所有属性不是都需要的。
这种情况在JS中也经常出现，传递到一个函数的数据可能有一个属性，也可能没有，也可能没有。

```
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

>在上面，`config`的两个属性都是可能出现的，所有在使用之前需要判断其出现与否。 如果我们要求这两个属性都必须出现，那么和JS中的处理方式就不一样了。 但是我们还是想要描述给出的数据需要满足的一个`shape`.

其语法就是如上的加一个`?`.

使用optional properties的好处是，在描述了那些可能出现的属性的同时，还可以保证其他的属性`不可能出现`。
比如，对于上面的代码，如果我们拼写错了，将color写成了collor,那么就会有编译错误发生。

```
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});  
```

>这个特别重要，其限制了在函数中使用这个变量是可以使用的属性，相当于一个`最小权限`的作用，可以尽可能的减少错误。

### function type
interface除了可以定义一个object需要有的属性外，也可以定义一个函数的类型。

在interface中可以函数类型。
```
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```
就是指定其接受的参数和返回类型。

定义了这个interface之后，就可以像其他interface一样使用了。

```
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

比如上面，`mySearch`要满足`SearchFunc`的定义，即其是一个函数，而且其输出和输出类型都必须满足定义。

对于函数重要的是其输入和输出的类型，而其参数的名字是不重要的。

```
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

上面函数的名字就和interface定义的时候的不一样，也是可以的。

函数的参数类型是一起检查的，就是说个数和位置都是重要的。要注意的是，定义的函数没有声明返回类型，但是编译器会根据其返回的类型自动推断出为`Boolean`.

### array types
和使用interface定义function一样，也可以使用interface来定义array.
Array有一个index类型和一个返回类型。

```
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```
>上面的代码中，`["Bob", "Fred"]`是JS中的array，其使用的index是nubmer，而返回的是string,所以满足`StringArray`的要求。

对于index type，有两种类型，string和number。
可以同时支持两种index类型，但是必须保证使用 number index返回的类型是使用string index返回类型的子类型。

### class typescript
#### implemente a interface
在C#和java中，interface的用处就是让一个class来implement它，这样这个类的实例就一定可以使用interface定义了的东西了。
在TS中也可以这样用。

```
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

>在上面的代码中，如果注释掉`currentTime: Date;`那么会报错说`ClockInterface`没有被正确的实现。

还可以同时在interface中定义函数，这样在实现的class中这个函数也必须被实现。

```
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

interface定义了一个class的public side, 而不是同时有public side和private side。


