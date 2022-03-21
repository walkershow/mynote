---
created: 2021-12-14T14:19:40 (UTC +08:00)
tags: [前端,JavaScript,面试]
source: https://juejin.cn/post/7038371452084551694
author: 前端胖头鱼
---

# 因为实现不了Promise.all，一场面试凉凉了 - 掘金

> ## Excerpt
> Promise.all、new、apply、call、bind这些常见的手写题早已成为面试的宠儿，你如果不会写，可能就被pass了噢，和胖头鱼一起构建这个手写系列，一起努力向前吧！

---
## 前言

(ಥ﹏ಥ)曾经真实发生在一个朋友身上的真实事件，面试官让他手写一个`Promise.all`，朋友现场发挥不太好，没有写出来，事后他追问面试官给的模糊评价是**基础不够扎实，原理性知识掌握较少...** 当然整场面试失利，并不仅仅是这一个题目，肯定还有其他方面的原因。

但是却给我们敲响一个警钟：`Promise手写实现`、`Promise静态方法实现`早已经是面试中的高频考题，如果你对其还不甚了解，耽误你10分钟，我们一起**干到他懂**O(∩\_∩)O

## 常见面试手写系列

> `胖头鱼`最近很想做一件事情，希望可以将前端面试中`常见的手写题`写成一个系列，尝试将其中涉及到的知识和原理都讲清楚，如果你对这个系列也感兴趣，欢迎一起来学习噢，目前已有`66+手写题`实现啦！

[1\. 点击查看日拱一题源码地址（目前已有66+个手写题实现）](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fqianlongo%2Ffe-handwriting "https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fqianlongo%2Ffe-handwriting")

[2\. 掘金专栏](https://juejin.cn/column/7038241267137904677 "https://juejin.cn/column/7038241267137904677")

![[64c15e836f414da592ada77eb9411528~tplv-k3u1fbpfcp-watermark.awebp]]

## Promise.resolve

### 简要回顾

1.  `Promise.resolve(value)` 方法返回一个以给定值解析后的[`Promise`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise") 对象。
    
2.  如果这个值是一个 promise ，那么将返回这个 promise ；
    
3.  如果这个值是thenable（即带有[`"then"`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fthen "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then") 方法），返回的promise会“跟随”这个thenable的对象，采用它的最终状态；否则返回的promise将以此值完成。
    

这是MDN上的解释，我们挨个看一下

1.  `Promise.resolve`最终结果还是一个`Promise`，并且与`Promise.resolve(该值)`传入的值息息相关
    
2.  传入的参数可以是一个`Promise实例`，那么该函数执行的结果是直接将实例返回
    
3.  这里最主要需要理解**跟随**，可以理解成`Promise最终状态`就是这个thenable对象输出的**值**
    

___

**小例子**

```


Promise.resolve(1).then(console.log) 


const p2 = new Promise((resolve) => resolve(2))

Promise.resolve(p2).then(console.log) 


const p3 = new Promise((_, reject) => reject('err3'))

Promise.resolve(p3).catch(console.error) 


const p4 = {
  then (resolve) {
    setTimeout(() => resolve(4), 1000)
  }
}
Promise.resolve(p4).then(console.log) 


Promise.resolve().then(console.log) 

复制代码
```

### 源码实现

```
Promise.myResolve = function (value) {
  
  if (value && typeof value === 'object' && (value instanceof Promise)) {
    return value
  }
  
  return new Promise((resolve) => {
    resolve(value)
  })
}



Promise.myResolve(1).then(console.log) 


const p2 = new Promise((resolve) => resolve(2))

Promise.myResolve(p2).then(console.log) 


const p3 = new Promise((_, reject) => reject('err3'))

Promise.myResolve(p3).catch(console.error) 


const p4 = {
  then (resolve) {
    setTimeout(() => resolve(4), 1000)
  }
}
Promise.myResolve(p4).then(console.log) 


Promise.myResolve().then(console.log) 


复制代码
```

**疑问**

从源码实现中，并没有看到对于`thenable`对象的特殊处理呀！其实确实也不需要在`Promise.resolve`中处理，真实处理的地方应该是在`Promise`构造函数中，如果你对这块感兴趣，马上就会写`Promise`的实现篇，期待你的阅读噢。

## Promise.reject

### 简要回顾

> `Promise.reject()` 方法返回一个带有拒绝原因的`Promise`对象。

```

Promise.reject(new Error('fail'))
  .then(() => console.log('Resolved'), 
        (err) => console.log('Rejected', err))




复制代码
```

### 源码实现

> reject实现相对简单，只要返回一个新的Promise，并且将结果状态设置为拒绝就可以

```
Promise.myReject = function (value) {
  return new Promise((_, reject) => {
    reject(value)
  })
}


Promise.myReject(new Error('fail'))
  .then(() => console.log('Resolved'), 
        (err) => console.log('Rejected', err))



复制代码
```

## Promise.all

### 简要回顾

> `Promise.all()`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。**这个静态方法应该是面试中最常见的啦**

```
const p = Promise.all([p1, p2, p3])
复制代码
```

最终`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

（1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

```
const p1 = Promise.resolve(1)
const p2 = new Promise((resolve) => {
  setTimeout(() => resolve(2), 1000)
})
const p3 = new Promise((resolve) => {
  setTimeout(() => resolve(3), 3000)
})

const p4 = Promise.reject('err4')
const p5 = Promise.reject('err5')

const p11 = Promise.all([ p1, p2, p3 ])
.then(console.log) 
      .catch(console.log)
      

const p12 = Promise.all([ p1, p2, p4 ])
.then(console.log)
      .catch(console.log) 
      

const p13 = Promise.all([ p1, p4, p5 ])
.then(console.log)
      .catch(console.log) 
复制代码
```

### 源码实现

```

Promise.myAll = (promises) => {
  return new Promise((rs, rj) => {
    
    let count = 0
    
    let result = []
    const len = promises.length
    
    if (len === 0) {
      return rs([])
    }
    
    promises.forEach((p, i) => {
      
      Promise.resolve(p).then((res) => {
        count += 1
        
        result[ i ] = res
        
        if (count === len) {
          rs(result)
        }
        
      }).catch(rj)
    })
  })
}


const p1 = Promise.resolve(1)
const p2 = new Promise((resolve) => {
  setTimeout(() => resolve(2), 1000)
})
const p3 = new Promise((resolve) => {
  setTimeout(() => resolve(3), 3000)
})

const p4 = Promise.reject('err4')
const p5 = Promise.reject('err5')

const p11 = Promise.myAll([ p1, p2, p3 ])
.then(console.log) 
      .catch(console.log)
      

const p12 = Promise.myAll([ p1, p2, p4 ])
.then(console.log)
      .catch(console.log) 
      

const p13 = Promise.myAll([ p1, p4, p5 ])
.then(console.log)
      .catch(console.log) 

复制代码
```

## Promise.allSettled

### 简要回顾

> 有时候，我们希望等到一组异步操作都结束了，不管每一个操作是成功还是失败，再进行下一步操作。显然`Promise.all`(其只要是一个失败了，结果即进入失败状态)不太适合，所以有了`Promise.allSettled`

> `Promise.allSettled()`方法接受一个数组作为参数，数组的每个成员都是一个 Promise 对象，并返回一个新的 Promise 对象。只有等到参数数组的所有 Promise 对象都发生状态变更（不管是`fulfilled`还是`rejected`），返回的 Promise 对象才会发生状态变更,一旦发生状态变更，状态总是`fulfilled`，不会变成`rejected`

还是以上面的例子为例， 我们看看与`Promise.all`有什么不同

```
const p1 = Promise.resolve(1)
const p2 = new Promise((resolve) => {
  setTimeout(() => resolve(2), 1000)
})
const p3 = new Promise((resolve) => {
  setTimeout(() => resolve(3), 3000)
})

const p4 = Promise.reject('err4')
const p5 = Promise.reject('err5')

const p11 = Promise.allSettled([ p1, p2, p3 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))



      

const p12 = Promise.allSettled([ p1, p2, p4 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))
        


      

const p13 = Promise.allSettled([ p1, p4, p5 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))
        



复制代码
```

**可以看到：**

1.  不管是全部成功还是有部分失败，最终都会进入`Promise.allSettled`的`.then`回调中
    
2.  最后的返回值中，成功和失败的项都有`status`属性，成功时值是`fulfilled`，失败时是`rejected`
    
3.  最后的返回值中，成功含有`value`属性，而失败则是`reason`属性
    

### 源码实现

```
Promise.myAllSettled = (promises) => {
  return new Promise((rs, rj) => {
    let count = 0
    let result = []
    const len = promises.length
    
    if (len === 0) {
      return resolve([])
    }

    promises.forEach((p, i) => {
      Promise.resolve(p).then((res) => {
        count += 1
        
        result[ i ] = {
          status: 'fulfilled',
          value: res
        }
        
        if (count === len) {
          rs(result)
        }
      }).catch((err) => {
        count += 1
        
        result[i] = { 
          status: 'rejected', 
          reason: err 
        }

        if (count === len) {
          rs(result)
        }
      })
    })
  })
}


const p1 = Promise.resolve(1)
const p2 = new Promise((resolve) => {
  setTimeout(() => resolve(2), 1000)
})
const p3 = new Promise((resolve) => {
  setTimeout(() => resolve(3), 3000)
})

const p4 = Promise.reject('err4')
const p5 = Promise.reject('err5')

const p11 = Promise.myAllSettled([ p1, p2, p3 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))



      

const p12 = Promise.myAllSettled([ p1, p2, p4 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))
        


      

const p13 = Promise.myAllSettled([ p1, p4, p5 ])
.then((res) => console.log(JSON.stringify(res, null,  2)))
        



复制代码
```

## Promise.race

### 简单回顾

`Promise.race()`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

```
const p = Promise.race([p1, p2, p3])
复制代码
```

只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给`p`的回调函数。

```
const p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, 1)
})

const p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 2)
})

Promise.race([p1, p2]).then((value) => {
  console.log(value) 
})

Promise.race([p1, p2, 3]).then((value) => {
  console.log(value) 
})

复制代码
```

### 源码实现

> 聪明的你一定马上知道该怎么实现了，只要了解哪个实例先改变了，那么`Promise.race`就跟随这个结果，那么就可以写出以下代码

```

Promise.myRace = (promises) => {
  return new Promise((rs, rj) => {
    promises.forEach((p) => {
      
      
      Promise.resolve(p).then(rs).catch(rj)
    })
  })
}


const p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, 1)
})

const p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 2)
})

Promise.myRace([p1, p2]).then((value) => {
  console.log(value) 
})

Promise.myRace([p1, p2, 3]).then((value) => {
  console.log(value) 
})

复制代码
```

## 结尾

> 也许你我素未谋面，但很可能相见恨晚。希望[这里](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fqianlongo%2Ffe-handwriting%23%25E5%2585%25AC%25E4%25BC%2597%25E5%258F%25B7 "https://github.com/qianlongo/fe-handwriting#%E5%85%AC%E4%BC%97%E5%8F%B7")能成为你的栖息之地，我愿和你一起收获喜悦，奔赴成长。

以上就是第一篇手写实现原理解析啦！欢迎大家指正其中可能存在的错误和问题
