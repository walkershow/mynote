
# JS中的bind的实现以及使用 - heshan珊 - 博客园

> ## Excerpt
> 在讨论bind()方法之前我们先来看一道题目： 对于上面这道题目，答案并不是太难，主要考点就是this指向的问题，altwrite()函数改变this的指向global或window对象，导致执行时提

---
在讨论`bind()`方法之前我们先来看一道题目：

var altwrite = document.write;
altwrite("hello");//报错，因为altwrite方法是在window上进行调用的，而window上是没有该方法的

对于上面这道题目，答案并不是太难，主要考点就是this指向的问题，altwrite()函数改变this的指向global或window对象，导致执行时提示非法调用异常，正确的方案就是使用`bind()`方法：

altwrite.bind(document)("hello")

当然也可以使用`call()`方法：

altwrite.call(document, "hello")

本文的重点在于讨论第三个问题`bind()`方法的实现，在开始讨论`bind()`的实现之前，我们先来看看`bind()`方法的使用：

### 绑定函数

`bind()`最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的this值。常见的错误就像上面的例子一样，将方法从对象中拿出来，然后调用，并且希望this指向原来的对象。如果不做特殊处理，一般会丢失原来的对象。使用`bind()`方法能够很漂亮的解决这个问题：

![[copycode.gif]]

this.num = 9; var mymodule = {
  num: 81,
  getNum: function() { return this.num; }
};

module.getNum(); // 81

var getNum = module.getNum;
getNum(); // 9, 因为在这个例子中，"this"指向全局对象

// 创建一个'this'绑定到module的函数
var boundGetNum = getNum.bind(module);
boundGetNum(); // 81

![[copycode.gif]]

### 偏函数（Partial Functions）

Partial Functions也叫[Partial Applications](http://benalman.com/news/2012/09/partial-application-in-javascript/)，这里截取一段关于偏函数的定义：

> Partial application can be described as taking a function that accepts some number of arguments, binding values to one or more of those arguments, and returning a new function that only accepts the remaining, un-bound arguments.

这是一个很好的特性，使用`bind()`我们设定函数的预定义参数，然后调用的时候传入其他参数即可：

![[copycode.gif]]

function list() { return Array.prototype.slice.call(arguments);
} var list1 = list(1, 2, 3); // \[1, 2, 3\]

// 预定义参数37
var leadingThirtysevenList = list.bind(undefined, 37); var list2 = leadingThirtysevenList(); // \[37\]
var list3 = leadingThirtysevenList(1, 2, 3); // \[37, 1, 2, 3\]

![[copycode.gif]]

### 和setTimeout一起使用

一般情况下`setTimeout()`的this指向window或global对象。当使用类的方法时需要this指向类实例，就可以使用`bind()`将this绑定到回调函数来管理实例。

![[copycode.gif]]

function Bloomer() { this.petalCount = Math.ceil(Math.random() \* 12) + 1;
} // 1秒后调用declare函数
Bloomer.prototype.bloom = function() {
  window.setTimeout(this.declare.bind(this), 1000);
};

Bloomer.prototype.declare \= function() {
  console.log('我有 ' + this.petalCount + ' 朵花瓣!');
};

![[copycode.gif]]

**注意：对于事件处理函数和setInterval方法也可以使用上面的方法**

### 绑定函数作为构造函数

绑定函数也适用于使用new操作符来构造目标函数的实例。当使用绑定函数来构造实例，**注意：this会被忽略**，但是传入的参数仍然可用。

![[copycode.gif]]

function Point(x, y) { this.x = x; this.y = y;
}

Point.prototype.toString \= function() { return this.x + ',' + this.y; 
}; var p = new Point(1, 2);
p.toString(); // '1,2'

var emptyObj = {}; var YAxisPoint = Point.bind(emptyObj, 0/\*x\*/); // 实现中的例子不支持, // 原生bind支持:
var YAxisPoint = Point.bind(null, 0/\*x\*/); var axisPoint = new YAxisPoint(5);
axisPoint.toString(); // '0,5'
 axisPoint instanceof Point; // true
axisPoint instanceof YAxisPoint; // true
new Point(17, 42) instanceof YAxisPoint; // true

![[copycode.gif]]

上面例子中Point和YAxisPoint共享原型，因此使用[instanceof](http://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/)运算符判断时为true。

### 捷径

`bind()`也可以为需要特定this值的函数创造捷径。

例如要将一个类数组对象转换为真正的数组，可能的例子如下：

var slice = Array.prototype.slice; // ...
 slice.call(arguments);

如果使用`bind()`的话，情况变得更简单：

var unboundSlice = Array.prototype.slice; var slice = Function.prototype.call.bind(unboundSlice); // ...
 slice(arguments);

### 实现

上面的几个小节可以看出`bind()`有很多的使用场景，但是`bind()`函数是在 ECMA-262 第五版才被加入；它可能无法在所有浏览器上运行。这就需要我们自己实现`bind()`函数了。

首先我们可以通过给目标函数指定作用域来简单实现`bind()`方法：

Function.prototype.bind = function(context){
  self \= this;  //保存this，即调用bind方法的目标函数
  return function(){ return self.apply(context,arguments);
  };
};

考虑到[函数柯里化](http://blog.jobbole.com/77956/)的情况，我们可以构建一个更加健壮的`bind()`：

![[copycode.gif]]

Function.prototype.bind = function(context){ var args = Array.prototype.slice.call(arguments, 1),
  self \= this; return function(){ var innerArgs = Array.prototype.slice.call(arguments); var finalArgs = args.concat(innerArgs); return self.apply(context,finalArgs);
  };
};

![[copycode.gif]]

这次的`bind()`方法可以绑定对象，也支持在绑定的时候传参。

继续，Javascript的函数还可以作为构造函数，那么绑定后的函数用这种方式调用时，情况就比较微妙了，需要涉及到原型链的传递：

![[copycode.gif]]

Function.prototype.bind = function(context){ var args = Array.prototype.slice(arguments, 1),
  F \= function(){},
  self \= this,
  bound \= function(){ var innerArgs = Array.prototype.slice.call(arguments); var finalArgs = args.concat(innerArgs); return self.apply((this instanceof F ? this : context), finalArgs);
  };

  F.prototype \= self.prototype;
  bound.prototype \= new F();
  retrun bound;
};

![[copycode.gif]]

这是《JavaScript Web Application》一书中对`bind()`的实现：通过设置一个中转构造函数F，使绑定后的函数与调用`bind()`的函数处于同一原型链上，用new操作符调用绑定后的函数，返回的对象也能正常使用instanceof，因此这是最严谨的`bind()`实现。

对于为了在浏览器中能支持`bind()`函数，只需要对上述函数稍微修改即可：
