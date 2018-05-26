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