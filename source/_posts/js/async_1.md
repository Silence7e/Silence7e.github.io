---
title: JS中的异步
date: 2017-07-30 10:46:25
updated: 2017-11-25 09:40:00
categories: 编程
tags: [JS,ES6,前端]
---

ES6发布很久了，提出了很多新的特性，异步方面提供了很多新的功能和方法，你是否还满足于仅用回调函数来控制异步，是否对新的异步控制方式充满好奇，让我们一起来深入学习JS的异步流程控制。
<!--more-->
## 概述

可能提到异步，最先想到的东西就是`回调函数`，然后可能还会想到JQuery中的`Promise`，还有`Bluebird`,其实JS发展到现在，还有很多和异步相关的新兴技术，希望本次分享能带给大家一些收获。

主要内容包括：*回调函数*、*Promise*、*Generator*、*async/await*。
主要取自 [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS)

### 前戏:这段代码的结果是？
```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```
## 一、现在与稍后
想象一下，我们如何表达和操作跨越一段时间的程序行为？

这一段时间是指你的程序`现在`运行的一部分和`稍后`运行的另一部分之间的东西——`现在 和 稍后` 之间有一个间隙，在这个间隙中你的程序并没有在活跃地执行。这里边 `现在与稍后` 的部分之间的关系，就是异步编程的核心。


首先，我们要明白的是，虽然你可能将你的所有JS写在一个.js文件中，但你的程序是有几个代码块组成的，有的`现在`执行，而其他的会在`稍后`执行，最常见的代码块是`function`。
考虑如下代码:
``` javascript
// ajax(..)是某个包中任意的Ajax函数
var data = ajax( "http://some.url.1" );

console.log( data );
// 啊哦！得不到结果。
```
因为Ajax不会同步执行，所以`ajax(..)`还没有返回值给`data`。

所以我们`现在`制造一个ajax异步请求，但`稍后`我们才能得到结果。

从 `现在`等到`稍后`最简单的方法，就是回调函数了：
```javascript
ajax( "http://some.url.1", function callback(data){
    console.log( data ); // Yay, 我得到了一些`data`!
} );
```
再来个例子：

```javascript
function now() {
    return 21;
}

function later() {
	//稍后的部分
    answer = answer * 2;
    console.log( "Meaning of life:", answer );
}

var answer = now();
setTimeout( later, 1000 );
```
这段代码中有两个代码块， later函数中的代码是`稍后`将会运行的部分，其他的是`现在`的部分。

### 事件轮询（Event Loop）
但现在我们需要明白：ES6之前，JS本身没有什么异步的概念。
那这些异步的功能是靠什么来实现的呢？ 这里要提到一个东西叫：`宿主环境`。

最常见的宿主环境就是浏览器，还有NodeJs。他们在每次调用JS引擎时，可以*随着时间的推移*执行程序的多个代码块儿，这被称为`“事件轮询（Event Loop）”`。

JS引擎自己对`时间`没有概念,他们只能让宿主环境来为自己安排"事件"。

举个例子：
>当你向服务器发起一个Ajax请求时，你在一个函数(回调)中写好接收数据的代码，然后JS引擎就会告诉宿主环境：
>"嘿，哥们，我要暂停执行了，但是，当你完成这个网络请求并且得到数据时，请调用这个方法。"
>然后浏览器会为这个网络请求设置个监听：当请求有数据返回时，他会将刚才的函数插入到事件轮询中安排执行。


什么是事件轮询？先来看一段代码：

```javascript
// `eventLoop`是一个像队列一样的数组（先进先出）
var eventLoop = [ ];
var event;

// “永远”执行
while (true) {
	// 执行一个"tick"
	if (eventLoop.length > 0) {
		// 在队列中取得下一个事件
		event = eventLoop.shift();

		// 现在执行下一个事件
		try {
			event();
		}
		catch (err) {
			reportError(err);
		}
	}
}
```
这只是一个简化版的假想代码：

一个通过`while`循环来表现的持续不断的循环，这个循环的每一次迭代称为一个“tick”。在每一个“tick”中，如果队列中有一个事件在等待，它就会被取出执行。这些事件就是你的函数回调。

注意：`setTimeout(..)`不会将你的回调放在事件轮询队列上。它设置一个定时器；当这个定时器超时的时候，环境才会把你的回调放进事件轮询，这样在某个未来的tick中它将会被取出执行。

如果在那时事件轮询队列中已经有了20个事件会怎样？你的回调要等待，它会排到队列最后。

这就解释了为什么`setTimeout(..)`计时器可能不会完美地按照预计时间触发，你的回调不会在你指定的时间间隔之前被触发，但是可能会在这个时间间隔之后被触发。


## 运行至完成

因为JavaScript是单线程的，`foo()`（和`bar()`）中的代码具有原子性，这意味着一旦`foo()`开始运行，它的全部代码都会在`bar()`中的任何代码可以运行之前执行完成，这称为“运行至完成”行为。
考虑下面这段代码：

```javascript
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
因为`foo()`不能被`bar()`打断，而且`bar()`不能被`foo()`打断，所以这个程序根据哪一个先执行只有两种可能的结果（这里`foo()`和`bar()` 都有可能先执行，所有会有两个可能的结果），如果线程存在，`foo()`和`bar()`中的每一个语句都可能被穿插，可能的结果数量将会极大地增长!

### Jobs

在ES6中，在事件轮询队列之上引入了一层新概念，称为“工作队列（Job queue）”。

“工作队列”是一个挂靠在事件轮询队列的每个tick末尾的队列。在事件轮询的一个tick期间内，某些可能发生的隐含异步动作的行为将不会导致一个全新的事件加入事件轮询队列，而是在当前tick的工作队列的末尾加入一个新的记录（也就是一个Job）。

它好像是在说，“哦，另一件需要我 *稍后* 去做的事儿，但是保证它在其他任何事情发生之间发生。”

一个Job还可能会导致更多的Job被加入同一个队列的末尾。所以，一个在理论上可能的情况是，Job“轮询”（一个Job持续不断地加入其他Job等）会无限地转下去，从而拖住程序不能移动到一下一个事件轮询tick。这与在你的代码中表达一个长时间运行或无限循环（比如`while (true) ..`）在概念上几乎是一样的。

Job的精神有点儿像`setTimeout(..0)`黑科技，但以一种定义明确得多的方式实现，而且保证顺序： **稍后，但尽快**。

让我们想象一个用于Job的API，并叫它`schedule(..)`。考虑如下代码：

```javascript
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// 理论上的 "Job API"
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```

你肯能会期望它打印出`A B C D`，但是它将会打出`A C D B`，因为Job发生在当前的事件轮询tick的末尾，而定时器会在 *下一个* 事件轮询tick才会被执行。

##  二、回调

### 什么是回调？
```javascript
// A
ajax( "..", function(..){
	// C
} );
// B
```

`// A`和`// B`代表程序的前半部分（也就是 *现在*），`// C`标识了程序的后半部分（也就是 *稍后*）。前半部分立即执行，然后会出现一个不知多久的“暂停”。在未来某个时刻，如果Ajax调用完成了，那么程序会回到它刚才离开的地方，并 *继续* 执行后半部分。
```javascript
// A
setTimeout( function(){
	// C
}, 1000 );
// B
```
做A，设置一个1000毫秒的定时器，然后做B，然后在超时事件触发后，做C。

### 嵌套/链接的回调
```javascript
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 500) ;
} );
```
这样的代码常被称为“回调地狱（callback hell）”，虽然你可能一眼就能认出这代码的执行顺序：首先，我们等待“click”事件，然后我们等待定时器触发，然后我们等待Ajax应答回来，它很巧合的和写的顺序一致，但如果稍微复杂点：
```javascript
doA( function(){
	doB();

	doC( function(){
		doD();
	} )

	doE();
} );

doF();
```
这些操作将会以这种顺序发生：
* `doA()`
* `doF()`
* `doB()`
* `doC()`
* `doE()`
* `doD()`

可能会有人说是因为嵌套的原因而导致难以推断，当然，有一部分是嵌套的原因。如果我们把上面的代码改为如下：
```javascript
listen( "click", handler );

function handler() {
	setTimeout( request, 500 );
}

function request(){
	ajax( "http://some.url.1", response );
}

function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```
样的代码组织形式几乎看不出来有前一种形式的嵌套/缩进困境，但它的每一处依然容易受到“回调地狱”的影响。为什么呢？

当我们线性地（顺序地）推理这段代码，我们不得不从一个函数跳到下一个函数，再跳到下一个函数，并在代码中弹来弹去以“看到”顺序流。而且这个代码是很简单的，当复杂时，情况会更加恶劣。

另一件需要注意的事是：为了将第2，3，4步链接在一起使他们相继发生，回调独自给我们的启示是将第2步硬编码在第1步中，将第3步硬编码在第2步中，将第4步硬编码在第3步中，如此继续。硬编码不一定是一件坏事，如果第2步应当总是在第3步之前真的是一个固定条件。

不过硬编码绝对会使代码变得更脆弱，因为它不考虑任何可能使在步骤前行的过程中出现偏差的异常情况。举个例子，如果第2步失败了，第3步永远不会到达，第2步也不会重试，或者移动到一个错误处理流程上，等等。

### 信任问题
```javascript
// A
ajax( "..", function(..){
	// C
} );
// B
```
`// A`和`// B`*现在* 发生，在JS主程序的直接控制之下。但是`// C`被推迟到 *稍后* 再发生，并且在另一部分的控制之下——这里是`ajax(..)`函数。在基本的感觉上，这样的控制交接一般不会让程序产生很多问题。

但是不要被这种控制切换不是什么大事的罕见情况欺骗了。事实上，它是回调驱动的设计的最可怕的（也是最微妙的）问题。这个问题围绕着一个想法展开：有时`ajax(..)`（或者说你向之提交回调的部分）不是你写的函数，或者不是你可以直接控制的函数。很多时候它是一个由第三方提供的工具。

当你把你程序的一部分拿出来并把它执行的控制权移交给另一个第三方时，我们称这种情况为“控制倒转”。在你的代码和第三方工具之间有一个没有明言的“契约”——一组你期望被维护的东西。

因为控制倒转可能存在不正常运行的方式：
* 调用回调太早
* 调用回调过晚 (或不调)
* 调用回调太少或太多次（就像你遇到的问题！）
* 没能向你的回调传递必要的参数
* 吞掉了可能发生的错误/异常
* ...
### 尝试拯救回调
举个例子，为了更平静地处理错误，有些API设计提供了分离的回调（一个用作成功的通知，一个用作错误的通知）：
```javascript
function success(data) {
    console.log( data );
}

function failure(err) {
    console.error( err );
}

ajax( "http://some.url.1", success, failure );
```
在这种设计的API中，`failure()`错误处理器通常是可选的，而且如果不提供的话它会假定你想让错误被吞掉，ES6的Promises的API使用的就是这种分离回调设计。

另一种常见的回调设计模式称为“错误优先风格”（“Node风格”），回调的第一个参数为一个错误对象保留（如果有的话）。如果成功，这个参数将会是空/falsy（而其他后续的参数将是成功的数据），但如果出现了错误的结果，这第一个参数就会被设置：

```javascript
function response(err,data) {
	// 有错？
	if (err) {
		console.error( err );
	}
	// 否则，认为成功
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", response );
```

这两种方法都有几件事情应当注意。

首先，它们没有像看起来那样真正解决主要的信任问题。在这两个回调中没有关于防止或过滤意外的重复调用的东西。而且，事情现在更糟糕了，因为你可能同时得到成功和失败信号，或者都得不到，你仍然不得不围绕着这两种情况写代码。

还有，不要忘了这样的事实：虽然它们是你可以引用的标准模式，但它们绝对更加繁冗，而且是不太可能复用的模板代码，所以你将会对在你应用程序的每一个回调中敲出它们感到厌倦。

回调从不被调用的信任问题怎么解决？你可能需要设置一个超时来取消事件。你可以制作一个工具来帮你：
```javascript
function timeoutify(fn,delay) {
	var intv = setTimeout( function(){
			intv = null;
			fn( new Error( "Timeout!" ) );
		}, delay )
	;

	return function() {
		// 超时还没有发生？
		if (intv) {
			clearTimeout( intv );
			fn.apply( this, [ null ].concat( [].slice.call( arguments ) ) );
		}
	};
}
```
这是你如何使用它：

```javascript
// 使用“错误优先”风格的回调设计
function foo(err,data) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", timeoutify( foo, 500 ) );
```
又一个信任问题被“解决了”！但它很低效，而且又有更多臃肿的模板代码让你的项目变得沉重。

这只是关于回调一遍又一遍地发生的故事。它们几乎可以做任何你想做的事，但你不得不努力工作来达到目的，而且大多数时候这种努力比你应当在推理这样的代码上所付出的多得多。

你可能发现自己希望有一些内建的API或语言机制来解决这些问题。终于ES6带着一个伟大的答案到来了。


## 三、Promises

回想一下回调函数中的操作，我们将我们的程序的延续包装进一个回调函数中，将这个回调交给另一个团体（甚至是外部代码），并双手合十祈祷它会做正确的事情并调用这个回调。

但是如果我们能够反向倒转这种 控制倒转 呢？如果不是将我们程序的延续交给另一个团体，而是希望它返回给我们一个可以知道它何时完成的能力，然后我们的代码可以决定下一步做什么呢？

这种规范被称为 Promise。

### 什么是Promise？
举个例子：去麦当劳---点餐---拿到点餐号（许诺）--- 听到叫号去拿汉堡（或者被通知卖完了）。

再举个例子：如何实现获取两个值的和的功能：
```javascript
var x, y = 2;

console.log( x + y ); // NaN  <-- 因为`x`还没有被赋值
```
```javascript
function add(getX,getY,cb) {
	var x, y;
	getX( function(xVal){
		x = xVal;
		// 两者都准备好了？
		if (y != undefined) {
			cb( x + y );	// 发送加法的结果
		}
	} );
	getY( function(yVal){
		y = yVal;
		// 两者都准备好了？
		if (x != undefined) {
			cb( x + y );	// 发送加法的结果
		}
	} );
}

// `fetchX()`和`fetchY()`是同步或异步的函数
add( fetchX, fetchY, function(sum){
	console.log( sum );
} );
```
#### Promise的实现：

```javascript
function add(xPromise,yPromise) {
	// `Promise.all([ .. ])`接收一个Promise的数组，
	// 并返回一个等待它们全部完成的新Promise
	return Promise.all( [xPromise, yPromise] )

	// 当这个Promise被解析后，我们拿起收到的`X`和`Y`的值，并把它们相加
	.then( function(values){
		// `values`是一个从先前被解析的Promise那里收到的消息数组
		return values[0] + values[1];
	} );
}

// `fetchX()`和`fetchY()`分别为它们的值返回一个Promise，
// 这些值可能在 *现在* 或 *稍后* 准备好
add( fetchX(), fetchY() )

// 为了将两个数字相加，我们得到一个Promise。
// 现在我们链式地调用`then(..)`来等待返回的Promise被解析
.then( function(sum){
	console.log( sum ); // 这容易多了！
} );
```
#### 完成事件

我们对获取X、Y的细节一无所知，我们也不关心。它可能会立即完成任务，也可能会花一段时间完成。

我们仅仅想简单地知道获取什么时候完成，以便于我们可以移动到下一个任务（求和）。换句话说，我们想要一种方法被通知他们的完成，以便于我们可以 *继续*。

在典型的JavaScript风格中，如果你需要监听一个通知，你很可能会想到事件（event）。那么我们可以将我们的通知需求重新表述为，监听由`foo(..)`发出的 *完成*（或 *继续*）事件。

使用回调，“通知”就是被任务（`foo(..)`）调用的我们的回调函数，但是使用Promise，我们将关系扭转过来，我们希望能够监听一个来自于`foo(..)`的事件，当我们被通知时，做相应的处理。

考虑一段代码：
```javascript
function foo(x) {
	// 开始做一些可能会花一段时间的事情

	// 制造一个`listener`事件通知能力并返回

	return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
	// 现在我们可以做下一步了！
} );

evt.on( "failure", function(err){
	// 噢，在`foo(..)`中有某些事情搞错了
} );
```
```javascript
var evt = foo( 42 );

// 让`bar(..)`监听`foo(..)`的完成
bar( evt );

// 同时，让`baz(..)`监听`foo(..)`的完成
baz( evt );
```
回调本身代表着一种 *控制反转*。所以反转回调模式实际上是 *反转的反转*，或者说是一个 *控制非反转*——将控制权归还给我们希望保持它的调用方代码，

控制非反转 导致了更好的 关注分离，也就是bar(..)和baz(..)不必卷入foo(..)是如何被调用的问题。相似地，foo(..)也不必知道或关心bar(..)和baz(..)的存在或它们是否在等待foo(..)完成的通知。
这里的evt其实就是一个Promise的类比。

那么用了Promise，foo、bar、baz内部分别是什么样的呢？

```javascript
function foo(x) {
	// 开始做一些可能会花一段时间的事情

	// 构建并返回一个promise
	return new Promise( function(resolve,reject){
		// 最终需要调用`resolve(..)`或`reject(..)`
		// 它们是这个promise的解析回调
	} );
}

var p = foo( 42 );

bar( p );

baz( p );
```
bar和baz方法内部:
```javascript
function bar(fooPromise) {
	// 监听`foo(..)`的完成
	fooPromise.then(
		function(){
			// `foo(..)`现在完成了，那么做`bar(..)`的任务
		},
		function(){
			// 噢，在`foo(..)`中有某些事情搞错了
		}
	);
}
```
当然我们也可以使用另一种方法：
```javascript
function bar() {
	// `foo(..)`绝对已经完成了，那么做`bar(..)`的任务
}

function oopsBar() {
	// 噢，在`foo(..)`中有某些事情搞错了，那么`bar(..)`不会运行
}

// `baz()`和`oopsBaz()`同上

var p = foo( 42 );

p.then( bar, oopsBar );

p.then( baz, oopsBaz );
```

**注意：** `p.then( .. ).then( .. )`和`p.then(..); p.then(..)`是不一样的效果。

### Promise的信任

#### 调的太早

这种顾虑主要是一个任务有时会同步完地成，而有时会异步地完成，这将导致竟合状态。

Promise被定义为不能受这种顾虑的影响，因为即便是立即完成的Promise（比如 `new Promise(function(resolve){ resolve(42); })`）也不可能被同步地 *监听*。

也就是说，当你在Promise上调用`then(..)`的时候，即便这个Promise已经被解析了，你给`then(..)`提供的回调也将 **总是** 被异步地调用。

所以不必再插入你自己的`setTimeout(..,0)`黑科技了。
```javascript
var log = console.log;

function MyPromise(fun){
  var deferred = $.Deferred();
  fun(deferred.resolve, deferred.reject);
  return deferred.promise();
}

setTimeout(function () {
    log(1);
});

new MyPromise(function(resolve, reject){
  log(2);
  resolve();
  log(3);
}).then(function(){
  log(4);
});

log(5);
```

#### 调的太晚

和前一点相似，在`resolve(..)`或`reject(..)`被Promise创建机制调用时，一个Promise的`then(..)`上注册的监听回调将自动地被排程。这些被排程好的回调将在下一个异步时刻被可预测地触发。

同步监听是不可能的，所以不可能有一个同步的任务链的运行来“推迟”另一个回调的发生。也就是说，当一个Promise被解析时，所有在`then(..)`上注册的回调都将被立即，按顺序地，在下一个异步机会时被调用，而且没有任何在这些回调中发生的事情可以影响/推迟其他回调的调用。

举例来说：

```js
p.then( function(){
	p.then( function(){
		console.log( "C" );
	} );
	console.log( "A" );
} );
p.then( function(){
	console.log( "B" );
} );
// A B C
```
`"C"`不可能干扰并优先于`"B"`。

#### 根本不调回调

这是一个很常见的顾虑。Promise用几种方式解决它。

首先，没有任何东西（JS错误都不能）可以阻止一个Promise通知你它的解析（如果它被解析了的话）。如果你在一个Promise上同时注册了完成和拒绝回调，而且这个Promise被解析了，两个回调中的一个总会被调用。

当然，如果你的回调本身有JS错误，你可能不会看到你期望的结果，但是回调事实上已经被调用了。我们待会儿就会讲到如何在你的回调中收到关于一个错误的通知，因为就算是它们也不会被吞掉。

那如果Promise本身不管怎样永远没有被解析呢？即便是这种状态Promise也给出了答案，使用一个称为“竞赛（race）”的高级抽象。

```js
// 一个使Promise超时的工具
function timeoutPromise(delay) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			reject( "Timeout!" );
		}, delay );
	} );
}

// 为`foo()`设置一个超时
Promise.race( [
	foo(),					// 尝试调用`foo()`
	timeoutPromise( 3000 )	// 给它3秒钟
] )
.then(
	function(){
		// `foo(..)`及时地完成了！
	},
	function(err){
		// `foo()`不是被拒绝了，就是它没有及时完成
		// 那么可以考察`err`来知道是哪种情况
	}
);
```
我们可以确保一个信号作为`foo(..)`的结果，来防止它无限地挂起我们的程序。

#### 调太少或太多次

根据定义，对于被调用的回调来讲 *一次* 是一个合适的次数。“太少”的情况将会是0次，和我们刚刚考察的从不调用是相同的。

“太多”的情况则很容易解释，但Promise被定义为只能被解析一次。如果因为某些原因，Promise的创建代码试着调用`resolve(..)`或`reject(..)`许多次，或者试着同时调用它们俩，Promise将仅接受第一次解析，而无声地忽略后续的尝试。

因为一个Promise仅能被解析一次，所以任何`then(..)`上注册的（每个）回调将仅仅被调用一次。

当然，如果你把同一个回调注册多次（比如`p.then(f); p.then(f);`），那么它就会被调用注册的那么多次。

#### 没能传入任何参数

Promise可以拥有最多一个解析值（完成或拒绝）。

如果无论怎样你没有用一个值明确地解析它，它的值就是`undefined`。

如果你使用多个参数调用`resolve(..)`或`reject(..)`，第一个参数之外的后续参数都会被忽略。
如果你想传递多个值，你必须将它们包装在另一个单独的值中，比如一个`array`或一个`object`。

#### 吞掉所有错误/异常

如果你用一个 *理由*（也就是错误消息）拒绝一个Promise，这个值就会被传入拒绝回调。
但是这里有一个更重要的事情。如果在Promise的创建过程中的任意一点，或者在监听它的解析的过程中，一个JS异常错误发生的话，这个异常将会被捕获，并且强制当前的Promise变为拒绝。

举例来说：

```js
var p = new Promise( function(resolve,reject){
	foo.bar();	// `foo`没有定义，所以这是一个错误！
	resolve( 42 );	// 永远不会跑到这里 :(
} );

p.then(
	function fulfilled(){
		// 永远不会跑到这里 :(
	},
	function rejected(err){
		// `err`将是一个来自`foo.bar()`那一行的`TypeError`异常对象
	}
);
```

在`foo.bar()`上发生的JS异常变成了一个你可以捕获并响应的Promise拒绝。

这是一个重要的细节，错误可能会产生一个同步的反应，而没有错误的部分还是异步的，Promise甚至将JS异常都转化为异步行为。

但是如果Promise完成了，但是在监听过程中（在一个`then(..)`上注册的回调上）出现了JS异常错误会怎样呢？即便是那些也不会丢失，但你可能会发现处理它们的方式有些令人诧异，除非你深挖一些：

```js
var p = new Promise( function(resolve,reject){
	resolve( 42 );
} );

p.then(
	function fulfilled(msg){
		foo.bar();
		console.log( msg );	// 永远不会跑到这里 :(
	},
	function rejected(err){
		// 也永远不会跑到这里 :(
	}
);
```

等一下，这看起来`foo.bar()`发生的异常确实被吞掉了。不要害怕，它没有。但更深层次的东西出问题了，也就是我们没能成功地监听他。`p.then(..)`调用本身返回另一个promise，是 *那个* promise将会被`TypeError`异常拒绝。

为什么它不能调用我们在这里定义的错误处理器呢？表面上看起来是一个符合逻辑的行为。但它会违反Promise一旦被解析就 **不可变** 的基本原则。`p`已经完成为值`42`，所以它不能因为在监听`p`的解析时发生了错误，而在稍后变成一个拒绝。

除了违反原则，这样的行为还可能造成破坏，假如说有多个在promise`p`上注册的`then(..)`回调，因为有些会被调用而有些不会，而且至于为什么是很明显的。

### 可信的Promise？

为了基于Promise模式建立信任，还有最后一个细节需要考察。

Promise根本没有摆脱回调。它们只是改变了回调传递的位置。与将一个回调传入`foo(..)`相反，我们从`foo(..)`那里拿回 *某些东西* （表面上是一个纯粹的Promise），然后我们将回调传入这个 *东西*。

但为什么这要比仅使用回调的方式更可靠呢？我们如何确信我们拿回来的 *某些东西* 事实上是一个可信的Promise？

一个Promise经常被忽视，但是最重要的细节之一，就是它也为这个问题给出了解决方案。包含在原生的ES6`Promise`实现中，它就是`Promise.resolve(..)`。

如果你传递一个立即的，非Promise的，非thenable的值给`Promise.resolve(..)`，你会得到一个用这个值完成的promise。换句话说，下面两个promise`p1`和`p2`的行为基本上完全相同：

```js
var p1 = new Promise( function(resolve,reject){
	resolve( 42 );
} );

var p2 = Promise.resolve( 42 );
```

但如果你传递一个纯粹的Promise给`Promise.resolve(..)`，你会得到这个完全相同的promise：

```js
var p1 = Promise.resolve( 42 );

var p2 = Promise.resolve( p1 );

p1 === p2; // true
```

更重要的是，如果你传递一个非Promise的thenable值给`Promise.resolve(..)`，它会试着将这个值展开，而且直到抽出一个最终具体的非Promise值之前，展开操作将会一直继续下去。

考虑这段代码：

```js
var p = {
	then: function(cb) {
		cb( 42 );
	}
};

p.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// 永远不会跑到这里
	}
);
```

这个`p`是一个thenable，但它不是一个纯粹的Promise。很走运，它是合理的，所以不会报错。

但是如果你得到的是看起来像这样的东西：


```js
var p = {
	then: function(cb,errcb) {
		cb( 42 );
		errcb( "evil laugh" );
	}
};

p
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// 噢，这里本不该运行
		console.log( err ); // evil laugh
	}
);
```

这个`p`也是一个thenable，但它不是表现良好的promise。它是恶意的吗？或者它只是不知道Promise应当如何工作？老实说，这不重要。不管哪种情况，它都不那么可靠。

尽管如此，我们可以将这两个版本的`p`传入`Promise.resolve(..)`，而且我们将会得到一个我们期望的泛化，安全的结果：

```js
Promise.resolve( p )
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// 永远不会跑到这里
	}
);
```

`Promise.resolve(..)`会接受任何thenable，而且将它展开直至非thenable值。但你会从`Promise.resolve(..)`那里得到一个真正的，纯粹的Promise，**一个你可以信任的东西**。如果你传入的东西已经是一个纯粹的Promise了，那么你会单纯地将它拿回来，所以通过`Promise.resolve(..)`过滤来得到信任没有任何坏处。

那么我们假定，我们在调用一个`foo(..)`工具，而且不能确定我们能相信它的返回值是一个行为规范的Promise，但我们知道它至少是一个thenable。`Promise.resolve(..)`将会给我们一个可靠的Promise包装器来进行链式调用：

```js
// 不要只是这么做：
foo( 42 )
.then( function(v){
	console.log( v );
} );

// 相反，这样做：
Promise.resolve( foo( 42 ) )
.then( function(v){
	console.log( v );
} );
```

**注意：** 将任意函数的返回值（thenable或不是thenable）包装在`Promise.resolve(..)`中的另一个好的副作用是，它可以很容易地将函数调用泛化为一个行为规范的异步任务。如果`foo(42)`有时返回一个立即值，而其他时候返回一个Promise，`Promise.resolve(foo(42))`，将确保它总是返回Promise。


#### 回顾前戏的代码：[流程](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```


## 四、 Generator

在前面的部分中，我们发现了在使用回调表达异步流程控制时的两个关键缺陷：

* 基于回调的异步与我们的大脑规划任务(顺序)各个步骤的过程不相符。
* 由于 *控制倒转* 回调是不可靠的。

之前我们详细地讨论了Promise如何反转回调的 *控制倒转*，重建了可靠性/可组合性。

现在让我们把注意力集中到用一种顺序的，看起来同步的风格来表达异步流程控制。使这一切成为可能的是ES6的 **generator**。

### 打破运行至完成

之前我们提到一个认识：一旦函数开始执行，它将运行直至完成，没有其他的代码可以在运行期间干扰它。

现在ES6引入了一种新型的函数，它不按照“运行至完成”的行为进行动作。这种新型的函数称为“generator（生成器）”。

为了理解它的含义，让我们看看这个例子：

```javascript
var x = 1;

function foo() {
	x++;
	bar();				// <-- 这一行会发生什么？
	console.log( "x:", x );
}

function bar() {
	x++;
}

foo();					// x: 3
```

在这个例子中，我们确信`bar()`会在`x++`和`console.log(x)`之间运行。

要是`bar()`不存在，但以某种方式依然可以在`x++`和`console.log(x)`语句之间运行呢？

JS不是抢占式的，也（还）不是多线程的。但是，如果`foo()`本身可以用某种办法在代码的这一部分指示一个“暂停”，那么这种“干扰”（并发）的形式就是可能的。


这就是实现这种操作的ES6代码：

```javascript
var x = 1;

function *foo() {
	x++;
	yield; // 暂停！
	console.log( "x:", x );
}

function bar() {
	x++;
}
```

**注意：** 一个generator的声明为`function* foo() { .. }`或者`function *foo() { .. }`——唯一的区别是摆放`*`位置的风格。这两种形式在功能性/语法上是完全一样的，还有第三种`function*foo() { .. }`（没空格）形式。

我们该如何运行上面的代码，使`bar()`在`yield`那一点取代`*foo()`的执行？

```js
// 构建一个迭代器`it`来控制generator
var it = foo();

// 在这里开始`foo()`！
it.next();
x;						// 2
bar();
x;						// 3
it.next();				// x: 3
```

1. `it = foo()`操作 *不会* 执行`*foo()`generator，它只不过构建了一个用来控制它执行的 *迭代器（iterator）*。
2. 第一个`it.next()`启动了`*foo()`generator，并且运行`*foo()`第一行上的`x++`。
3. `*foo()`在`yield`语句处暂停，就在这时第一个`it.next()`调用结束。在这个时刻，`*foo()`依然运行而且是活动的，但是处于暂停状态。
4. 我们观察`x`的值，现在它是`2`.
5. 我们调用`bar()`，它再一次用`x++`递增`x`。
6. 我们再一次观察`x`的值，现在它是`3`。
7. 最后的`it.next()`调用使`*foo()`generator从它暂停的地方继续运行，而后运行使用`x`的当前值`3`的`console.log(..)`语句。

清楚的是，`*foo()`启动了，但 *没有* 运行到底——它停在`yield`。我们稍后继续`*foo()`，让它完成，但这甚至不是必须的。

所以，一个generator是一种函数，它可以开始和停止一次或多次，甚至没必要一定要完成。

#### 输入和输出

generator函数是一种有上述功能的函数。但它仍然是一个函数，它可以接收参数（也就是“输入”），而且它依然返回一个值（也就是“输出”）：

```js
function *foo(x,y) {
	return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;		// 42
```

我们将`6`和`7`分别作为参数`x`和`y`传递给`*foo(..)`。而`*foo(..)`将值`42`返回给调用端代码。

现在我们可以看到发生器的调用和一般函数的调用的一个不同之处了。`foo(6,7)`显然看起来很熟悉。但`*foo(..)`generator不会像一个函数那样实际运行起来。

我们只是创建了 *迭代器* 对象，将它赋值给变量`it`，来控制`*foo(..)`generator。当我们调用`it.next()`时，它指示`*foo(..)`generator从现在的位置向前推进，直到下一个`yield`或者generator的最后。

`next(..)`调用的结果是一个带有`value`属性的对象，它持有从`*foo(..)`返回的任何值（如果有的话）。换句话说，`yield`导致在generator运行期间，一个值被从中发送出来，有点儿像一个中间的`return`。

但是，为什么我们需要这个完全间接的 *迭代器* ？

#### 迭代通信

generator除了接收参数和拥有返回值，它们还内建有更强大，更吸引人的输入/输出消息能力，这是通过使用`yield`和`next(..)`实现的。

考虑下面的代码：

```js
function *foo(x) {
	var y = x * (yield);
	return y;
}

var it = foo( 6 );

// 开始`foo(..)`
it.next();

var res = it.next( 7 );

res.value;		// 42
```

首先，我们将`6`作为参数`x`传入。之后我们调用`it.next()`，它启动了`*foo(..)`.

在`*foo(..)`内部，`var y = x ..`语句开始被处理，但它运行到了一个`yield`表达式。就在这时，它暂停了`*foo(..)`，而且请求调用端代码为`yield`表达式提供一个结果值。接下来，我们调用`it.next(7)`，将`7`这个值传回去作为暂停的`yield`表达式的结果。

所以，在这个时候，赋值语句实质上是`var y = 6 * 7`。现在，`return y`将值`42`作为结果返回给`it.next( 7 )`调用。

看起来在`yield`和`next(..)`调用之间似乎存在着错位。一般来说，你所拥有的`next(..)`调用的数量，会比你所拥有的`yield`语句的数量多一个——前面的代码段中有一个`yield`和两个`next(..)`调用。

为什么会有这样的错位？

因为第一个`next(..)`总是启动一个generator，然后运行至第一个`yield`。但是第二个`next(..)`调用满足了第一个暂停的`yield`表达式，而第三个`next(..)`将满足第二个`yield`，如此反复。

##### 两个疑问的故事

实际上，你主要考虑的是哪部分代码会影响你是否感知到错位。

仅考虑generator代码：

```js
var y = x * (yield);
return y;
```

这 **第一个** `yield`基本上是在 *问一个问题*：“我应该在这里插入什么值？”

谁来回答这个问题？好吧，**第一个** `next()`在这个时候已经为了启动generator而运行过了，所以很明显 *它* 不能回答这个问题。所以，**第二个** `next(..)`调用必须回答由 **第一个** `yield`提出的问题。

看到错位了吧——第二个对第一个？

但是让我们反转一下我们的角度。让我们不从generator的角度看问题，而从迭代器的角度看。

**注意**：消息可以双向发送——`yield ..`作为表达式可以发送消息来应答`next(..)`调用，而`next(..)`可以发送值给暂停的`yield`表达式。

考虑一下这段稍稍调整过的代码：

```javascript
function *foo(x) {
	var y = x * (yield "Hello");	// <-- 让出一个值！
	return y;
}

var it = foo( 6 );

var res = it.next();	// 第一个`next()`，不传递任何东西
res.value;				// "Hello"

res = it.next( 7 );		// 传递`7`给等待中的`yield`
res.value;				// 42
```

`yield ..`和`next(..)`一起成对地 **在generator运行期间** 构成了一个双向消息传递系统。

那么，如果只看 *迭代器* 代码：

```js
var res = it.next();	// 第一个`next()`，不传递任何东西
res.value;				// "Hello"

res = it.next( 7 );		// 传递`7`给等待中的`yield`
res.value;				// 42
```

我们没有传递任何值给第一个`next()`调用，因为只有一个暂停的`yield`才能接收被`next(..)`传递的值，但是当我们调用第一个`next()`时，在generator的最开始并 **没有任何暂停的`yield`** 可以接收这样的值。 任何传入第一个`next()`的东西会被 **丢弃** 。所以总是用一个无参数的`next()`来启动generator。

第一个`next()`调用（没有任何参数的）基本上是在 *问一个问题*：“`*foo(..)`generator将要给我的 *下一个* 值是什么？”，谁来回答这个问题？第一个`yield`表达式。

看到了？这里没有错位。

但等一下！跟`yield`语句的数量比起来，还有一个额外的`next()`。那么，这个最后的`it.next(7)`调用又一次在询问generator *下一个* 产生的值是什么。但是没有`yield`语句剩下可以回答了，不是吗？那么谁来回答？

`return`语句回答这个问题！

而且如果在你的generator中 **没有`return`**——则会有一个假定/隐式的`return;`（也就是`return undefined;`），它默认的目的就是回答由最后的`it.next(7)`调用 *提出* 的问题。

这些问题与回答——用`yield`和`next(..)`进行双向消息传递——十分强大，但现在我们似乎还看不出来这些机制与异步流程控制有什么联系。

### 多迭代器

从语法使用上来看，当你用一个 *迭代器* 来控制generator时，你正在控制声明的generator函数本身。但这里有一个容易忽视的微妙细节：每当你构建一个 *迭代器*，你都隐含地构建了一个将由这个 *迭代器* 控制的generator的实例。

你可以让同一个generator的多个实例同时运行，它们甚至可以互动：

```js
function *foo() {
	var x = yield 2;
	z++;
	var y = yield (x * z);
	console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;			// 2 <-- 让出2
var val2 = it2.next().value;			// 2 <-- 让出2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
										// 20 300 3
it2.next( val1 / 4 );					// y:10
										// 200 10 3
```

处理过程：

1. 两个`*foo()`在同时启动，而且两个`next()`都分别从`yield 2`语句中得到了`2`的`value`。
2. `val2 * 10`就是`2 * 10`，它被发送到第一个generator实例`it1`，所以`x`得到值`20`。`z`将`1`递增至`2`，然后`20 * 2`被`yield`出来，将`val1`设置为`40`。
3. `val1 * 5`就是`40 * 5`，它被发送到第二个generator实例`it2`中，所以`x`得到值`200`。`z`又一次递增，从`2`到`3`，然后`200 * 3`被`yield`出来，将`val2`设置为`600`。
4. `val2 / 2`就是`600 / 2`，它被发送到第一个generator实例`it1`，所以`y`得到值`300`，然后分别为它的`x y z`值打印出`20 300 3`。
5. `val1 / 4`就是`40 / 4`，它被发送到第一个generator实例`it2`，所以`y`得到值`10`，然后分别为它的`x y z`值打印出`200 10 3`。

这是在你脑海中跑过的一个“有趣”的例子。你还能保持清醒？

#### 穿插

之前我们提过一个“运行至完成”的概念：

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}
```

使用普通的JS函数，当然要么是`foo()`可以首先运行完成，要么是`bar()`可以首先运行至完成，但是`foo()`不可能与`bar()`穿插它的独立语句。所以，前面这段代码只有两个可能的结果。

然而，使用generator，就可以明确地穿插了：

```js
var a = 1;
var b = 2;

function *foo() {
	a++;
	yield;
	b = b * a;
	a = (yield b) + 3;
}

function *bar() {
	b--;
	yield;
	a = (yield 8) + b;
	b = a * (yield 2);
}
```

根据 *迭代器* 控制`*foo()`与`*bar()`分别以什么样的顺序被调用，前面这段代码可以产生几种不同的结果。


### 发生器与迭代器
`generator`有一种用法，作为一种生产值的方式，也因为这种用法，给它起了个名字：生成器。

想象你正在生产一系列的值，它们中的每一个都与前一个值有可定义的关系。为此，你将需要一个有状态的发生器来记住上一个给出的值，可以用闭包来直接地实现这样的东西：

```javascript
var gimmeSomething = (function(){
	var nextVal;

	return function(){
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		return nextVal;
	};
})();

gimmeSomething();		// 1
gimmeSomething();		// 9
gimmeSomething();		// 33
gimmeSomething();		// 105
```

直到 *下一次* `gimmeSomething()`调用发生之前，我们不会计算 *下一个值*（也就是`nextVal`），可以节省资源。

这种任务可以用迭代器解决。一个 *迭代器* 是一个明确定义的接口，用来逐个从发生器得到一系列的值。迭代器的JS接口，和大多数语言一样，是在你每次想从发生器中得到下一个值时调用的`next()`。

我们可以为刚才的数字序列发生器实现标准的 *迭代器*；

```javascript
var something = (function(){
	var nextVal;

	return {
		// `for..of`循环需要这个
		[Symbol.iterator]: function(){ return this; },

		// 标准的迭代器接口方法
		next: function(){
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			return { done:false, value:nextVal };
		}
	};
})();

something.next().value;		// 1
something.next().value;		// 9
something.next().value;		// 33
something.next().value;		// 105
```

`[ .. ]`语法称为一个 *计算型属性名*，`Symbol.iterator`是ES6预定义的特殊`Symbol`值。

`next()`调用返回一个对象，它带有两个属性：`done`是一个`boolean`值表示 *迭代器* 的完成状态；`value`持有迭代的值。

ES6还增加了`for..of`循环，它意味着一个标准的 *迭代器* 可以使用原生的循环语法来自动地被消费：

```javascript
for (var v of something) {
	console.log( v );

	// 不要让循环永无休止！
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

因为我们的`something`迭代器总是返回`done:false`，这个`for..of`循环将会永远运行，所以我们需要`break`。

`for..of`循环为每一次迭代自动调用`next()`，不会给`next()`传值，他在收到一个`done:true`时自动终结。

你也可以手动循环一个迭代器，调用`next()`并检查`done:true`条件来知道什么时候停止：

```javascript
for (
	var ret;
	(ret = something.next()) && !ret.done;
) {
	console.log( ret.value );

	// 不要让循环永无休止！
	if (ret.value > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

这种手动的`for`方式当然要比ES6的`for..of`循环语法难看，但它的好处是它提供给你一个机会，在有必要时传值给`next(..)`调用。

除了制造你自己的 *迭代器* 之外，许多JS中（就ES6来说）内建的数据结构，比如`array`，也有默认的 *迭代器*：

```js
var a = [1,3,5,7,9];

for (var v of a) {
	console.log( v );
}
// 1 3 5 7 9
```

`for..of`循环向`a`要来它的迭代器，并自动使用它迭代`a`的值。

普通的`object`没有像`array`那样的默认 *迭代器*。如果你想要迭代一个对象的属性（不保证顺序），`Object.keys(..)`返回一个`array`，它可以像`for (var k of Object.keys(obj)) { ..`这样使用。像这样用`for..of`循环一个对象上的键，与用`for..in`循环内很相似，除了在`for..in`中会包含`[[Prototype]]`链的属性，而`Object.keys(..)`不会。

### Iterables

在我们运行的例子中的`something`对象被称为一个 *迭代器*，因为它的接口中有`next()`方法。一个紧密关联的术语是 *iterable*，它指 **包含有** 一个可以迭代它所有值的迭代器的对象。

在ES6中，从一个 *iterable* 中取得一个 *迭代器* 的方法是，*iterable* 上必须有一个函数，它的名称是特殊的ES6符号值`Symbol.iterator`。当这个函数每次被调用时，它就会返回一个 全新的*迭代器*。

前一个代码段的`a`就是一个 *iterable*。`for..of`循环自动地调用它的`Symbol.iterator`函数来构建一个 *迭代器*。我们可以手动地调用这个函数，然后使用它返回的 *iterator*：

```javascript
var a = [1,3,5,7,9];

var it = a[Symbol.iterator]();

it.next().value;	// 1
it.next().value;	// 3
it.next().value;	// 5
..
```

在前面定义`something`的代码段中，有这一行：

```javascript
[Symbol.iterator]: function(){ return this; }
```

这段代码制造了`something`值——`something`*迭代器* 的接口——也是一个 *iterable*；现在它既是一个 *iterable* 也是一个 *迭代器*。然后，我们把`something`传递给`for..of`循环：

```js
for (var v of something) {
	..
}
```

`for..of`循环期待`something`是一个 *iterable*，所以它会寻找并调用它的`Symbol.iterator`函数。我们将这个函数定义为简单地`return this`，所以它将自己给出，而`for..of`不会知道这些。

#### Generator迭代器

generator可以被看做一个值的发生器，我们通过一个 *迭代器* 接口的`next()`每次从中取一个值。

一个generator本身在技术上讲并不是一个 *iterable*，虽然很相似——当你执行generator时，你就得到一个 *迭代器*：

```js
function *foo(){ .. }

var it = foo();
```

我们可以用generator实现早前的`something`无限数字序列发生器，就像这样：

```javascript
function *something() {
	var nextVal;

	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		yield nextVal;
	}
}
```

在一个真实的JS程序中含有一个`while..true`循环通常是一件非常不好的事情，至少如果它没有一个`break`或`return`语句，它就很可能永远运行，并同步地，阻塞/锁定浏览器UI。

然而，在generator中，如果循环含有`yield`，那它就是完全没有问题的，因为generator将在每次迭代后暂停，`yield`回主程序。因为generator会暂停在每个`yield`，`*something()`函数的状态（作用域）被保持着，这就没有必要用闭包来保留变量的状态了。

它更清晰地表达了意图。比如，`while..true`循环告诉我们这个generator将要永远运行——只要我们一直向它请求，它就一直 *产生* 值。

现在我们可以在`for..of`循环中使用新的`*something()`generator了，而且你会看到它工作起来基本一模一样：

```javascript
for (var v of something()) {
	console.log( v );

	// 不要让循环永无休止！
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

不要跳过`for (var v of something()) ..`！我们不仅仅像之前的例子那样将`something`作为一个值引用了，而是调用`*something()`generator来得到它的 *迭代器*，并交给`for..of`使用。

如果你仔细观察，在这个generator和循环的互动中，你可能会有两个疑问：

* 为什么我们不能说`for (var v of something) ..`？因为这里的`something`是一个`generator`，而不是一个 *iterable*。我们不得不调用`something()`来构建一个发生器给`for..of`，以便它可以迭代。
*
* `something()`调用创建一个 *迭代器*，但是`for..of`想要一个 *iterable*，对吧？对，generator的 *迭代器* 上也有一个`Symbol.iterator`函数，这个函数基本上就是`return this`，就像我们刚才定义的`something`*iterable*。也就是说generator的 *迭代器* 也是一个 *iterable*！

#### 停止Generator

看起来在循环的`break`被调用后，`*something()`generator的 *迭代器* 实例基本上被留在了一个永远挂起的状态。

其实这里有一个隐藏的行为为你处理这件事：`for..of`循环的“异常完成”(般是由`break`，`return`，或未捕捉的异常导致的)会向generator的 *迭代器* 发送一个信号，以使它终结。

有时你可能会希望手动发送信号给一个 *迭代器*；你可以通过调用`return(..)`来这么做。

如果你在generator内部指定一个`try..finally`从句，它将总是被执行，即便是generator从外部被完成：

```javascript
function *something() {
	try {
		var nextVal;

		while (true) {
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
				nextVal = (3 * nextVal) + 6;
			}

			yield nextVal;
		}
	}
	// 清理用的从句
	finally {
		console.log( "cleaning up!" );
	}
}
```

前面那个在`for..of`中带有`break`的例子将会触发`finally`从句。但是你可以用`return(..)`从外部来手动终结generator的 *迭代器* 实例：

```js
var it = something();
for (var v of it) {
	console.log( v );

	// 不要让循环永无休止！
	if (v > 500) {
		console.log(
			// 使generator得迭代器完成
			it.return( "Hello World" ).value
		);
		// 这里不需要`break`
	}
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

当我们调用`it.return(..)`时，它会立即终结generator，从而运行`finally`从句。而且，它会将返回的`value`设置为你传入`return(..)`的任何东西，这就是`Hellow World`如何立即返回来的。

现在也不必再包含一个`break`，因为generator的 *迭代器* 会被设置为`done:true`，所以`for..of`循环会在下一次迭代时终结。

#### 异步地迭代Generator

现在我们更加全面地了解了generator的机制是如何工作的, 但它是怎样处理异步模式，解决回调等类似的问题？

回想一下这个回调方式：

```javascript
function foo(x,y,cb) {
	ajax("http://some.url.1/?x=" + x + "&y=" + y, cb);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}else {
		console.log( text );
	}
} );
```

如果我们想用generator表示相同的任务流控制，我们可以：

```javascript
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// 向`*main()`中扔进一个错误
				it.throw( err );
			}else {
				// 使用收到的`data`来继续`*main()`
				it.next( data );
			}
		}
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();
// 使一切开始运行！
it.next();
```

一眼看上去，这个代码段要比以前的回调代码更长，而且也许看起来更复杂。但其实generator部分的代码段实际上要好 **太多** 了！

```js
var text = yield foo( 11, 31 );
console.log( text );
```

我们调用了一个函数`foo(..)`，而且我们显然可以从Ajax调用那里得到`text`，即便它是异步的。这怎么可能？我们有一个几乎完全一样的代码：

```js
var data = ajax( "..url 1.." );
console.log( data );
```
不同点就在于：generator中使用的`yield`。

它允许我们拥有一个看起来是阻塞的，同步的，但实际上不会阻塞整个程序的代码；它仅仅暂停/阻塞在generator本身的代码。

在`yield foo(11,31)`中，首先`foo(11,31)`调用被发起，它什么也不返回（也就是`undefined`），所以我们发起了数据请求，我们实际上做的是`yield undefined`。没有用`yield`传递消息。

那么，generator暂停在了`yield`，它实质上在问一个问题，“我该将什么值返回并赋给变量`text`？”谁来回答这个问题？

看一下`foo(..)`。如果Ajax请求成功，我们调用：

```javascript
it.next( data );
```

这行代码让generator使用ajax应答的数据继续运行，暂停的`yield`表达式收到这个值，然后重新开始运行generator代码，这个值被赋给本地变量`text`。

有没有觉得狂拽炫酷吊炸天？

我们在generator内部的代码看起来完全是同步的（除了`yield`关键字本身），但隐藏在幕后的是，在`foo(..)`内部，操作可以完全是异步的。

我们现在用以一种顺序，同步的风格表达异步处理。

实质上，我们将异步处理作为实现细节抽象出去，以至于我们可以同步地/顺序地推理我们的流程控制：“发起Ajax请求，然后在它完成之后打印应答。” 当然，我们仅仅在这个流程控制中表达了两个步骤，但同样的能力可以无边界地延伸，让我们需要表达多少步骤，就表达多少。

#### 同步错误处理

前面的generator代码会 让出更多的好处给我们。让我们把注意力移到generator内部的`try..catch`上：

```js
try {
	var text = yield foo( 11, 31 );
	console.log( text );
}
catch (err) {
	console.error( err );
}
```

`foo(..)`调用是异步完成的，`try..catch`可以捕捉这里的错误吗？

我们已经看到了`yield`如何让赋值语句暂停，来等待`foo(..)`去完成，以至于完成的响应可以被赋予`text`。牛X的是，`yield`暂停 *还* 允许generator来`catch`一个错误。在前面的例子，用这一部分代码将这个错误抛出到generator中：

```javascript
if (err) {
	// 向`*main()`中扔进一个错误
	it.throw( err );
}
```

generator的`yield`暂停特性不仅意味着我们可以从异步的函数调用那里得到看起来同步的`return`值，还意味着我们可以同步地捕获这些异步函数调用的错误！

那么我们看到了，我们可以将错误 *抛入* generator，但是如何将错误 *抛出* 一个generator呢？

```javascript
function *main() {
	var x = yield "Hello World";

	yield x.toLowerCase();	// 引发一个异常！
}

var it = main();

it.next().value;			// Hello World

try {
	it.next( 42 );
}
catch (err) {
	console.error( err );	// TypeError
}
```

当然，这里也可以用`throw ..`手动地抛出一个错误，而不是制造一个异常。

我们甚至可以`catch`我们`throw(..)`进generator的同一个错误，实质上给了generator一个机会来处理它，但如果generator没处理，那么 *迭代器* 代码必须处理它：

```javascript
function *main() {
	var x = yield "Hello World";

	// 永远不会跑到这里
	console.log( x );
}

var it = main();

it.next();

try {
	// `*main()`会处理这个错误吗？我们走着瞧！
	it.throw( "Oops" );
}
catch (err) {
	// 不，它没处理！
	console.error( err );			// Oops
}
```

使用异步代码的，看似同步的错误处理，在可读性和可推理性上大获全胜。

### Generators + Promises

我们展示了generator如何异步地迭代，用顺序的可推理性来取代混乱的回调。现在我们将generator（看似同步的异步代码）与Promise（可靠性和可组合性）组合起来。

回想一下之前基于Promise的方式运行Ajax的例子：

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

foo( 11, 31 ).then(
	function(text){
		console.log( text );
	},
	function(err){
		console.error( err );
	}
);
```

发挥Promise和generator的最大功效的方法是 **`yield`一个Promise**，并将这个Promise连接到generator的 *迭代器* 的控制端。

首先，我们将Promise相关的`foo(..)`与generator`*main()`放在一起：

```js
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
```

在这个重构中，`*main()`内部的代码 **更本就没变！** 在generator内部，无论什么样的值被`yield`出去都是一个不可见的实现细节，我们甚至不会察觉它发生了，也不用担心它。

那么我们现在如何运行`*main()`？接收并连接`yield`的promise，使它能够根据解析来继续运行generator。

```js
var it = main();

var p = it.next().value;

// 等待`p` promise解析
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```
我们利用了这样一个事实：我们知道`*main()`里面只有一个Promise相关的步骤。

如果想要能用Promise驱动一个generator而不管它有多少步骤呢？
要是有这样一种方法该多好：可以重复（“循环”）迭代的控制，而且每次一有Promise出来，就在继续之前等待它的解析。

另外，如果generator在`it.next()`调用期间抛出一个错误怎么办？我们是该退出，还是应该`catch`它并把它送回去？相似地，要是`it.throw(..)`一个Promise拒绝给generator，但是没有被处理，又直接回来了呢？

#### 带有Promise的Generator运行器

这里我们可以定义一个名为`run(..)`的独立工具：

```js
// 感谢Benjamin Gruenbaum (@benjamingr在GitHub)在此做出的巨大改进！
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	// 在当前的上下文环境中初始化generator
	it = gen.apply( this, args );

	// 为generator的完成返回一个promise
	return Promise.resolve()
		.then( function handleNext(value){
			// 运行至下一个让出的值
			var next = it.next( value );

			return (function handleResult(next){
				// generator已经完成运行了？
				if (next.done) {
					return next.value;
				}
				// 否则继续执行
				else {
					return Promise.resolve( next.value )
						.then(
							// 在成功的情况下继续异步循环，将解析的值送回generator
							handleNext,

							// 如果`value`是一个拒绝的promise，就将错误传播回generator自己的错误处理g
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})(next);
		} );
}
```

如你所见，它可能比你想要自己编写的东西复杂得多，特别是你将不会想为每个你使用的generator重复这段代码。所以，一个帮助工具/库绝对是可行的。

你如何在我们 *正在讨论* 的例子中将`run(..)`和`*main()`一起使用呢？

```js
function *main() {
	// ..
}

run( main );
```

#### Generator中的Promise并发

至此，所有我们展示过的是一种使用Promise+generator的单步异步流程。但是现实世界的代码将总是有许多异步步骤。

如果你不小心，generator看似同步的风格也许会蒙蔽你，使你在如何构造你的异步并发上感到自满，导致性能次优的模式。那么我们想花一点时间来探索一下其他选项。

想象一个场景，你需要从两个不同的数据源取得数据，然后将这些应答组合来发起第三个请求，最后打印出最终的应答。

你的第一直觉可能是像这样的东西：

```js
function *foo() {
	var r1 = yield request( "http://some.url.1" );
	var r2 = yield request( "http://some.url.2" );

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// 使用刚才定义的`run(..)`工具
run( foo );
```

这段代码可以工作，但在我们特定的这个场景中，它不是最优的。

因为`r1`和`r2`请求应该并发运行，但在这段代码中它们将顺序地运行；直到`"http://some.url.1"`请求完成之前，`"http://some.url.2"`URL不会被Ajax取得。这两个请求是独立的，所以性能更好的方式可能是让它们同时运行。

但是使用generator和`yield`，到底应该怎么做？

最自然和有效的答案是基于Promise的异步流程，最简单的方式：

```js
function *foo() {
	// 使两个请求“并行”
	var p1 = request( "http://some.url.1" );
	var p2 = request( "http://some.url.2" );

	// 等待两个promise都被解析
	var r1 = yield p1;
	var r2 = yield p2;

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

run( foo );
```

`p1`和`p2`是并发地（也就是“并行”）发起的Ajax请求promise。然后我们使用两个连续的`yield`语句等待并从promise中取得解析值（分别取到`r1`和`r2`中）。如果`p1`首先解析，`yield p1`会首先继续执行然后等待`yield p2`继续执行。如果`p2`首先解析，它将会耐心地保持解析值知道被请求，但是`yield p1`将会首先停住，直到`p1`解析。

不管是哪一种情况，`p1`和`p2`都将并发地运行，并且在`r3 = yield request..`Ajax请求发起之前，都必须完成，无论以哪种顺序。
这和`Promise.all([ .. ])`的模式是相同的。所以，我们也可以像这样表达这种流程控制：

```javascript
function *foo() {
	// 使两个请求“并行”并等待两个promise都被解析
	var results = yield Promise.all( [
		request( "http://some.url.1" ),
		request( "http://some.url.2" )
	] );

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// 使用前面定义的`run(..)`工具
run( foo );
```

#### Promises，隐藏起来
要小心你在 **你的generator内部** 包含了多少Promise逻辑。以我们描述过的方式在异步性上使用generator的全部意义，是要创建简单，顺序，看似同步的代码，并尽可能多地将异步性细节隐藏在这些代码之外。

比如，这是一种更干净的方式：

```js
// 注意：这是一个普通函数，不是generator
function bar(url1,url2) {
	return Promise.all( [
		request( url1 ),
		request( url2 )
	] );
}

function *foo() {
	// 将基于Promise的并发细节隐藏在`bar(..)`内部
	var results = yield bar(
		"http://some.url.1",
		"http://some.url.2"
	);

	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield request(
		"http://some.url.3/?v=" + r1 + "," + r2
	);

	console.log( r3 );
}

// 使用刚才定义的`run(..)`工具
run( foo );
```

在`*foo()`内部，它更干净更清晰地表达了我们要做的事情,我们不必关心在底层一个`Promise.all([ .. ])`的Promise组合将被用来完成任务。


## ES7: `async` 和 `await`？

前面的模式——generator让出一个Promise，然后这个Promise控制generator的 *迭代器* 向前推进至它完成——如果我们不需要通过乱七八糟的工具库（也就是`run(..)`）来使用它就更好了。

这时候ES7中推出了` async `函数，使得异步操作变得更加方便。

`async `函数是什么？一句话，它就是 `Generator`函数的语法糖。

```javascript
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}


----------


async function main() {
	try {
		var text = await foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

main();
```
这里没有`run(..)`调用来驱动和调用`main()`——它仅仅像一个普通函数那样被调用。另外，`main()`不再作为一个generator函数声明；它是一种新型的函数：`async function`。而最后，与`yield`一个Promise相反，我们`await`它解析。

如果你`await`一个Promise，`async function`会自动地知道做什么——它会暂停这个函数（就像使用generator那样）直到Promise解析。我们没有在这个代码段中展示，但是调用一个像`main()`这样的异步函数将自动地返回一个promise，它会在函数完全完成时被解析。

正常情况下，`await`命令后面是一个` Promise `对象。如果不是，会被转成一个立即`resolve`的 `Promise `对象。
```javascript
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123
```
上面代码中，`await`命令的参数是数值`123`，它被转成 `Promise` 对象，并立即`resolve`。
