---
title: css-flex布局
name: css-flex
date: 2020-04-17
id: 0
tags: [html]
categories: []
abstract: ""
---


CSS的Flex布局

<!--more-->

为一个盒子模型设置为flex布局

```html
<div style="display: flex">
    hello
</div>
```

>  设为Flex布局以后，子元素的float、clear和vertical-align属性将失效                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        

预览

<div style="display: flex">hello</div>

### 概念

使用flex布局的元素成为**容器**container，其子元素成为**项目**item

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做cross start，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做`cross size`。

### 容器属性

容器有6个属性，`flex-direction`,`flex-wrap`,`flex-flow`,`justify-content`,`align-items`,`align-content`

**flex-direction**

决定主轴的排列方向，默认是水平，起点在最左端。可选参数row | row-reverse | column | column-reverse

```html
<div>
    <div style="display: flex;flex-direction: row">
        left
    </div>
    <div style="display: flex;flex-direction: row-reverse">
        right
    </div>
</div>
```

预览

<div>
    <div style="display: flex;flex-direction: row">
        left
    </div>
    <div style="display: flex;flex-direction: row-reverse">
        right
    </div>
</div>

**flex-wrap**

定义项目在flex布局里排列时该如何换行，默认时项目是排列在一条线上

可选参数flex-wrap: nowrap | wrap | wrap-reverse

分别代表不换行，换行后第一行在上方，换行后第一行在下方

```html
<div>
    <div style="display: flex;flex-wrap: nowrap">
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
    </div>
</div>
```

预览

<div>
    <div style="display: flex;flex-wrap: nowrap">
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
        <span>1</span>
    </div>
</div>

**flex-flow**

flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为`row nowrap`。

**justify-content**

定义了项目在主轴上的排列方式，可选flex-start | flex-end | center | space-between | space-around

假设主轴是从左到右的

- flex-start（默认值）：左对齐
- flex-end：右对齐
- center： 居中
- space-between：两端对齐，项目之间的间隔都相等。
- space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

```html
<div style="display: flex;justify-content: center">
    <span>hello</span>
    <span>world</span>
</div>
<!--居中对齐-->
```

预览

<div style="display: flex;justify-content: center">
    <span>hello</span>
    <span>world</span>
</div>

**align-items**

定义了项目怎么在交叉轴对齐，可选flex-start | flex-end | center | baseline | stretch

假设默认的交叉轴是从上到下

- flex-start：交叉轴的起点对齐。
- flex-end：交叉轴的终点对齐。
- center：交叉轴的中点对齐。
- baseline: 项目的第一行文字的基线对齐。
- stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

```html
<div style="display: flex;align-items: center;background: yellow;height: 100px;color: black">
    <span>hello</span>
    <span>world</span>
</div>
<!--垂直居中对齐-->
```

预览

<div style="display: flex;align-items: center;background: yellow;height: 100px;color: black">
    <span>hello</span>
    <span>world</span>
</div>

**align-content**

定义多根轴线存在时的对齐方式，只存在一根轴线时不起作用。

可选参数flex-start | flex-end | center | space-between | space-around | stretch

- flex-start：与交叉轴的起点对齐。
- flex-end：与交叉轴的终点对齐。
- center：与交叉轴的中点对齐。
- space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
- space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
- stretch（默认值）：轴线占满整个交叉轴。



### 项目属性

项目属性有6个，`order`,`flex-grow`,`flex-shrink`,`flex-basis`,`flex`,`align-self`

**order**

定义项目的排列顺序，默认0，数值越小排列越靠前

**flex-grow**

flex-grow属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。

**flex-shrink**

flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

负值对该属性无效。

**flex-basis**

定义在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

**flex**

flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。

**align-self**

align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

```html
<div style="display: flex;align-items: center;background: yellow;height: 100px;color: black">
    <span>hello</span>
    <span style="align-self: flex-end">world</span>
</div>
```

预览

<div style="display: flex;align-items: center;background: yellow;height: 100px;color: black">
    <span>hello</span>
    <span style="align-self: flex-end">world</span>
</div>

`hello`根据父元素设置为垂直交叉轴中心对齐，`world`重写了align属性变为交叉轴末端对齐

### 参考

[CSS flex布局语法](https://www.runoob.com/w3cnote/flex-grammar.html)