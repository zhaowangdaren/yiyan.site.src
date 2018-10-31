---
title: 'ES6:看不懂的let类型与闭包间的关系+Babel的bug'
date: 2018-10-10 14:13:27
tags:
---

> `let`是ES6变量类型，其只在代码块内有效。
> 闭包是有函数以及创建该函数的词法环境组合而成，这个环境包含了这个闭包创建时所能访问的所有局部变量。

如果合理使用闭包，可以使用`var`实现`let`局部有效的效果。显然这是多余的，因为`babel`已经能帮我们做了，并且很多新版浏览器都支持`let`，所有本文结束。

等等，`babel`都干了啥？为啥能将`let`转码使代码兼容旧版浏览器？

## 兼容转码
我们先来看一段比较简单的`babel`转码
<div style="text-align:center;">
  <img src="sample-let.png">
</div>

我们来搬一段特别复杂的`let`类型申明代码

```js code1
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

在`babel`官网将其转码为兼容代码：

```js code2
"use strict";

var a = [];

var _loop = function _loop(i) {
  a[i] = function () {
    console.log(i);
  };
};

for (var i = 0; i < 10; i++) {
  _loop(i);
}
a[6](); // 6
```

老司机马上就看出了：将`let`转为了闭包

## 闭包

我们来看看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)上的一段例子

```js code3
function makeFunc() {
    var name = "Mozilla";
    function displayName() {
        alert(name);
    }
    return displayName;
}

var myFunc = makeFunc();
myFunc();
```

闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量。
我们一通分析便可以看出`code3`闭包的基本结构：

```js code4
function (j) {
  return function () {
    console.log(j)
  }
}(i)
```

`code4`的闭包是立即执行

## 利用闭包，使var == let

`code4`的结构不正是`code2`的简写吗？我们可以将`code2`改写为：
```js code5
"use strict";

var a = [];

for (var i = 0; i < 10; i++) {
  a[i] = function (j) {
    return function () {
      console.log(j)
    }
  }(i)
}
a[6](); // 6
```

说好的简写呢？！
仔细阅读这句话<b>闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量。</b>

`let`是块级作用域，也就是局部作用域。闭包也是能访问局部作用域。

`code4`正是将变量`i`赋值给函数的局部变量`j`，`code5`中数组`a`的每个item都是该函数实例的引用。
在`let`出现之前，在循环中写闭包是一个比较常见的问题。

我们不鼓励在循环中创建闭包，过多的闭包会付出性能代价。

## Babel的bug

意外发现了一个Babel的小bug。我们在最新的Chrome/Firefox浏览器中运行这段代码：

```js code6
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i, new Date().getTime());
}
```
会发现输出结果为<b>3</b>次"abc 1539160907***"。将`code6`通过`Babel`转码得到`code7`

```js code7
for (var i = 0; i < 3; i++) {
  var i = 'abc';
  console.log(i, new Date().getTime());
}
```

`code7`在浏览器中仅仅运行了<b>1</b>次，输出结果为"abc 1539160907***"

[阮老师](http://es6.ruanyifeng.com/#docs/let)讲：
> `for`循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。