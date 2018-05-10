---
title: ES2017中async&await踩坑指南
date: 2018-05-10 16:15:26
tags:
---

ES2017标准引入了async、await函数，使得异步操作变得更加方便了，而不用每次`new Promise(...)`了

> 本文还使用到了es6的其他特性，例如箭头函数`=>`，以及不用说的`Promise`，所以为了在生产上兼容，需要引入babel-polyfill，并对代码进行Babel处理。这里暂时不对相关知识进行细说。
> 如果你使用最新版的Chrome浏览器进行<a href="https://www.toutiao.com/i6553080029855613443/">开发调试</a>，就可以暂时不用关心相关知识了。

### 基本用法

#### async
`async`其实是`Promise`语法糖。我们首先来写一个`Promise`：
```js
function asyncWork () {
  return new Promise((resolve, reject) => {
    // TODO something
    var isSucc = true
    if (isSucc) {
      reslove()
    } else {
      reject()
    }
  })
}

function doWrok () {
  asyncWork().then(resp => {
    console.log('success')
  })
}
```
将其改为`async`写法：
```js
async function asyncWork () {
  // TODO something
  var isSucc = true
  if (isSucc) {
    return
  } else {
    throw 'error'
  }
}

function doWork () {
  asyncWork().then(resp => {
    console.log('success')
  })
}
```
可以看到使用`async`语法糖后，`asyncWork`代码量减少了，特别是在代码逻辑比较复杂的时候，使用`async`会直观很多。

#### await
往往使用`await`的函数需要使用`async`。

`async`表示异步，而`await`表示需要等待。进一步说，`async`函数可以看成将异步操作包装成一个Promise对象，而`await`就是`then`命令的语法糖。
await使用方法：
```js
async function asyncWork () {
  // TODO something
  var isSucc = true
  if (isSucc) {
    return
  } else {
    throw 'error'
  }
}

async function doWork () {
  // 将等待两个asyncWork的返回结果，然后执行下一步
  var workResult1 = await asyncWork()
  var workResult2 = await asyncWork()
  return workResult1 + workResult2
}

function mainWork() {
  doWrok.then(resp => {
    console.log(resp)
  })
}
```
当然，也可以将上面的`asyncWork`方法改为使用`Promise`。

### 进阶踩坑
既然使用了javaScript的ES2017特性，那么你一定会使用箭头函数`=>`。相应的，在实际项目中，为了兼容性，我们会使用`babel-preset-env`等进行转义。
坑来了，我们将上面的`doWork`方法改为如下：
```js
async function doWork () {
  var array = [1, 2, 3]
  array.forEach(item => {
    // 将等待两个asyncWork的返回结果，然后执行下一步
    var workResult1 = await asyncWork()
    var workResult2 = await asyncWork()
    console.log(workResult1 + workResult2)
  })
  console.log(array)
}
```
我们来创建两个文件`index.html`、`main.js`来进行验证:
<div style="text-align: center;">
  <img src="files.png">
</div>
`index.html`:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>async&await</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="./main.js"></script>
</head>
<body>
  
</body>
</html>
```
`main.js`
```js
async function asyncWork () {
  // TODO something
  var isSucc = true
  if (isSucc) {
    return
  } else {
    throw 'error'
  }
}

async function doWork () {
  var array = [1, 2, 3]
  array.forEach(item => {
    // 将等待两个asyncWork的返回结果，然后执行下一步
    var workResult1 = await asyncWork()
    var workResult2 = await asyncWork()
    console.log(workResult1 + workResult2)
  })
  console.log(array)
}

function mainWork() {
  doWrok.then(resp => {
    console.log(resp)
  })
}
```
这里，我们使用<a href="https://parceljs.org/">Pracel</a>来进行打包。Parcel号称极速零配置Web应用打包工具，但是还是需要一点点配置的（当然也有一些限制），不过比那个vue-cli的webpack模板配置要简单多了!!!

使用`npm init`或者`yarn init`对项目初始化后，命令行运行：`parcel index.html`，运行结果：
<div style="text-align: center;">
  <img src="await-error.png">
</div>
OMG!!!竟然报错了，什么情况？`async`、`await`这对鸳鸯成对出现了，怎么还会报错？！
且慢，我们不妨看看`Parcel`对`doWork`的处理结果是什么。我们打开项目下的dist文件夹。dist文件夹下存放了`Parcel`的打包结果。
<div style="text-align: center;">
  <img src="dist-folder.png">
</div>
...
哈哈，竟然只有`index.html`文件。那我们上述`doWork`函数改一下：
```js
async function doWork () {
  var array = [1, 2, 3]
  for (var i = 0; i < array.length; i++) {
    // 将等待两个asyncWork的返回结果，然后执行下一步
    var workResult1 = await asyncWork()
    var workResult2 = await asyncWork()
    console.log(workResult1 + workResult2)
  }
  console.log(array)
}
```
再次运行`parcel index.html`，编译成功：
<div style="text-align: center;">
  <img src="for-succ.png">
</div>
机智的我们发现是箭头`=>`函数在添乱。我们再看一下es6中`=>`的定义：
```js
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```
那上面包含箭头函数的`doWork`也就等同于：
```js
async function doWork () {
  var array = [1, 2, 3]
  array.forEach(function(item) {
    // 将等待两个asyncWork的返回结果，然后执行下一步
    var workResult1 = await asyncWork()
    var workResult2 = await asyncWork()
    console.log(workResult1 + workResult2)
  })
  console.log(array)
}
```
外层的`doWork`函数和包裹await的函数已经不是同一个函数了，所以`Parcel`编译失败。如果想继续使用箭头函数，可以改为：
```js
function doWork () {
  var array = [1, 2, 3]
  array.forEach(async(item) => {
    // 将等待两个asyncWork的返回结果，然后执行下一步
    var workResult1 = await asyncWork()
    var workResult2 = await asyncWork()
    console.log(workResult1 + workResult2)
  })
  console.log(array)
}
```
所以，`async`、`await`、`=>`等es6特性，在使用时要注意它们的作用，避免增加不必要的调试时间。