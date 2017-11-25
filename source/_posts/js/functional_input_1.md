---
title: 初探函数式
date: 2017-09-12 16:00:00
updated: 2017-11-25 09:40:00
categories: 学习
tags: [JS,ES6,函数式]
---
很早之前就听说函数式了，但只了解过一丁点东西，最近开始学习这块的东西，刚入坑感觉好难啊 ಥ_ಥ
<!--more-->
这端代码是照着教程敲下来的，部分代码有改动，

很短的一段代码，我却花了挺长时间才走通了整个流程。
```javascript
/*
var common = {
    log(...args) {
        console.log(...args);
    }
}
*/
let common = require('../common');
/*
function partial(fn, ...args) {
    return function partiallyApplied(...laterArgs) {
        return fn(...args, ...laterArgs);
    }
}
*/
let partial = require('./partial');
function reverseArgs(fn) {
    return function argsReversed(...args) {
        return fn(...args.reverse());
    };
}
function partialRight(fn, ...presetArgs) {
    return reverseArgs(partial(reverseArgs(fn), ...presetArgs.reverse()))
}
function foo(x, y, z) {
    var rest = [].slice.call(arguments, 3);
    common.log(x, y, z, ...rest);
}
var f = partialRight(foo, 3, 4,);
f(1, 2);
```
代码的作用是先输入一些参数，并且这些参数在最后调用的参数列表中处于靠后的位置，

partial 方法是分割先输入和后输入的参数。

reverseArgs 的作用是翻转参数。

反转第一次输入参数-》分割输入-》翻转第二次的输入，并拼接在第一次输入的后边 -》最后翻转所有的输入。

3，4 -翻转-》4，3 -》输入1，2 -翻转—》2，1-拼接-》4，3，2，1-翻转-》1，2，3，4

大概是这样一个流程吧。

看起来真心觉得很累，功力不行啊 ，还得再修炼修炼。

A Za A Za, fighting!!!