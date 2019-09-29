---
layout:     post
title:      "JavaScript 基础强化：函数的应用（高阶函数）"
subtitle:   "基础成就高度"
date:       2019-09-21 12:00:00
author:     "Mopecat"
header-img: "img/post-bg-js-version.jpg"
catalog: true
tags:
    - Javascript 基础强化
    - Javascript
    - 发布订阅
    - 观察者模式
    - 函数柯里化
    - react
---

学习东西总是要记录一下的。多学多看，明年的工资才能更高的呀。加油啊，老铁。我可以的！！！！
### 高阶函数
所谓的高阶函数从通俗的意义上说就是以下两点满足任何一个都是高阶函数：

* 一个函数的参数是函数 

```
function a(){}
a(()=>{})
```
* 一个函数返回一个函数

```
function b(){
    return function(){}
}
```
从我们的经验上可以得知，第一种参数是函数的，我们称之为回调方法。第二种返回一个函数的我们可以叫他拆分方法（把一个功能复杂的函数拆分出去）
结合上面两种定义，我们写一个封装功能时常用的例子

`before`方法:我们希望在调用一个方法之前先调用`before`函数，然后在调用核心功能函数（一步一个注释，像我这么细心的老大哥可不多咯）
```
// 核心功能函数
const core = ()=>{
    console.log('World') // 就这么一个打印信息 一点都不核心好吗？对付看看吧肚子里的油水有限
}
// 我们要实现的功能是这样的 调用newCore 先打印 Hello 然后再打印 World
const newCore = core.before(()=>{
    console.log('Hello')
})
// 如果你不是很满意 还可以调用newCore1 
const newCore1 = core.before(()=>{
    console.log('Fuck The')
})
// 这样你就看出来了吧 我们实现了 核心功能core的复用 然后是怎么实现的呢
```
要实现上面的功能的代码其实很简单
```
// 在原型上扩展一个方法
Function.prototype.before = function(beforeFn){ // 参数是个函数
    return ()=>{ // 返回一个函数 调用传入的参数 然后在调用core
        beforeFn()
        this() // 上面是core调用的before所以 这里的this就是core
    }
}
```
接下来就可以看看效果了，全部是这样的如下：
```
// 在原型上扩展一个方法
Function.prototype.before = function(beforeFn) { // 参数是个函数
  return () => {    // 返回一个函数 调用传入的参数 然后在调用core
    beforeFn();
    this(); // 箭头函数没有this指向，所以向外层寻找，上面是core调用的before所以 这里的this就是core
  };
};
// 核心功能函数
const core = () => {
  console.log("World"); // 就这么一个打印信息 一点都不核心好吗？对付看看吧肚子里的油水有限
};
// 我们要实现的功能是这样的 调用newCore 先打印 Hello 然后再打印 World
const newCore = core.before(() => {
  console.log("Hello");
});
// 如果你不是很满意 还可以调用newCore1
const newCore1 = core.before(() => {
  console.log("Fuck The");
});
// 这样你就看出来了吧 我们实现了 核心功能core的复用 然后是怎么实现的呢 继续往下看

// 接下来我们就来试试结果咯
newCore(); // Hello World
newCore1(); // Fuck the World
```
什么？你还想传递参数啊？哎呀 你可真是个小机灵鬼
下面是传递参数的版本
```
Function.prototype.before = function(beforeFn) { 
  return (...arg) => { // 箭头函数不止没有this, 也没有arguments 所以我用运算符将所有的参数收集成一个数组
    beforeFn();
    this(...arg);  // 然后我再一个一个的展开放到core里
  };
};
// 核心功能函数
const core = (...arg) => { // 然后我再将参数收集称为一个数组 下面再打印一下下
  console.log("World", arg); 
};

const newCore = core.before(() => {
  console.log("Hello");
});

const newCore1 = core.before(() => {
  console.log("Fuck The");
});

// 接下来我们就来试试结果咯 这里我想传多少个参数就传多少个啦
newCore(1,2,3,4,5,6); // Hello World [1,2,3,4,5,6]
// 不传也可以呢 就是打印一个空数组
newCore1(); // Fuck the World []
```
#### 发布订阅
趁热打铁，我们再来看一个示例，`react`的事务队列的原理。

可以在某件事的前面和后面同事增加方法，同时也是发布订阅的一种应用。既然说到发布订阅，那就简单说两句（就两句）

* 特点一 预先定义好一个东西，等某个东西发生的时候执行（这大白话可以吧）
* 特点二 发布 和 订阅 之间**是没有关系的** 
好了，两句写完了（我也就能总结成这样了，再通俗的我也不知道咋写了）还可以用现实中的一些例子来理解一下，
如：**我要在9点钟吃一个苹果** （注意一下下面把这个例子称为苹果例）

预先定义好一个东西： 吃一个苹果 （也就是订阅）

等某个东西发生再执行：9点钟 （也就是发布）

9点钟和吃一个苹果没有关系 （满足特点二）

遮掩是不是好理解一点？？？（大概吧 😯）

来说回代码，下面这个例子就是的发布订阅的简单应用了
```
const perform = (anymethod, wrappers) => { // wrappers是下面传进来的数组
  wrappers.forEach(wrap => {
    wrap.initilizae();
  });
  anymethod(); 
  wrappers.forEach(wrap => {
    wrap.close();
  });
};

perform(() => {
  console.log("核心功能");
}, [
  {
    initilizae() { //（订阅）
      console.log("开始时1");
    },
    close() {
      console.log("结束时1");
    }
  },
  {
    initilizae() { // （订阅）
      console.log("开始时2");
    },
    close() {
      console.log("结束时2");
    }
  }
]);
// 输出结果应该是 开始时1 开始时2 核心功能 结束时1 结束时2
```
那我们再来看一下另一个例子，用发布订阅处理并发问题，但是在此之前先来看看计数器方式处理并发问题的例子了解一下这个故事的前因后果（想跟我发生故事吗？）
```
// 这段代码请在node环境下执行 或者vscode 安装插件 code runner 并在你当前项目的根目录下 新建两个文件，name.txt age.txt 
const fs = require('fs')
let info = {}
let index = 0
function out(){
    if(index === 2){ // 是2的时候info对象里面就有name和age属性了 不管这两个文件哪个先读到 都不会影响输出结果
        console.log('这种方式有点low', info)
    }
}
fs.readFile("name.txt",'utf8',(err,data)=>{
    info['name'] = data
    index++
    out()
})
fs.readFile("age.txt", "utf8", (err, data) => {
  info["age"] = data
  index++
  out()
});
```
再看看优雅的实现方式，应用了高阶函数，可见高阶函数在实际应用中还是非常广泛的
```
// 先写一个after函数
const after = (times,fn) => () => --times === 0 && fn() // 返回一个方法 当参数times减少为0的时候执行回调方法
let info = {}
const out = after(2, () => console.log("优雅的", info));

fs.readFile("name.txt", "utf8", (err, data) => {
  info["name"] = data;
  out();
});
fs.readFile("age.txt", "utf8", (err, data) => {
  info["age"] = data;
  out();
});
```
现在故事情节了解的差不多了吧，我们来看看发布订阅的实现方式吧

```
// 用发布订阅的方式实现
// 用on订阅，emit来发布实现
let e = {
  arr: [],
  on(fn) {
    this.arr.push(fn);
  },
  emit() {
    this.arr.forEach(fn => fn());
  }
};
let info = {};
e.on(() => {
  console.log("ok");
});
// 想写多少个就写多少个
e.on(() => { // 订阅
  if (Object.keys(info).length === 1) {
    console.log("这个订阅会在length为1的时候发布");
  }
});
e.on(() => { // 订阅
  if (Object.keys(info).length === 2) {
    console.log("发布订阅实现", infoOnEmit);
  }
});
fs.readFile("name.txt", "utf8", (err, data) => {
  info["name"] = data;
  e.emit(); //  发布
});
fs.readFile("age.txt", "utf8", (err, data) => {
  info["age"] = data;
  e.emit();  //  发布
});
```
以上就是发布订阅的标准写法了，应付面试题应该足够了 `on`-订阅 `emit`-发布，台下的同学跳起来说了，“这个这个怎么跟Vue里的那个事件啥啥啥的那么像呢”，这位同学请坐下，你说的没错哦，`vue`里面也有很多发布订阅应用，但是不要急哦，我还没写到呢，后面会在本博客的源码篇里面陆续分析哦。（标签就叫源码）

#### 函数柯里化
函数柯里化属于高阶函数，但什么时候函数柯里化呢，通俗点说就是把一个大函数拆分成多个函数（大概就是不停的返回函数）

真理还是要得实践嘛，所以咱们主要还是看例子，那我们就继续来了解一下故事情节好了

下面我们准备写一个类型判断的方法，一般的类型判断怎么实现呢？

```
Object.prototype.toString.call() 
// 随便试两个 就两个
console.log(Object.prototype.toString.call("123")); // [object String]
console.log(Object.prototype.toString.call([123])); // [object Array]
```

然后我们再看看一般的封装是怎么实现的

```
const checkType = (content, type) => {
  return Object.prototype.toString.call(content) === `[object ${type}]`;
};
const b = checkType(123, "Number");
console.log(b); // true
```
功能实现了没有问题，好了故事结束了。 哎！！导演导演，剧本不是这样的！！

好吧 我还以为可以回家吃火锅了呢！

上面的一般封装实现了判断类型的功能是没错的，所有的类型都每次判断的时候手动写入如果写错了就会导致错误，像我这种手残的就很容易敲错啊，老人家太难了，我太难了（尺神经损伤已经好几个月不能打球了，还要奋（划）斗（水）在一线战场搬砖）。

咳咳！说回猪蹄！呸！是主题。恩 主题！

所以我们要尽量不要每次判断都自己写类型，所以我们应用 **函数柯里化** 的时候到了，先来个基础版的尝尝鲜

```
// 柯里化实现（简单基础版）
const checkType = type => {
    return content => {
        return Object.prototype.toString.call(content) === `[object ${type}]`
    }
}
const isString = checkType('String') // 返回内层函数
console.log(isString("123")) // true 
```
到这里我成功应用了**函数柯里化**把一个方法拆分成了多个方法（这篇文章已经够长了，我就写一个`String`的就好了）且每个类型就写了一遍，减少了犯错几率

“这也太麻烦了吧，那么多个类型我要写那么多啊，好烦的啊，我很懒的啊”，好这位同学你没有错，错的是这个世界，懒才是我们的第一动力。

所以上面是基础版嘛，简单补充一下就是下面这个样子了

```
const checkType = type => {
    return content => {
        return Object.prototype.toString.call(content) === `[object ${type}]`
    }
}
const utils = {} // 声明一个工具方法对象
const TYPES = ["Number","String","Object","Array","Boolean"] // 行了行了 手疼
TYPES.forEach( type => {
    utils[`is${type}`] = checkType(type)
})
```
也很简单是吧。哎呀！代码可真是太好玩了，我的天啊。

#### 观察者模式
观察者模式的特点有三个
* 观察者和被观察者是有联系的
* 被观察者里面存了观察者
* 观察者模式包含发布订阅

继续看码了，我敲代码千百遍，代码对我如初见~，当然这种事不会的，正所谓码敲百遍，其意自现。没事多敲敲，肯定好处多多啊

```
// 被观察者
class Subject {
  constructor() {
    this.arr = [];
    this.state = "我不饿";
  }
  //  通过这个方法将观察者存入arr
  attach(o) { 
    this.arr.push(o);
  }
  setState(newState) { // 通过这个方法来设置状态并通知观察者
    this.state = newState;
    this.arr.forEach(o => o.update(newState));
  }
}
// 观察者
class Observer {
  constructor(name) {
    this.name = name;
  }
  update(newState) { // 通过这个方法来更新到被观察者的状态
    console.log(`${this.name}  知道了 九儿  ${newState}`);
  }
}

let o1 = new Observer("Mopecat"); // 创建一个观察者 Mopecat（我）
let o2 = new Observer("Sean");  // 创建一个观察者 Sean（我媳妇）
let s = new Subject("九儿"); // 九儿是我家的猫
s.attach(o1); // 将o1存入被观察者 也就是Mopecat
s.attach(o2); // 将o2存入被观察者 也就是Sean
s.setState("又饿了"); // 更新状态 
// 会输出Mopecat  知道了 九儿  又饿了
// Sean  知道了 九儿  又饿了
```
上面就是也一个简单的**观察者模式**的例子

其中向被观察者中存入观察者的直到状态更新的时候再通知观察者的过程就是发布订阅的应用

而被观察者和观察者是有联系的：被观察者中存了观察者。好像上面的三个特点改成两个就可以了。算了就这吧

### 总结
高阶函数的应用可以说在我们的工作中无处不在，学好了他，我们就可以写出更加优雅的代码。一起加油共勉吧。
同时，来收藏[我的博客][1]吧，来互加友链吧！我的朋友们！！


  [1]: https://mopecat.cn/


