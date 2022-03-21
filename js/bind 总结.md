[[理解Array.prototype.slice.call(arguments) - LXF&蒲公英 - 博客园]]
[[JavaScript 中 call()、apply()、bind() 的用法  菜鸟教程]]

```
	 Function.prototype.mybind = function(context){
    var self = this
     console.log(arguments)
    var _this = Array.prototype.shift.call(arguments)
	//_this 就是context
    var argss = Array.prototype.slice.call(arguments)
    console.log(this, argss)
    return function(){
	//第二次获取传入参数
        bindArgs = Array.prototype.slice.call(arguments)
	//将预设参数与参数参数组合后传给原函数
        return self.apply(_this, argss.concat(bindArgs))
    }
}

function sum(a, b,c)
{
    sum = a + b +c
    return sum
}
sum2 = sum.mybind(null,10)
console.log(sum2(2,3))
```

Function.prototype解释：
具体到`Function.prototype`，就是所有函数的原型，因为通常函数都可以认为是通过new Function制造出来的。换句话说，`Function.prototype`上面承载了用于继承给所有函数的那些属性，例如：call、bind、apply等。
![[Pasted image 20211231152013.png]]

如此sum.mybind才能成立

bind 获取两次arguments
第一次是bind发生时， 低而已bind后的函数执行