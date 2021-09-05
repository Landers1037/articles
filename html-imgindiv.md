---
title: html设置div内部图片自适应
name: html-imgindiv
date: 2019-07-31
id: 0
tags: [html]
categories: []
abstract: ""
---


假如有一个div元素块，里面有一个img标签，如何使图片跟随div变化，充满整个div块


<!--more-->


假如有一个div元素块，里面有一个img标签，如何使图片跟随div变化，充满整个div块

<!--more-->

### 利用两个元素块约束

```html
<div class="out">
   <div class="in">
     <img src='static/img/face-2.jpg'>
   </div>
</div>
```

css样式表

```css
.in{
    position:relative;
    width:100%;
    height:0;
    padding-top: 100%; /*padding-bottom都可以*/
    img{
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
  }
```

### 使内部图片居中

#### 使用flex布局居中

```css
dispaly: flex;
```

#### 使用transform属性

`transform()`对x，y方向进行偏移
`transformY()`对y方向偏移
`transformX()`对x方向偏移

首先使用绝对位置，`top: 50%`，图片最上方从div父元素的50%高度开始

```css
img{
    transform: translateY(-50%);
}
```

对图片添加属性，在y方向上，偏移自身高度50%，达到垂直居中的目的

#### 水平居中

使用`margin: 0 auto`属性