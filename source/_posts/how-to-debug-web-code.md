---
title: 前端代码如何调试？
date: 2018-05-06 16:57:44
tags: h5  web debug
---
> 前端代码的调试即javaScript调试，调试方法有很多，不同的开发者有不同的调试方式。这里介绍几种从入门到入坑的调试方法。

## 最简单调试方法：打印日志

打印日志不仅仅是前端代码的调试方法，它几乎是大部分开发语言的入门级调试方法。很多开发者在新手阶段都会采用这一方式，将变量值等信息打印出来。
javaScript输出日志主要方法是console对象。console对象有很多个方法，例如console.assert、console.clear、console.count等方法，其中与日志输出有关的方法有：

```js
console.log() // 打印字符串

console.info() //打印一条信息

console.error() // 打印一条错误信息

console.warn() //打印一个警告信息
```
打印日志的调试方法超级简单，只需要在需要查看信息的代码行调用上面的方法即可输出想要的信息。上述4个方法均可将对象打印出来，例如：
```js
var person = {
  name: 'Rose'
}

console.info('person', person)
```
如果使用的是Chrome浏览器，右键页面=>检查，在console中即可看到相应的输出:
<div style="text-align: center;">
  <img src="console.info.png">
</div>

## 代码中插入debugger
javascript提供了一个关键字：`debugger`，这个语句厉害了，可以调用任何可用的调试功能，设置断点。但是，如果没有调试功能可用，此语句将不起作用。
下面是一个包debugger语句的html代码，当代码被执行时，会尝试调用一个可用的调试器进行调试。
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>test</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
<script>
  var a = 1
  a = a + 1
  debugger
  a = a + 2
</script>
</body>
</html>
```
保存该文件，使用Chrome浏览器将其打开。打开Chrome developer tool， 即右键页面=>检查，发现没啥呀，只有下面这个页面：
<div style="text-align: center">
  <img src="debugger.png">
</div>
此时我们刷新页面，dang～dang～，变这样了：
<div style="text-align: center">
  <img src="debuggering.png">
</div>
看到图中标红的地方了吗？那就是我们设置的断点，程序运行到那里停止了。此时我们将鼠标移至变量`a`上，即可看到`a`的值。
下图中红圈内数字也可以设置断点。
<div style="text-align: center">
  <img src="debuggering-left.png">
</div>

## 编辑器中调试
