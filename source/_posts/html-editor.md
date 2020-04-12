---
title: 抛弃contenteditable，实现富文本编辑器
date: 2020-04-03 17:45:17
tags:
---

> 有时候使用`contenteditable=true`来实现编辑器并不能满足需求

## 移动端使用`contenteditable=true`的一些不成问题的问题

使用`contenteditable=true`实现编辑器，经过不断的优化，是可以解决很多问题的。下面一一列出一些问题及其解决方法。

### 问题一：插入超链接和图片

如果不是复制粘贴的方式插入，而是采用点击工具栏，编辑超链接或者是调用相册的方式插入，则中间会出现焦点丢失，那么如何实现插入的图片在焦点的位置？
解决方法是：

- 1、在点击用户工具栏时，记录焦点位置。截获用户点击最好采用`touchstart`事件，避免记录的焦点位置不是编辑区内。
  
  ```js
  var selection = getSelection()
  this.globalRange = selection.getRangeAt(0)
  ```

- 2、在获取到图片资源后，利用`this.globalRange`插入图片节点

  ```js
  // 创建图片节点
  var imgNode = this.createImgNode(src)
  this.globalRange.insertNode(imgNode)
  ```

### 问题二：光标位置

有些情况下，你可能需要知道光标的位置，top、left是多少。可以使用`getSelection().focusNode()`获取到聚焦的节点，有时得到的节点是text类型（没有top、left属性），可以通过通过其父节点大致得到位置。所以，只是得不到准确的位置。

为什么要得到光标位置？iOS端键盘弹出后，`position:fixed`的效果发生了变化，难以固定dom。所以，如果存在工具栏，可以将工具栏跟随光标，提升一点点体验。

### 问题三：删除图片

需求的设计稿是要求图片左上角有一个X按钮，来删除图片。

<div style="text-align: center;">
  <img src="del-img.png">
</div>

如果不对这个这个删除按钮添加属性，那么用户可以编辑这个按钮，虽然影响不大，导致这个删除按钮的意义荡然无存。
讲道理，这个删除按钮应该不能编辑，所以，在删除按钮和图片上加上属性`contenteditable="false"`。但是，这又会导致其他问题：插入图片后，我的光标聚焦到哪里？按钮删除图片也可能导致一些难以预料的问题。

加不加`contenteditable="false"`是矛盾的。也正是这样一个矛盾的存在，导致了我们不使用`contenteditable="true"`来实现编辑器，可以说是一个X引发的大型~~搬砖事件~~富文本重构。

## 自实现富文本编辑器需要确定的数据结构

我们需要获取用户输入，所以可以用隐藏的`textare`或者`input`等来获取输入。首先需要确定的是数据结构（内容和光标），以便实现自动换行、行内富文本，渲染内容

### 数据结构：内容

网页文本有自动换行的特性，在移动端则更为明显，因为屏幕更小。另外，行内还要支持行内图片、超链接等数据。所以，基本可以确定是，行内要包括多种类型的数据。

```json
// line
{
  "inlines": [
    {
      "type": "text",
      "value": ""
    },
    {
      "type": "href",
      "value": "超链接"
    },
    {
      "type": "img",
      "value": "图片src"
    }
  ]
}
```

如果一个div内文本或图片等内容足够多，那么会被自动换行，此时，如果用户聚焦到该div进行编辑，我们其实很难计算用户点击了哪里，很难计算行末尾的位置，也难以计算光标位置。

<div style="text-align: center;">
  <img src="long-content-div.png">
</div>

如果将一个div内的内容看成很多行的合集，那么我们能知道每一行的宽度，是否占满了一行。

<div style="text-align: center;">
  <img src="long-content-div_line.png">
</div>

根据自动换行，如果没占满一行，则要向下一行借，补足一行。而两个div之间，则不存在前一行向下一行借的情况。
所以，我们可以将div视为一个段落，段落每行自动占满一行。那么每一段又分为很多行。
段分为两种类型：图文混排的**rich**和独占一段的**block**

```json
// 段
{
  "type": "rich",
  "lines": [
    { // line
      "inlines": [
        {
          "type": "text",
          "value": ""
        },
        {
          "type": "href",
          "value": "超链接"
        },
        {
          "type": "img",
          "value": "图片src"
        }
      ]
    }
  ]
}
```

所以，富文本是很多段的组成，和word文档一样。

```json
[
  { // 段
    "lines": []
  }
]
```

将数据结构渲染出来，大致的DOM结果为：

```html
<div class="paragraph">
  <div class="line">
    <span class="inline_text">文本</span><img class="inline_img" />
  </div>
  <div class="line">...</div>
</div>
<div class="paragraph"></div>
```

直观的效果便是：

<div style="text-align: center;">
  <img src="ui-struct.jpg">
</div>

### 数据结构：光标

定义好数据结构，意味着如何计算光标位置也有了方向。光标的位置采用`postion: absolute;`来定位，所以富文本编辑器DOM应该是`postion: releative;`
如果存在以下一段内容。

```json
[
  // 段
  {
    "lines": [
      { // line 第一行
        "inlines": [
          {
            "type": "text",
            "value": "富文本编辑器"
          }
        ]
      }
    ]
  }
]
```

那么，我们想定位到第一行行末，那么光标的left值则是**富文本编辑器**的宽度，也就是`<span>富文本编辑器</span>`宽度，光标的高度也可以得到。而top值则只需要在渲染出来的DOM中，找到第一段第一行的`offsetTop`。所以，如果是要将光标定位到**富文本**后，则top值不变，left值为**富文本**的宽度。

我们要总结一下光标所具有的属性：段、行，以及行内位置，行内位置则需要记录所在行内文本的字符在第几个字符后面。所以基本属性为

```js
cursor = {
  top: 0,
  left: 0,
  // 所在段下标
  paragraphIndex: 0,
  // 行下标
  lineIndex: 0,
  // 行内下标
  inlineIndex: 0,
  // 字符位置
  charAt: 0
}
```

这帮助我们确认了光标的位置，在内容中的位置，以便进行内容的增、删操作。

突然想到我们可以看看vscode的源码，看它是如何利用electron来实现换行、光标定位。我们虽然分析过ace.js，但是其对代码是不自动换行的，虽然vscode也是如此，但是对markdown却是自动换行的。

## 层次结构

我们需要将编辑的内容、光标、隐藏的textarea等渲染出来。另外还需要计算字符宽度来计算光标位置。所以，我们需要将编辑区域分为以下几层：

```html
<div class="content">渲染内容层</div>
<div class="cursor">光标层</div>
<textarea>隐藏的输入层</textarea>
<div class="measure">计算层</div>
```

- 渲染内容层：如名
- 光标层：显示光标
- 隐藏的输入层：获取用户输入
- 计算层：计算字符宽度，行高等。

## 逻辑

逻辑涉及到：渲染、输入、点击

### 渲染内容

渲染内容上文有提到，只要每一行的内容不超过一行的行宽，渲染出来的结构基本如上。而保证每一行的行高，则需要计算每一行的内容，在后面获取用户输入会提到。

### 获取用户输入

获取用户输入，需要监听两种事件：input、keydown，处理四种基本状态：insertText、compositionend、Backspace、Enter

#### 事件：input

input需要监听的基本事件：insertText、compositionend。两者都是插入字符，compositionend则是连续输入的结束状态，例如拼音输入。

在光标位置插入字符，并将聚焦的inline右侧的inlines和后续行均进行后移，也就是重新分行，这是整个编辑器的核心，删除、换行、插入都是基于此流程来实现。具体流程为：

<div style="text-align: center;">
  <img src="insert-str-program.png">
</div>

#### 事件：keydown

键盘按键事件，移动端最基本的是需要处理两种key：Backspce、Enter

##### Backspace:删除

删除需要将cursor前的字符或者内容删除，同时将聚焦的inline右侧的inlines和后续行均进行缩进

<div style="text-align: center;">
  <img src="ui-del.jpg">
</div>

删除字符需要注意的是：字符长度不一定为1，一些表意文字，例如输入法的表情😄，其字符长度是2。所以需要用正则匹配光标前的字符。

##### Enter:换行

换行是将光标后的内容插入到新段中。

### 点击

鼠标的点击分为两种：聚焦和删除（点击了删除图标）

#### 聚焦

在渲染内容时将paragraph、line、inline的信息都记录到dom中

```html
<div class="paragraph" data-paragraph="0">
  <div class="line" data-paragraph="0" data-line="0">
    <span class="inline_text" data-paragraph="0" data-line="0" data-inline="0">文本</span><img class="inline_img" data-paragraph="0" data-line="0" data-inline="1"/>
  </div>
  <div class="line" data-paragraph="0" data-line="1">...</div>
</div>
<div class="paragraph" data-paragraph="1"></div>
```

通过点击的target，获取聚焦的inline，通过点击的offsetY确定光标top位置，offsetX（相对target的偏移）计算出光标left位置。left计算方法，则是依次将inline左侧的字符取出，得到字符宽度与offsetX进行比较。

#### 删除

鼠标点击的删除，主要是针对段type="block"。删除则是删除整段，更新DOM。

## 总结

比较简单的富文本还是建议使用contenteditable=true来实现。如果在移动端，则建议不要采用富文本的思想，而是文本与图片分离的数据结构。
