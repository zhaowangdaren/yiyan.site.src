---
title: 从零开始，再造一个小程序平台
date: 2019-02-23 19:14:41
tags:
---

微信发布小程序平台后，国内各大APP火速跟进，纷纷发布了自己的小程序平台。

速度真的很快，难道被微信形容“拥有出色体验”的小程序平台很容易实现？我们可以再造一个吗？

> 起初，作者甚至不知道小程序是由Native实现，还是网页实现。既然要再造一个，那么首先要做的就是来了解小程序平台实现原理。

## 调研~~搜索~~

没有参与过小程序平台的建设，所以只能发动搜索技能。所以这篇文章也更多的是管中窥豹，抛砖引玉。

搜索多篇与小程序实现原理有关的文章，认真拜读，发现在微信小程序平台发布之初，微信团队将小程序构建相关的脚本直接下发给了微信开发者工具，也就是小程序构建是在开发者端。现在新版的微信开发者工具中已经没有相关脚本了，推测是将构建转移到了服务器端。

从相关的文章中，我们了解~~猜想~~到了微信小程序实现方式：UI层由网页+原生渲染（部分组件是原生实现），Native运行逻辑层。
<div style="text-align: center;">
  <img src="LV.png"/>
</div>

## 技术方案

UI层渲染采用webview，js逻辑层呢？

React Native的js引擎主要采用了`JavaScriptCore`(苹果Safari浏览器js引擎)，Weex则是采用了`Chrome V8`引擎。

## 实现

道理你都懂，那怎么实现呢？

### UI和js逻辑

在实现之前，我们首先来看看微信小程序的代码，分析一下。以一个小程序的index页面为例，其代码包含文件：index.js、index.json、index.wxml、index.wxss

*** index.js ***
index.js主要是业务逻辑代码，内容大致为：
```js
//index.js
//获取应用实例
const app = getApp()

Page({
  data: {
    motto: 'Hello World',
    userInfo: {},
    hasUserInfo: false,
    canIUse: wx.canIUse('button.open-type.getUserInfo')
  },
  //事件处理函数
  bindViewTap: function() {
    wx.navigateTo({
      url: '../logs/logs'
    })
  },
  onLoad: function () {
    if (app.globalData.userInfo) {
      this.setData({
        userInfo: app.globalData.userInfo,
        hasUserInfo: true
      })
    }
  }
})
```
小程序的页面逻辑代码均包含在`Page({...})`中。在js中，Page是一个函数。所以，小程序应该是调用了Native或者js的Page函数。

小编倾向于Page是调用了js函数。因为如果是Native函数，那么js中的`this`关键字将需要Native进行处理，js引擎可以处理的内容，没有必要交给Native来进行处理，或者是小编不知道Native有比较好的处理方式。

所以，我们需要处理的第一个内容是：使用js定义Page函数或者class，两者均可以实现。

*** index.wxml ***
index.wxml是UI，是微信自定义xml标签，例如：
```xml
<!--index.wxml-->
<view class="container">
  <view class="userinfo">
    <button wx:if="{{!hasUserInfo && canIUse}}" open-type="getUserInfo" bindgetuserinfo="getUserInfo"> 获取头像昵称 </button>
    <block wx:else>
      <image bindtap="bindViewTap" class="userinfo-avatar" src="{{userInfo.avatarUrl}}" mode="cover"></image>
      <text class="userinfo-nickname">{{userInfo.nickName}}</text>
    </block>
  </view>
  <view class="usermotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
```
自定义一整套的xml标签、属性会比较耗费时间，我们可以直接使用html标签。

*** index.wxss ***
index.wxss主要是样式，和css基本一样，其内容我们完全可以直接引入网页中。
微信小程序则是统一了rpx单位，想必是为了更好的兼容不同屏幕大小。
```css
/**index.wxss**/
.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.userinfo-avatar {
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.userinfo-nickname {
  color: #aaa;
}

.usermotto {
  margin-top: 200px;
}
```

*** index.json ***
这个文件则是该页面的一些基本配置。我们主要关心一下usingComponents
```json
{
  "usingComponents": {}
}
```
这个应该就是小程序可以实现组件化开发的原因。usingComponents直接告诉了Native页面使用了哪些组件，方便Native获取组件模板和逻辑，然后进行渲染。

#### 定义Page
我们来使用js定义一个`Page` class，来作为每个页面的实例。当然也可以使用函数实现。
```js
class Page {
  constructor(config) {
    this.data = config.data
    JSBridge.setData(config.data)
    // 小程序的onLoad生命周期函数
    this.onLoad = config.onLoad
    // 这里与小程序有点不一样
    // 模仿vue将非生命周期的函数都赋给methods属性
    if (config.methods) {
      // 将所有method都指给class
      for (let key in config.methods) {
        this.key = config.methods[key]
      }
    }
  }
}
```
另外我们需要注意的是小程序的`setData`函数。`setData`函数应该是调用Native，通知Native数据变更，重新渲染UI，所以我们还要在该class实现`setData`函数

```js
// page.js
class Page {
  constructor(config) {
    this.data = config.data
    JSBridge.setData(config.data)
    // 小程序的onLoad生命周期函数
    this.onLoad = config.onLoad
    // 这里与小程序有点不一样
    // 模仿vue将非生命周期的函数都赋给methods属性
    if (config.methods) {
      // 将所有method都指给class
      for (let key in config.methods) {
        this.key = config.methods[key]
      }
    }
  }

  setData (data) {
    // JSBridge提供Native与js互相调用的api
    JSBridge.setData(data)
  }
}
```

#### 定义模板
我们应该定义一个html模板，用来在webview中渲染UI。
```html
<!-- templage.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>hybrid</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <!-- app dom下渲染UI -->
  <div id="app">{{template}}</div>
</body>
</html>

```

这里涉及到一个比较重要的问题：UI的更新。
考虑到性能，实现差量更新比较好。小编因此也再看了一遍Vue的虚拟DOM实现，其中涉及到差量更新。
但是为了快速实现原型，我们直接进行全局更新。如何进行更新，这里采用native调用webview中js方法来实现。
```js
// templage.js
// 可以inline
var preTree = null
/**
* 渲染UI
* @param {Object} domTree 树结构
*/
window.render = function (domTree) {
  if (!domTree) return
  // init UI
  document.querySelector('#app').innerHTML = domTree
}
```

```java
// java
this.webView.loadUrl("javascript:render('" + newTemplate + "')");
```

至此，我们就可以作手来实现业务代码了。
#### 业务代码
实现模板index.xml和index.js逻辑，css暂不实现。
*** index.xml ***
```html
<div>
  <div onclick="onText">{{text}}</div>
</div>
```

*** index.js ***
```js
var page = new Page({
  data: {
    text: 'Hi, Java'
  },
  onLoad: function () {
    this.setData({
      text: 'I am Javascript'
    })
  },
  methods: {
    onText: function () {
      this.setData({
        text: 'Hi, Javascript'
      })
    }
  }
})
```

### Native端

通过实现上文的**UI和js逻辑**我们需要Native端实现`JSBridge` api。
我们首先要解决的是js引擎的使用。本文以Android为例，选取`Chrome V8`作为js引擎。

#### J2V8

`J2V8`使用Java封装了对`Chrome V8`的操作，跨平台开发Tabris正是使用了`J2V8`来实现js构建原生应用。

我们可以通过官网[https://eclipsesource.com/blogs/tutorials/getting-started-with-j2v8/](https://eclipsesource.com/blogs/tutorials/getting-started-with-j2v8/)了解如何使用`J2V8`。

##### Android引入

使用Android Studio新建APP后，在build.gradle文件中添加J2V8依赖，选取aar版（已编译）或者使用J2V8源码自行编译：
```java
dependencies {
    implementation 'com.eclipsesource.j2v8:j2v8:4.8.2@aar'
}
```

#### 实现JSBridge
定义一个Java类JSBridge，该类至少包含`setData`接口
```java
package test.site.yiyan.j2v8;

import android.util.Log;
import android.webkit.WebView;
import com.eclipsesource.v8.V8Object;

public class JSBridge {
  String template;
  WebView webView;
  V8Object page;

  public void setPage (V8Object page) {
    this.page = page;
  }

  public void setWebView (WebView webView) {
    this.webView = webView;
  }

  public void setTemplate (String xml) {
    this.template = xml;
  }

  public String getTemplate () {
    return this.template;
  }

  public void setData (V8Object data) {
    // data业务逻辑状态
    String[] keys = data.getKeys();
    String newTemplate = this.template;
    // 遍历data中的状态
    for (int i = 0; i < keys.length; i++) {
      Log.i("setData", keys[i] + ':' + data.get(keys[i]));
      // 将data状态写入UI
      newTemplate = newTemplate.replace("{{" + keys[i] + "}}", data.get(keys[i]).toString());
    }
    data.release();
    if (keys.length > 0) {
      // Native通过webview调用js，更新UI
      this.webView.loadUrl("javascript:render('" + newTemplate + "')");
    }
  }
}
```

#### Native初始化渲染
Android中我们首先获取`template.html`内容，话不多说，看下面的Android代码注释。
```java
V8 jsRuntime;
WebView mWebview;
ProgressBar mProgressBar;
V8Object mJSPageClass;
JSBridge jsBridge;
V8Object page;

void loadHTMLTemplate () {
  String htmlTemplate = Utils.getAssetsScript(this, "js/template.html");
  // 加载JSBridge
  initJSBridge();

  mWebview.setWebChromeClient(new WebChromeClient() {
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
      mProgressBar.setProgress(newProgress);
      if (newProgress == 100) {
        mProgressBar.setVisibility(View.GONE);
        // webview加载完成后
        loadBussiness();
      }
      super.onProgressChanged(view, newProgress);
    }
  });
  mWebview.loadDataWithBaseURL(null, htmlTemplate, "text/html", "utf-8", null);
}

void initJSBridge () {
  jsBridge = new JSBridge();
  V8Object jsBridgeObj = new V8Object(jsRuntime);
  jsRuntime.add("JSBridge", jsBridgeObj);
  jsBridgeObj.registerJavaMethod(jsBridge, "setData", "setData", new Class<?>[] {V8Object.class});
  jsBridgeObj.release();
}

void loadBussiness () {
  // Chrome V8 js引擎加载page.js
  String pageJS = Utils.getAssetsScript(this, "js/page.js");
  mJSPageClass = jsRuntime.executeObjectScript(pageJS);

  // 获取业务模板
  String indexXML = Utils.getAssetsScript(this, "demo/index/index.xml");
  jsBridge.setWebView(this.mWebview);
  jsBridge.setTemplate(indexXML);
  Log.i("J2V8", "indexXML:" + indexXML);

  // 加载业务js
  String indexJS = Utils.getAssetsScript(this, "demo/index/index.js");
  jsRuntime.executeScript(indexJS);
  page = jsRuntime.getObject("page");
  Log.i("J2V8", page.toString());

  // 获取page的data属性
  V8Object pageData = page.getObject("data");

  // 渲染初始UI
  mWebview.loadUrl("javascript:render('" + indexXML + "')");

  // Native调用V8中js函数，执行生命周期：onLoad
  page.executeFunction("onLoad", null);
  pageData.release();
}
```

## 总结

这是啥啊？这不是再造一个浏览器吗？！

不不，浏览器远比这复杂，并且js不能直接操作dom（~~也可以定义接口来实现~~），还有很多内容需要完善，例如实现js的window函数：setTimeout等。

至于性能方面，是不是比直接使用webview的性能要好，这还要进一步完善丰富原型和DEMO。