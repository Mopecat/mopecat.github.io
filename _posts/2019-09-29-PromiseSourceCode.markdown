---
layout:     post
title:      "JavaScript 基础强化：手写Promise源码"
subtitle:   "基础成就高度"
date:       2019-09-29 12:00:00
author:     "Mopecat"
header-img: "img/post-bg-js-version.jpg"
catalog: true
tags:
    - Javascript 基础强化
    - Javascript
    - ES6
    - 发布订阅
    - Promise
    - 手写源码系列
---

### 前言
随着ES6的广泛应用，越来越多的前端er认识了`Promise`，他是javascript处理异步请求的标准，而且近年公布的ES版本来看，很多的`api`都是在`Promise`的基础上封装的~，比如ES7的`async,await`。由此可见`Promise`在未来会越来越重要，所以为了更好的迎合未来的趋势以及在日常工作中更好的应用`Promise`，我们应该去深入的理解它的原理，这样在处理`bug`的时候才有不慌的底气呀~

接下来本文将分一下几块来对`Promise`进行剖析~
* `Promise`简介，简单介绍一下这是个啥玩意
* `Promise`基础用法，连怎么用都不知道，就想写源码？(╯‵□′)╯︵┻━┻
* `Promise`的基本实现，这个版本不包括链式调用，就是一个基础版本的实现
* `Promise`的链式调用用法，会简单介绍几个例子，还是那句话，先会用再研究源码，有助于理解
* `Promise`链式调用的实现
* `Promise/A+`验证通过的源码，比上面稍微全面一点
* `Promise.all`的用法及实现

全部内容大概就这些，应该篇幅会巨长，所以，可能会分为好几篇~ 写完了再出个统计的吧

### `Promise`简介
首先，`Promise`是被`new`出来的，所以确定 `Promise`是一个类，然后请仔细阅读几遍并记住下面关于`Promise`这个类的特点
* 每次 `new` 一个 `Promise` 都需要传递一个执行器(`excutor`)，执行器是立即执行的
* 执行器函数中有两个参数 `resolve` `reject`都是回调方法
* 默认 `Promise` 有三个状态 `pending(等待)` => `resolve(成功)` `reject(失败)`
* 状态一经改变就不能再次更改即一旦成功不能失败，一旦失败不能成功
* 每个 `promise` 都有一个 `then` 方法

了解了`Promise`的特点，再说一下他的作用，概括的说可以归为两点：

1. 解决并发问题 （同步过个异步方法的执行结果）
2. 解决回调嵌套问题 （比如先获取name 再通过name获取age这种问题）

完事了，了解 `what` `why` 现在开始 `how` 了，黄金法则嘛，咱也是个普通人，得按照规矩来。

编程界名言：`talk is cheap show me your code` 咳咳，我一直把他翻译成 => 敲起来吧我的宝贝们。

🌰-1：基本用法也就是这样了
```
let p = new Promise((resolve, reject) => {
  resolve("success");
  throw new Error("失败"); //如果抛出异常也会执行成功
});

p.then(
  data => {
    console.log(data); // success
  },
  err => {
    console.log(err);
  }
);
```
上述代码，打印success，就是说呢 resolve之后，即使是抛出错误也不会改变状态到失败，即状态一经改变不会再次更改

🌰-2：
```
let p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("resolve在then之后执行"); // 发布
  }, 1e3);
});
// 订阅多个then方法
p2.then(
  data => {
    console.log(1,data);
  },
  err => {
    console.log(err);
  }
);
p2.then(
  data => {
    console.log(2,data);
  },
  err => {
    console.log(err);
  }
);
```
这个例子显示，第一点：`Promise`的实例可以有多个绑定多个`then`方法，第二点： `resolve`异步可能在`then`之后执行，仍然有执行结果。对，那两条注释就是告诉你，这里应用了发布订阅模式。

### `Promise`基础版的实现
这个基础版就是实现上面的基础用法不包含链式调用的，饭得一口一口吃，路得一步一步走，先从基础版开始。
先说一下思路和关键信息，
1. 是个类 
2. 有三种固定状态 => 常量 
3. 创建类的时候传递一个立即执行的执行器 => 传入个方法在 `constructor`中调用 
4. 每个`promise`都有一个`then`方法 => 方法在原型上。then有两个参数，都是方法，一个成功状态调用，一个失败状态调用
5. 由第4条可知需要有一个用来标记状态的属性可以在then中被访问到 => 类需要有个属性 `status`来标记状态，由上面特点可知，默认的状态是`PENDING(等待)`

基于以上四个点可以写出一个这个类的大概架子

```
// 2. 有三种固定状态 => 常量
const PENGDING = "PENGDING" // 等待状态
const FULFILLED = "FULFILLED" // 成功状态
const REJECTED = "REJECTED" // 失败状态
// 1. 是个类
class Promise{
    // 3. 创建类的时候传递一个立即执行的执行器 => 传入个方法在 `constructor`中调用 
    constructor(excutor){
        // 5. 由第4条可知需要有一个用来标记状态的属性可以在then中被访问到 => 类需要有个属性 `status`来标记状态，由上面特点可知，默认的状态是`PENDING(等待)`
        this.status = PENGDING
        // 执行器
        excutor(resolve,reject)
    }
    // 4. 每个`promise`都有一个`then`方法 => 方法在原型上。then有两个参数，都是方法，一个成功状态调用，一个失败状态调用
    then(onFulfilled,onRejected){
        if(this.status === FULFILLED){
            onFulfilled()
        }
        if(this.status === REJECTED){
            onRejected()
        }
    }
}
```
这样一个架子就出来了，然后我们继续完善一下细节，在从上面看到的那些已经写出来的代码继续分析。
6. 回到🌰-1，我们可以看到执行器中是可以抛出错误的，而在改变状态之前抛出错误是会被捕捉到的。用什么捕捉错误？聪明如你一定会想到 => `try,catch`呀
7. 执行器传入的两个方法`resolve`和`reject`，需要将这两个方法定义在执行器之前，然后我们思考一下，这个方法里应该做些什么呢？
这里解析一下： 首先我们可以知道，这两个方法分别对应着两个状态，所以我们可以知道，调用对应的方法时应该修改为对应的状态。其次我们可以知道，调用方法时是可以传参的，那我们需要把这个参数保存下来，因为后面的`then`方法的回调方法中的参数就是数据对应🌰-1中的就是'success'，传到`then`中的`data`

```
// 2. 有三种固定状态 => 常量
const PENGDING = "PENGDING" // 等待状态
const FULFILLED = "FULFILLED" // 成功状态
const REJECTED = "REJECTED" // 失败状态
// 1. 是个类
class Promise{
    // 3. 创建类的时候传递一个立即执行的执行器 => 传入个方法在 `constructor`中调用 
    constructor(excutor){
        // 5. 由第4条可知需要有一个用来标记状态的属性可以在then中被访问到 => 类需要有个属性 `status`来标记状态，由上面特点可知，默认的状态是`PENDING(等待)`
        this.status = PENGDING
        // 7.定义resolve和reject 太长了哈 我就简写了一下
        this.value = undefined // 分别定义下 成功和失败传入的值
        this.reason = undefined
        const resolve = value=>{
          // 需要先判断一下当前状态是不是PENGDING状态，如果不是PENGDING证明已经更改过状态了，那么就不做处理
          if(this.status === PENGDING){
            this.value = value; // 将传入的状态存储在this.value上
            this.status = FULFILLED; // 修改当前实例的状态为 成功状态
          }
        }
        // 同理 reject与上面类似
        const reject = reason=>{
          if(this.status === PENGDING){
            this.reason = reason; 
            this.status = REJECTED; // 修改当前实例的状态为 失败状态
          }
        }
        // 执行器
        // 6. 回到🌰-1，我们可以看到执行器中是可以抛出错误的，而在改变状态之前抛出错误是会被捕捉到的。用什么捕捉错误？聪明如你一定会想到 => `try,catch`呀
        try{
          excutor(resolve,reject)
        }catch(e){
          // 如果抛出错误，promise就会走失败的逻辑，也就是走reject
          reject(e)
        }
        
    }
    // 4. 每个`promise`都有一个`then`方法 => 方法在原型上。then有两个参数，都是方法，一个成功状态调用，一个失败状态调用
    then(onFulfilled,onRejected){
        if(this.status === FULFILLED){
            // 7.需要将对应成功的参数传入到then的成功状态的回调方法中
            onFulfilled(this.value)
        }
        if(this.status === REJECTED){
            // 7.需要将对应失败的参数传入到then的失败状态的回调方法中
            onRejected(this.reason)
        }
    }
}
```
这样下来的一个简易版本的`promise`基本完事了，但是为什么说基本完事了呢？
![感情不够呗](https://mopecat.cn/img/20190126124608_FQjK2.jpeg)
如果你在敲的话，把上面的那两个🌰在这里用一下，你就会看到，🌰-1 是ok的 但是🌰-2就不行了，有问题了，没打印也没报错，恩？这是什么鬼？那么请思考为什么`then`调用的了没有报错也没有打印呢？为什么呢？因为异步了呀，`resolve`是在`then`调用之后才执行的，`then`调用的时候当前`promise`实例是什么状态？由于没有调用`resolve`和`reject`，所以当前实例的状态仍然是 `PENGDING`
![鼓掌](https://mopecat.cn//img/20160205143722_dCrVy.gif)