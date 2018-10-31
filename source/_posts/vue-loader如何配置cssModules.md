---
title: vue-loader如何配置cssModules
date: 2018-10-31 19:16:32
tags: vue-cli
---

在使用vue cli 2系列进行模块编程时，为避免css冲突，我们常采用vue-loader的scoped和module方法来实现。

## scoped
scoped使用方法最为简单，只需要在`.vue`文件的`<style>`标签增加`scoped`属性即可。
```js
<template>
  <div class="example">hi</div>
</template>

<style scoped>
.example {
  color: red;
}
</style>
```

#### scoped:注意⚠️
但是会有一个问题：子组件的跟节点会收到父组件的影响，这也是scoped本身的特性决定的。例如：
我们在子组件的根结点设置`wrap` class，在父组件中也设置`wrap` class。
```js child.vue
<template>
  <div class="wrap">test</div>
</template>

<style scoped>
.wrap{
  margin-top: 10px
}
</style>
```

```js parent.vue
<template>
  <div class="wrap">
    <child />
  </div>
</template>

<script>
import Child from './child'
export {
  components: {
    Child
  }
}
</script>

<style scoped>
.wrap{
  padding-top: 20px
}
</style>
```

效果为：
<div style="text-align: center;">
  <img src="scoped.png"/>
</div>

解决方法：
  - 改子组件或父组件的class名：改名字是不阔能的，这辈子都不阔能的。如果要改，我们要模块化css有何用？
  - 子组件外再包裹一个无class的元素：确实能解决，但是这种方法我是拒绝的，因为会增加dom

## CSS Modules

说到上面的问题，CSS Modules就可以解决：CSS Modules会给每个clsss生成一个唯一的class名，其依据了文件路径等信息，采用hash加随机数得出唯一的class名。
在`.vue`文件中使用方式如下：
```js
<template>
  <div :class="$style.wrap"></div>
</template>

<style module>
.wrap{
  margin: auto;
}
</style>
```
但是我会小改一下，因为懒得每次写`$style`
```js
<template>
  <div :class="s.wrap"></div>
</template>

<style module="s">
.wrap{
  margin: auto;
}
</style>
```

## 本文中心思想：表达了作者...
在vue-cli 2系列中使用CSS Modules时，如果仅仅采用上述方式，会发现很难进行调试了：完全不知道网页渲染出的class对应源码中的哪一个。

所以需要在`vue-load.conf.js`文件中进行配置，来告诉vue-loader如何生成class名。作者通常采用以下配置来命名：
```js
cssModules: {
  localIdentName: '[local]-[hash:base64:5]', // 指定命名采用：局部名-5位的hash
  camelCase: true
}
```
指定命名完整参考为
```js
localIdentName: '[path][name]---[local]---[hash:base64:5]'
```
命名具体可以参考[postcss-modules](https://github.com/css-modules/postcss-modules)，以及[vue-loader v14](https://vue-loader-v14.vuejs.org/en/features/css-modules.html)