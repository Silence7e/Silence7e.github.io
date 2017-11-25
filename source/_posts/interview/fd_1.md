---
title: 前端面试
date: 2017-09-12 17:00:00
updated: 2017-11-25 09:40:00
categories: 面试
tags: [JS,CSS,HTML,面试,前端]
---
不久前扛起了公司前端面试的大旗，总结了一些常用面试题吧。
<!--more-->
### 基本问题

开发过程中使用过哪些浏览器，分别是什么内核，如何解决兼容性问题。

用过哪些标签，作用分别是什么。

用过哪些CSS3的特性，用的时候怎么保证兼容性问题。

用过哪些js框架，用来实现一个什么样的功能。

### 具体问题

#### 题目

什么是SEO，怎么做SEO优化。

#### 题目

<img>的title和alt有什么区别

#### 题目

什么是浮动，如何清除浮动，用sass实现一个清除浮动的mixin。

#### 题目

div中的div如何水平垂直居中

#### 题目

`var a=["a","b","c","d"]`每过 1 秒 打印出一个数组中的值。

#### 题目
```javascript
var y = 1, x = y = typeof x;
x
```
#### 题目
```javascript
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}
//请写出以下输出结果：
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```
#### 题目

打印出斐波那契数列的前n个值。

#### 题目

以下代码输出结果：
```javascript
var foo = {n:1};
var bar = foo;
foo.x = foo = {n:2};
console.log(foo.x);
console.log(bar.x);
```
#### 题目

定义一个函数spacify ,将一个字符串作为参数传入，然后返回一个字符串，不过该字符串相对原有传入参数的变化是字母与字母之间多了一个空格。

```javascript
spacify('hello world') // => 'h e l l o  w o r l d'
```
如何将这个函数直接作用在一个字符串对象上

```javascript
'hello world'.spacify();
```
定义一个log方法，实现控制台输出

```javascript
log('hello world');
```
如果需要传多个参数呢？

接下来的要求是给每一个 log 消息添加一个”(app)”的前辍?

如果要实现 HelloWorld.log()打印出helloworld呢？

题目

现有一个`<div></div>`和一个`<button></button>`, 每点击一次button向div内添加一个p标签，标签内容为序号, 单双数P标签显示不同颜色(:nth-child)，点击p标签打印出当前标签的内容。

如果要求用JQuery也可以。