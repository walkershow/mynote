
# 理解Array.prototype.slice.call(arguments) - LXF&蒲公英 - 博客园

> ## Excerpt
> 理解Array.prototype.slice.call(arguments) 在很多时候经常看到Array.prototype.slice.call()方法，比如Array.prototype.sl

---
在很多时候经常看到Array.prototype.slice.call()方法，比如Array.prototype.slice.call(arguments)，下面讲一下其原理：

## 1、基本讲解

-   1.在js里Array是一个类 slice是此类里的一个方法 ，那么使用此方法应该Array.prototype.slice这么去用  
    slice从字面上的意思很容易理解就是截取（当然你不是英肓的话） 这方法如何使用呢?  
    `arrayObj.slice(start, [end])` 很显然是截取数组的一部分。
-   2.我们再看call

```
 call([thisObj[,arg1[arg2[[argN]]]]]) 
```

thisObj是一个对象的方法  
arrg1~argN是参数

那么`Array.prototype.slice.call(arguments,1);`这句话的意思就是说把调用方法的参数截取出来。  
如：

```
 function test(a,b,c,d) 
   { 
      var arg = Array.prototype.slice.call(arguments,1); 
      alert(arg); 
   } 
   test("a","b","c","d"); 
```

## 2、疑惑解答

先给个例子，这是jqFloat插件里的代码：

```
if (element.data('jDefined')) {
    if (options && typeof options === 'object') {
        methods.update.apply(this, Array.prototype.slice.call(arguments, 1));
    }
} else {
    methods.init.apply(this, Array.prototype.slice.call(arguments, 1));
}
```

多次用到 Array.prototype.slice.call(arguments, 1)，不就是等于 arguments.slice(1) 吗？像前者那样写具体的好处是什么？这个很多js新手最疑惑的地方。那为什么呢？

因为arguments并不是真正的数组对象，只是与数组类似而已，所以它并没有slice这个方法，而`Array.prototype.slice.call(arguments, 1)`可以理解成是让arguments转换成一个数组对象，让arguments具有slice()方法。要是直接写arguments.slice(1)会报错。

```
typeof arguments==="Object" //而不是 "Array"
```

## 3、真正原理

`Array.prototype.slice.call(arguments)`能将具有length属性的对象转成数组，除了IE下的节点集合（因为ie下的dom对象是以com对象的形式实现的，js对象与com对象不能进行转换）  
如：

```
var a={length:2,0:'first',1:'second'};
console.log(Array.prototype.slice.call(a,0));

var a={length:2,0:'first',1:'second'};
console.log(Array.prototype.slice.call(a,1));

var a={0:'first',1:'second'};
console.log(Array.prototype.slice.call(a,0));

function test(){
  console.log(Array.prototype.slice.call(arguments,0));
  console.log(Array.prototype.slice.call(arguments,1));
}
test("a","b","c");
```

**补充：**  
将函数的实际参数转换成数组的方法

方法一：`var args = Array.prototype.slice.call(arguments);`

方法二：`var args = [].slice.call(arguments, 0);`

方法三：

```
var args = []; 
for (var i = 1; i < arguments.length; i++) { 
    args.push(arguments[i]);
}
```

最后，附个转成数组的通用函数

```
var toArray = function(s){
    try{
        return Array.prototype.slice.call(s);
    } catch(e){
        var arr = [];
        for(var i = 0,len = s.length; i < len; i++){
            
               arr[i] = s[i];  
        }
         return arr;
    }
}
```

> Array.prototype.slice.call(arguments,0) 经常会看到这段代码用来处理函数的参数

网上很多复制粘帖说：Array.prototype.slice.call(arguments)能将_**\*具有length属性的对象\***_ 转成数组，除了IE下的节点集合（因为ie下的dom对象是以com对象的形式实现的，js对象与com对象不能进行转换）

### **关键点：**

> 1、Array是构造函数
> 
> 2、arguments是类数组对象(缺少很多数组的方法)
> 
> 3、call让一个对象调用另一个对象的方法。你可以使用call()来实现继承：写一个方法，然后让另外一个新的对象来继承它（而不是在新对象中再写一次这个方法）
> 
> 4、 slice从一个数组中切割，返回新的数组，不修改切割的数组
> 
> so，其实本质就是arguments这个对象使用了数组的slice这个方法，得到了参数构成的数组（也可以用apply）。
> 
> Array.prototype.slice.call(arguments, \[0, arguments.length\])

```

Array.prototype.slice.call([1,2,3,4,5],0)
[].slice.call([1,2,3,4,5],1)

var a={length:2, 0:'first', 1:'second'};
Array.prototype.slice.call(a);
var a={length:2, 0:'first', 1:'second'};
Array.prototype.slice.call(a,1);
var a={0:'first', 1:'second'};
Array.prototype.slice.call(a,1);
```

### **slice大致内部实现**

```
Array.prototype.slice = function(start,end){
 var result = new Array();
 start = start || 0;
 end = end || this.length; 
 for(var i = start; i < end; i++){
result.push(this[i]);
}
 return result;
}
```

### **转成数组的通用函数**

```
var toArray = function(s){
    try{
        return Array.prototype.slice.call(s);
    } catch(e){
       var arr = [];
     for(var i = 0,len = s.length; i < len; i++){
                   
                   arr[i] = s[i]; 
            }
             return arr;
    }
}
```

call()和apply()

javascript call() apply()

## 一. call()

### 语法定义

```
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

-   thisArg  
    在fun函数运行时指定的是this值。在非严格模式下，thisArg为null和undefined的this值会指向全局对象(浏览器中就是window对象)，同时值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象
-   arg1,arg2,……  
    指定的对象的参数列表

### 方法描述

-   在运行的函数中调用另一个Object的this
-   通过 call 方法，你可以在一个对象上继承另一个对象上的方法

### 应用

#### 1.使用call连接对象的构造器（实现继承）

继承某个对象的方法，对于一些公用的方法可以采取这样的方式`Object.apply(this,arguments)`，没必要重复声明相同的方法。

```
function People(name, age) {
  this.name = name;
  this.age= age;
  this.sayName=function(){
    console.log("my Name is "+name);
   }
}

function one(name, age) {
   People.call(this, name, age);
}

function two(name, age) {
   People.call(this, name, age);
}

var one= new one('Jim', 25);
var two= new two('Tom', 40);
one.sayName(); 
two.sayName(); 
```

#### 2.使用call引用一个函数且指定上下文环境

```
function showContent(){
    var content="这篇文章的标题是"+this.title+" 作者是"+this.author;
    console.log(content);
}
var article={
title:"hello world",
author:"coder"
}
showContent.call(article) 
```

## 二. apply()

### 语法定义

```
fun.apply(thisArg, [argsArray])
```

-   thisArg  
    在fun函数运行时指定的是this值。在非严格模式下，thisArg为null和undefined的this值会指向全局对象(浏览器中就是window对象)，同时值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象
-   arg1,arg2,……  
    一个数组或者类数组对象,可为null和undefined

### 方法描述

与call()类似，只是传入参数的方式不同。

### 实际应用

#### 1\. 求出数组的中的最大值

```
function getMax(arr){
    return Math.max.apply(null,arr)
}
```

#### 2\. 实现继承

与call() 类似

#### 3.确保this指向

可以看一下hoverdelay.js的源码(这段代码是解决hover事件重复触发的bug)，setTimeout会改变this的指向(具体可以看一下，可用`that=this`，保存之前的this指向，再用apply或者call重定向为原来的this环境。

```
(function($){
    $.fn.hoverDelay = function(options){
        var defaults = {
            hoverDuring: 200,
            outDuring: 200,
            hoverEvent: function(){
                $.noop();
            },
            outEvent: function(){
                $.noop();    
            }
        };
        var sets = $.extend(defaults,options || {});
        var hoverTimer, outTimer, that = this;
        return $(this).each(function(){
            $(this).hover(function(){
                clearTimeout(outTimer);
                
                hoverTimer = setTimeout(function(){sets.hoverEvent.apply(that)}, sets.hoverDuring);
            },function(){
                clearTimeout(hoverTimer);
                outTimer = setTimeout(function(){sets.outEvent.apply(that)}, sets.outDuring);
            });    
        });
    }      
})(jQuery);
```

#### 4\. Array.prototype.shift.call(arguments)

`Array.prototype.shift.call(arguments)`，arguments是一个类数组对象，虽然有下标，但不是真正的数组，没有shift方法，这时可以通过call或者apply方法调用Array.prototype中的shift方法。

## 三.区别

MDN原文：

> While the syntax of this function is almost identical to that of call(), the fundamental difference is that call() accepts an argument list, while apply() accepts a single array of arguments.

call()和apply()的用法几乎相同，不同之处在于call()接受的是一个参数列表，而apply()接受的是一个数组参数。(方便记忆的小技巧：Apply for array and Call for comma.)

```
function showContent(title,author){
    var content="这篇文章的标题是"+title+" 作者是"+author;
    console.log(content);
}
showContent.apply(undefined, ["hello", "Jim"]); 
showContent.call(undefined, "world", "Join"); 
```

## 四.参考资料

1.  [Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
2.  [Function.prototype.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
3.  [What is the difference between call and apply?](http://stackoverflow.com/questions/1986896/what-is-the-difference-between-call-and-apply)
