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
随着ES6的广泛应用，越来越多的前端er认识了`Promise`，而且近年公布的ES版本来看，很多的`api`都是在`Promise`的基础上封装的~，比如ES7的`async,await`。由此可见`Promise`在未来会越来越重要，所以为了更好的迎合未来的趋势以及在日常工作中更好的应用`Promise`，我们应该去深入的理解它的原理，这样在处理`bug`的时候才有不慌的底气呀~

接下来本文将分一下几块来对`Promise`进行剖析~
* `Promise`简介，简单介绍一下这是个啥玩意
* `Promise`基础用法，连怎么用都不知道，就想写源码？(╯‵□′)╯︵┻━┻
* `Promise`的基本实现，这个版本不包括链式调用，就是一个基础版本的实现
* `Promise`的链式调用用法，会简单介绍几个例子，还是那句话，先会用再研究源码，有助于理解
* `Promise`链式调用的实现
* `Promise/A+`验证通过的源码，比上面稍微全面一点
* `Promise.all`的用法及实现

全部能容大概就这些，应该篇幅会巨长所以，可能会分为好几篇~ 写完了再出个统计的吧

### 
