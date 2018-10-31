---
title: '深入解析vue-cli 2.9.*实现原理'
date: 2018-10-08 17:10:03
tags: vue-cli-2.9
---

> vue-cli 3.0已出，你为什么还要看这个？

## Vue-cli项目结构
<div style="text-align: center;">
  <img src="files.png">
</div>

其中，我们主要关注点在`package.json\bin\lib`这3个地方。

### package.json
`package.json`中主要关注`bin`:
```json
"bin": {
  "vue": "bin/vue",
  "vue-init": "bin/vue-init",
  "vue-list": "bin/vue-list"
}
```
nodejs在解析命令时主要通过这个`bin`来运行相应的文件。`"vue":"bin/vue"`nodejs会运行bin文件夹下的`vue`文件，同理，其他两个命令也一样。

###  bin/vue
该文件内容比较少，其实整个脚本中代码量都比较少，因为模板内容都是从github上下载的。
```js
#!/usr/bin/env node

const program = require('commander')

program
  .version(require('../package').version)
  .usage('<command> [options]')
  .command('init', 'generate a new project from a template')
  .command('list', 'list available official templates')
  .command('build', 'prototype a new project')
  .command('create', '(for v3 warning only)')

program.parse(process.argv)
```
这里列出了运行`vue`命令时的提示。如果在控制台运行`vue init`则会出现如下内容：
<div style="text-align: center;">
  <img src="console-vue-init.png">
</div>

##### 疑惑：这个是哪来的?`bin/vue`文件中并没有这些内容。正是这个疑惑导致了这个文章的出现。
我们先来看一下`package.json`的bin下的`vue-init`命令，其运行的是`bin/vue-init`

### bin/vue-init
我们来看一下该文件的一部分内容
```js
#!/usr/bin/env node
const program = require('commander')

/**
 * Usage.
 */

program
  .usage('<template-name> [project-name]')
  .option('-c, --clone', 'use git clone')
  .option('--offline', 'use cached template')

/**
 * Help.
 */

program.on('--help', () => {
  console.log('  Examples:')
  console.log()
  console.log(chalk.gray('    # create a new project with an official template'))
  console.log('    $ vue init webpack my-project')
  console.log()
  console.log(chalk.gray('    # create a new project straight from a github template'))
  console.log('    $ vue init username/repo my-project')
  console.log()
})
```
这一段正是`vue init`命令给出的提示。我们在控制台运行`vue-init`，也会出现同`vue init`一样的结果。难道`vue init`和`vue-init`两个命令相同？nodejs支持这样不同的命令运行相同的脚本？

##### "vue init" != "vue-init"，使他们运行同一个脚本的另有原因

仔细分析，老司机就会发现其中共同的一点：`const program = require('commander')`。正是这个`commander`包实现了`"vue init" == "vue-init"`。

#### commander
commander是node.js 命令行接口的完整解决方案，灵感来自 Ruby 的 commander。其有Git风格的子命令，当使用`.command()`带有描述参数时，这告诉commander，你将采用当单独的可执行文件作为子命令。commander将会尝试在入口脚本的目录中搜索`program-command`形式的可执行文件，例如`bin/vue`文件中的`.command('init', 'generate a new project from a template')`，会使commander在bin文件夹下查找vue-init的可执行文件。

到这里，文章可以结束了。

## vue-cli创建模板逻辑
逻辑还是相对简单的，利用`download-git-repo`下载模板，`download-git-repo`默认从GitHub上下载。