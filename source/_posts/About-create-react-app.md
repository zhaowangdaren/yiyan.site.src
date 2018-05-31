---
title: 'About:create-react-app'
date: 2018-05-22 15:27:09
tags: react
---

> create-react-app是facebook官方维护的一套react开发框架（emmm...应该说是配置），目前的star数量接近5w+，可见其受欢迎程度。相比于其他一些react开发框架，例如react-boilerplate、react-starter-kit(star数都接近2w+)，create-react-app似乎更容易react初学者上手。

## HMR(hot module replace)
在使用的过程中，发现create-react-app不能做到HMR，每次修改代码都是整个页面刷新。这不可能，这么受欢迎的项目，怎么没有默认开启HMR？

在运行npm run eject后，发现在其配置文件webpack.config.dev.js文件中已经引用相应的webpack配置：

```js
// Add module names to factory functions so they appear in browser profiler.
new webpack.NamedModulesPlugin(),
// Makes some environment variables available to the JS code, for example:
// if (process.env.NODE_ENV === 'development') { ... }. See `./env.js`.
new webpack.DefinePlugin(env.stringified),
// This is necessary to emit hot updates (currently CSS only):
new webpack.HotModuleReplacementPlugin(),
```

同时在webpackDevServer.config.js文件中也配置了hot：

```js
// Enable hot reloading server. It will provide /sockjs-node/ endpoint
// for the WebpackDevServer client so it can learn when the files were
// updated. The WebpackDevServer client is included as an entry point
// in the Webpack development configuration. Note that only changes
// to CSS are currently hot reloaded. JS changes will refresh the browser.
hot: true,
```
对比webpack官网，需要在入口的js文件index.js文件中增加：
```js
if (module.hot) {
  module.hot.accept()
}
```

## BrowserRouter与HashRouter
vue-router有两种模式：hash模式和history模式。history模式需要服务端配合，否则会有一些小问题。

react-router中也存在这两种模式。BrowserRouter即为history模式，HashRouter即为hash模式。我们在部署的时候，如果将项目放在web服务器静态资源的二级目录中，则往往需要修改web服务器或者改用HashRouter，否则会出现找不到资源的情况。

## react函数式组件，事件处理
对于简单的组件，可以采用函数式来定义react组件。函数式组件可以简单快速的定义一个react组件，但是有一个缺点是：无状态。这里提供一个简单的事件处理demo：
```js
import React from 'react'
import s from './index.less'
import close from '../../assets/imgs/close.png'

function Footer (props) {
  const handleClose = () => {
    document.querySelector('#footer').style.display = 'none'
  }
  return (
    <footer className={s.footer} id='footer'>
      <img
        onClick={handleClose}        
        src={close}
        className={s.close}/>
    </footer>
  )
}

export default Footer
```