---
title: J2V8实现Java与Javascript互相调用
date: 2019-02-18 10:25:33
tags: J2V8, Java, Javascript
---

## Java调用JS方法
编写一个返回String的JS方法
```js
// js
function javaCallJSFunc (msg) {
  return msg + ' -> Hi, Java!'
}
```

引入J2V8库时可以引入jar或者Android的aar包，或者自行编译。例如Android Studio引入：
`implementation 'com.eclipsesource.j2v8:j2v8:4.8.2@aar'`
下述例子均是在Android中运行。

```java
// java
void javaCallJSFunc () {
  V8 jsRuntime = V8.createV8Runtime();
  jsRuntime.executeScript("function javaCallJSFunc (msg) {\n" +
          "    return msg + ' -> Hi, Java!'\n" +
          "}");
  // 添加参数
  V8Array params = new V8Array(jsRuntime).push("Hi, Javascript!");
  String result = jsRuntime.executeStringFunction("javaCallJSFunc", params);
  Log.e("Test", "javaCallJSFunc result = " + result);
  // 需手动释放资源
  jsRuntime.release();
}
```

## JS调用Java方法
js调用Java方法，从实现方式上有两种：JavaCallback、Relfectively

### JavaCallback
假设我们在Javascript中调用Java的setData方法
```js
//js
var date = new Date().toDateString()
setData(date)
```

在Java中实例化一个JavaVoidCallback的匿名类
```java
// java
// jsCallJavaFunc js调用Java方法
void jsCallJavaFunc () {
  V8 jsRuntime = V8.createV8Runtime();
  JavaVoidCallback setData = new JavaVoidCallback() {
    @Override
    public void invoke(V8Object v8Object, V8Array v8Array) {
      if (v8Array.length() > 0) {
        // 获取js传入的参数
        Object arg1 = v8Array.get(0);
        Log.i("J2V8", "jsCallJavaFunc -> " + arg1.toString());
        if (arg1 instanceof Releasable) {
          ((Releasable) arg1).release();
        }
      }
    }
  };
  jsRuntime.registerJavaMethod(setData, "setData");
  jsRuntime.executeScript("var date = new Date().toDateString()\n" +
      "  setData(date)");
  // 需手动释放资源
  jsRuntime.release();
}
```

### Relfectively
更确切的说是类似于在Javascript中，给对象定义方法，也就是js调用Java Object的方法。
创建一个Java类，该类包含Javascript可以调用的方法。
```java
// java
...
public class JSBridge {
  public void setData (String data) {
    Log.i("JSBridge", data);
  }
}
```

在js中调用`JSBridge.setData`方法
```js
var date = new Date().toDateString()
JSBridge.setData(date)
```

在Java中将JSBridge暴露给js
```java
// java
void jsCallJavaObject () {
  JSBridge jsBridge = new JSBridge();
  V8Object jsBridgeObj = new V8Object(jsRuntime);
  // 将jsBridge暴露给js
  jsRuntime.add("JSBridge", jsBridgeObj);
  // 注册方法
  jsBridgeObj.registerJavaMethod(jsBridge, "setData", "setData", new Class<?>[] {String.class});
  jsBridgeObj.release();
  jsRuntime.executeScript("var date = new Date().toDateString()\n" +
    "JSBridge.setData(date)");
}
```