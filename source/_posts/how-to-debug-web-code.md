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
c++、Java、c#等语言都能在编辑器中进行调试、打断点，主要原因是它们都可以在编辑器中运行，而Web前端代码只能运行在浏览器中。

#### webstorm
webstorm可以在编辑器中进行调试、打断点，十分方便。但是，webstorm运行时占用资源比较多，对于性能比较低的电脑，webstorm运行的不是很流畅。
webstorm的调试方法可以在官网中找到。

#### Sublime Text
Sublime Text是一个比较轻量的脚本编辑器，运行比较流畅，很多前端开发者都喜欢使用。
Sublime Text有几个可以实现调试的Package，但是试用了一下，没有成功

### Visual Studio Code
Visual Studio Code具有Sublime Text轻量，同时也能实现调试，简直要抛弃Sublime Text投靠Visual Studio Code的怀抱了❤️

#### 1、安装Debugger for Chrome插件
安装最新版的Visual Studio Code后，打开它，点击左侧扩展图标
<div style="text-align: center">
  <img src="vscode-install-chrome-start.png">
</div>

在“在商店中搜索扩展”中输入“Debugger for Chrome”，安装Debugger for Chrome即可
<div style="text-align: center">
  <img src="vscode-install-chrome-end.png">
</div>

安装成功后，先新建文件夹test/src，在其中新建index.html和index.js两个文件
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Page Title</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <div>o</div>
  <script src="./index.js"></script>
  <script>
   var result = add(1, 2)
   console.info('html', result)
  </script>
</body>
</html>
```
index.js:

```js
function add(a, b) {
  return a + b
}

var result = add(1, 2)
console.info(result)
```
首先在index.js中打一个断点。点击图中红框区域即可添加断点，也可将鼠标放在红框区域右键添加、删除断点
<div style="text-align: center">
  <img src="add-break-point.png">
</div>

选中test文件夹，按快捷键F5，或者点击调试->启动调试，选择Chrome。Visual Studio Code会自动在test文件夹下创建`.vscode/launch.json`文件。修改`launch.json`文件，修改或增加途中红线标识的行。
<div style="text-align: center">
  <img src="launch.json.png">
</div>

然后再次按F5或启动调试，此时Chrome会自动打开index.html，刷新页面，即可发现Visual Studio Code在刚才我们添加的断点出阻塞了
<div style="text-align: center">
  <img src="vscode-debugger.png">
</div>

## 移动端网页调试
移动端主要分Android和iOS，Windows Phone就暂时不考虑了，2017年销售的智能手机，99.9%的搭载了这两个操作系统。

### Android端调试
Android手机调试首先需要开启开发者模式，其次，如果是调试浏览器则需要安装Chrome浏览器，若是调试其他APP则需要APP开发者打开调试的开关。

完成上述步骤后，打开网页，使用usb线链接电脑，允许电脑调试手机。在PC端Chrome浏览器地址栏输入`chrome://inspect`
<div style="text-align: center">
  <img src="chrome-inspect.png">
</div>

### iOS端调试网页
macOS操作系统调试iOS网页比较简单，只需iPhone设置->Safari->高级->打开Web检查器开关，打开网页，iPhone通过数据线连接Mac电脑，打开Mac端Safari，点击开发->[你的手机]->选择调试的网页
<div style="text-align: center">
  <img src="safari-debug.png">
</div>

也可以使用iPhone模拟器调试，前提需要安装Xcode。上图正是模拟器调试。

Windows操作系统要调试iOS端网页，则需要通过Chrome调试，并安装一些工具便可以调试Safari了

## 最强大的调试方法
使用浏览器自带的调试工具，可以说是最常用，也是最强大的调试方法。不仅可以调试自己开发的网页，也可以hack其他网页，从别人那里学习一些前端知识。

Chrome浏览器的调试功能比较强大,其拥有很多功能，不仅可以查看、整理代码，还可以检查网页的性能、动画等等
<div style="text-align: center">
  <img src="chrome-debug-taobao.png">
</div>

Firefox浏览器调试功能也同样强大，完全媲美Chrome。最新版的Firefox在性能上有了很大的提升，渲染网页的速度赶超Chrome，资源占用却没有Chrome高。
<div style="text-align: center">
  <img src="chrome-debug-taobao.png">
</div>

国产浏览器中，如搜狗浏览器、UC浏览器等都支持调试，部分内核使用了Chromium，也就是Chrome内核，调试功能和Chrome差不多，可能有版本延迟，缺少部分新功能。