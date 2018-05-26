---
title: 'H5:echarts的tooltip与datazoom的优化'
date: 2018-05-25 13:06:37
tags: chart
---
> echarts版本3.X.X
<div style="text-align: center">
  <img src="h5-echarts-kchart.png">
</div>
在移动端同时使用Echarts的tooltip和datazoom时，你会发现tooltip显示的时候很难去对图表进行放大、缩小、水平移动，也就是比较难触发datazoom事件，用户体验有点差。因此，本文提出了一些优化方法，以便提高用户体验。

## 1. 交互套路
既然要优化，那就需要将tooltip和datazoom这两个事件拆分开，不能让它们同时触发。所以，交互的总方针是：
1. tooltip隐藏时，即允许datazoom也允许显示tooltip；
2. 点击，tooltip显示，datazoom无效；再次点击，tooltip隐藏，允许datazoom
3. 只有tooltip隐藏时，才允许datazoom。

## 2. 实现
实现时需要接管tooltip的触发，同时也要控制datazoom是否可用

### 2.1 接管tooltip

#### 2.1.1 echarts配置
echarts在配置tooltip提示框触发的条件：
```js
tooltip: {
  triggerOn: 'none'
}
```
不在'mousemove'或'click'时触发，我们改用dispatchAction来触发tooltip显示。

#### 2.1.2 主动触发tooltip，允许datazoom
我们需要处理图表dom标签的Touch事件，从而来主动分发showtip事件。按照我们的交互总方针，以下流程中的每一次允许tooltip，都伴随这禁用datazoom；每一次隐藏tooltip都伴随着启用datazoom。
处理Touch事件以及分发showtip的基本流程为：
##### 2.1.2.1 touchstart事件
1. 记录触发了touchstart
2. 记录并判断是不是多个手指，若是则隐藏tooltip，并允许datazoom。
3. 触发长按计时器，如果是长按，则要触发tooltip，并禁用datazoom。用户移动手指，来达到连续显示tooltip。

##### 2.1.2.2 touchmove事件
1. 记录触发了touchmove事件
2. 如果tooltip是不显示的状态，则要清除长按计时器，因为此时用户想要移动图表。并且，前提在tooltip是不显示的状态时，允许datazoom。
   如果tooltip是显示状态，则用户要达成连续显示tooltip。
3. 如果是长按，则需要显示tooltip

##### 2.1.2.3 touchend事件
1. 清除长按计时器；
2. 如果是多个手指操作，则函数结束，剩余流程不用处理；
3. 如果是长按，则显示tooltip，重置touch事件的状态，并return；
4. 2和3的条件都没满足的情况下，则表示用户是单次点击，如果tooltip是隐藏状态，则显示tooltip；如果tooltip是显示状态，则隐藏tooltip；
5. 重置touch状态。

### 2.2 datazoom处理
datazoom的处理只需要注意一个地方：拖动图表后，会触发touchend事件，可能会触发tooltip。所以，如果需要显示tooltip，则需要将在业务逻辑里面，监听到datazoom后，隐藏tooltip。
