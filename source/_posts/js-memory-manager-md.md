---
title: JavaScript的内存管理，以及如何处理4种常见的内存泄漏
date: 2018-02-27 11:12:22
posted: 2018-02-27 11:12:22
tags: JavaScript
author: 少盐
---

# JavaScript的内存管理，以及如何处理4种常见的内存泄漏

起初从一个公众号中看到了这篇文章，拜读的时候发现有一两处地方有点难懂。于是尝试翻译一下原文，原文地址为[How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

作者针对JavaScript写了一系列的文章，第三篇便是此文章。第一篇和第二篇分别是[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)、[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

文章开头讲叙了一些内存方面的基础知识，这里略过直接开车。
#### JavaScript的内存分配
JavaScript对内存的分配进行自动管理，而不同于C语言需要开发者管理。
```js
var n = 374; // 为数字分配内存
var s = 'sessionstack'; // 为字符分配内存
var o = {
  a: 1,
  b: null
}; // 为object和其属性分配内存
var a = [1, null, 'str'];  // 分配数组空间
function f(a) {
  return a + 3;
} // 为方法分配内存（方法本质上是一个可以回调的object）

// 函数表达式也作为object来分配内存
someElement.addEventListener('click', function() {
  someElement.style.backgroundColor = 'blue';
}, false);
```

一些函数的返回结果也当作object分配内存：
```js
var d = new Date(); //分配一个Date object

var e = document.createElement('div'); // 分配一个DOM 元素
```
一些方法也能分配新的值或object内存空间：
```js
var s1 = 'sessionstack';
var s2 = s1.substr(0, 3); // s2 是一个新的字符变量
// 由于已分配内存空间的字符,在内存空间中值是不变的
// JavaScript可能不会给2分配新的内存空间,而是仅仅记录[0, 3]的区间.
var a1 = ['str1', 'str2'];
var a2 = ['str3', 'str4'];
var a3 = a1.concat(a2);
// 数组拼接将会分配新的内存空间
```
#### 当内存不再需要时，内存空间会被释放
大多数的内存管理问题都出现在这个阶段。在这个阶段，比较难的是如何判断内存空间可以释放掉了。这通常需要开发者确定哪些内存不再需要，然后将其释放。
高等级的编程语言都会集成垃圾回收器。垃圾回收器对内存的分配和使用进行跟踪。当分配的内存不再需要时，垃圾回收器将会释放内存。

然而，总是有一些内存片段，垃圾回收器的算法难以判断其是否需要。大多数的垃圾回收器通过收集不再访问的内存，例如所有指向该内存的变量都已经不存在了。但是，可能存在一个变量一直指向某一内存空间，而程序不再使用该变量。

### 垃圾回收
由于难以决定内存是否继续需要，垃圾回收器难以提供相应的解决方案。本节我们将会对垃圾回收器的回收算法一探究竟，并发现其存在的缺陷。

#### 内存引用
垃圾回收算法主要依赖于引用。

在内存管理这中，引用是一个对象可以访问(显式或隐式)另一个对象。例如，JavaScript对象继承了Object.prototype的属性和方法。其继承Object。prototype便是隐式引用，属性和方法便是显式继承。

#### 垃圾回收算法：引用计数（reference-counting）
rc垃圾回收是最简单的垃圾回收算法。如果指向对象的引用数量为0则这个对象是可以回收的。

看一下以下代码：
```js
var o1 = {
  o2: {
    x: 1
  }
}
// 创建了2个对象。其中，‘o2’被'o1'对象引用，作为'o1'的属性
// 'o1'和'o2'都不能被回收

var o3 = o1; //o3引用o1
o1 = 1; // o1重新赋值，o1原来的内存现只被o3引用

var o4 = o3.o2 // o4引用o2属性。现在o2属性在两处被引用

o3 = '374' // 原o1的内存空间，被引用次数为0。但是其属性o2仍然被o4引用，不能被回收

o4 = null // o2不再被o4引用，原o1引用数为0，现在可以被回收了
```
#### 内存泄漏：循环引用
下面的例子中，两个对象相互引用。方法调用完后，两个对象将不再需要，可以被释放。然而，rc垃圾回收算法认为每个对象都被引用了，所以不会回收。
```js
function f() {
  var o1 = {}
  var o2 = {}
  o1.p = o2 // o1引用o2
  o2.p = o1 // o2引用o1, 这就产生了循环引用
}
```
<div style="text-align: center;">
  <img src="cycles-are-creating-problems.png">
</div>

#### 垃圾回收算法：标记&扫描（mark&sweep）
为了判断一个对象是否可以释放，m&s算法是通过判断对象是否可达。

m&s算法流程如下：

1. 垃圾回收器首先建立了一个roots列表。roots通常是一些存在引用的全局变量，可以理解为引用的入口。在JavaScript中，“window”对象就是一个可以作为入口的全局对象
2. 所有的root都被检查，并被标记为活动状态，也就是非垃圾。同时，所有root的所有孩子节点都也被递归检查。所有能从root可达的对象都不是垃圾。
3. 剩余没有油被标记为活动状态的对象，都可以作为垃圾回收。垃圾回收器可以释放相应的内存，归还给系统。
<div style="text-align: center;">
  <img src="mark-and-sweep-algorithm.gif">
</div>

m&s算法要优于前面提到的引用计数算法，因为引用计数算法会导致一些对象从root不可达，而其又被引用了。m&s算法可以释放部分存在循环引用的内存空间。

截止2012年，所有的浏览器都采取了m&s垃圾回收机制。[这里](https://en.wikipedia.org/wiki/Tracing_garbage_collection) 详细介绍了垃圾机制，其中也包括m&s垃圾回收机制及其优化。

#### 循环引用不再是问题
在上面的第一例子中，方法调用完后，方法中的两个对象不能从root可达，将被垃圾回收标记为不可达，进而被垃圾回收器回收。
<div style="text-align: center;">
  <img src="cycles-are-not-a-problem-anymore.png">
</div>

#### 垃圾回收机制的缺陷
虽然垃圾回收器能够自动回收不再使用的内存空，但是，垃圾回收器执行回收的时机是不可预测的，也就是我们不知道垃圾回收什么时候会执行m&s流程。有些情况下，即使里创建了一个数组，只要浏览器的可用内存足够，垃圾回收器可能就不会执行回收操作。

#### 什么是内存泄漏
内存泄漏可以定义为：应用程序不再需要的内存空间，由于某种原因，内存空间没有被系统回收，没有回到系统的可用内存池中。

编程语言通常有多种方法管理内存。但是，编程语言很难判断某一内存空间是否被使用。换句话说，只有开发者能明确的告诉浏览器某一内存空间是否能被系统回收。编程语言为开发提供了一些执行相应操作的功能，其他则需要开发者自身明确什么时候使用了内存。Wikipedia上有一些关于[自动](https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29)和[人工](https://en.wikipedia.org/wiki/Manual_memory_management)管理内存的文章。

## 四种常见的内存泄漏

#### 1. 全局变量
JavaScript处理没有申明的变量方式是将其作为浏览器的window的属性：
```js
function foo(arg) {
  bar = 'some text'
}
```
等同于:
```js
function foo(arg) {
  window.bar = 'some text'
}
```
浏览器测试效果：
<div style="text-align: center;">
  <img src="global-var.png">
</div>

所以，如果不小心忘记使用var或let等等申明bar变量，将会在方法内创建全局变量。使用ESlint进行代码规范是个不错的选择。

另一不小心创建全局变量的途径是通过this ：

```js
function foo() {
  this.var1 = 'potential accidental global'
}
// 调用foo方法时，‘this’指向全局对象window，而不是undefined
foo()
```

为避免这样的错误发生，建议在js文件第一行加入`use strict`。这能阻止意外产生全局变量。有关strict模式的更多内容，可以查看这里 [严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)。

在申明全局变量时建议赋值为null，在程序需要的地方再分配内存。

#### 2. 遗漏计时器或者回调函数
JavaScript的setInterval和setTimeout方法，如果不主动回收都可能导致内存泄漏。

关于回调函数导致的内存泄漏，原文提到了addEventListener和removeEventListener函数。然而，现代的浏览器都会在DOM被销毁时，其上绑定的事件也都会被回收，并不是严格要求addEventListener后，必须removeEventListener后才能回收内存。

#### 3. 闭包
JavaScript开发的一个重要特性就是闭包：内部的方法能够访问到外部（闭环）方法的变量。由于JavaScript运行时的具体实现，下面的代码可能导致内存泄漏：

```js
var theThing = null

var replaceThing = function () {
  var originalThing = theThing
  var unused = function () {
    if (originalThing) // 引用originalThing
      console.log('hi')
  }

  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('message')
    }
  }
}

setInterval(replaceThine, 1000)
```

该代码执行内容是：每次replaceThing方法被调用，theThing都获取到一个包含大数组属性和闭包（函数）。同时，变量unused获得一个引用originalThing的闭包。是不是有点懵了？其实，只要记住一点：对子闭包而言，同一父闭包的内存空间是共享的。

在上面的例子中，`unused`是可以访问到`someMethod`的。在方法`replaceThing`方法外面，`theThing`可以调用`someMethod`。由于`someMethod`与`unused`共享闭包区域，`unused`对`originalThing`的引用，导致`someMethod`一直保持活动状态，阻止了其被回收，即使`unused`方法没有被回调。

仔细一想，原文的意思应该如下图所示：
<div style="text-align: center;">
  <img src="closures.png">
</div>

随着定时器的执行，不断的有新的内存空间被分配，同时老的内存空间依然被引用，得不到释放。这个问题是[Meteor](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156)团队发现的

#### 4. DOM外引用
有时候我们可能需要存储一些DOM的引用，以便多次访问。但是，当引用的DOM元素被从页面移除时，我们存储的DOM引用可能会导致内存泄漏。例如：
```js
var elements = {
  button: document.getElementById('button'),
  image: document.getElementById('image')
}

function doStuff() {
  elements.image.src = 'http://example.com/image_name.png'
}

function removeImage() {
  document.body.removeChild(document.getElementById('image'))
  // 这时，我们依然在全局变量elements中引用了image，也就是，image元素仍然保存在内存中，不会被垃圾回收器回收
}
```
所以，我们在创建引用的时候需要注意对DOM的引用。