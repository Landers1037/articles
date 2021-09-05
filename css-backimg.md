---
title: css为div添加背景图片
name: css-backimg
date: 2019-07-07
id: 0
tags: [html]
categories: []
abstract: ""
---


使用css为div块设置背景图片
`background: url(/img/0.png)`


<!--more-->


使用css为div块设置背景图片
`background: url(/img/0.png)`

<!--more-->

使用`background`属性

```css
div{
   background: url(/img/0.png) 
}
```

为图片添加属性

#### 图片起始位置

```css
div{
   background: url(/img/0.png) 5px 0px
}
```

`5px`是左间距，`0px`是顶部间距

#### 居中属性

```css
div{
   background: url(/img/0.png) center 0px
}
```

#### 图片不重复

```css
div{
   background: url(/img/0.png) no-repeat
}
```

其他属性：`repeat-x`横向平铺，`repeat-y`纵向平铺

#### 设置图片随着div的大小动态变化

```css
div{
   background: url(/img/0.png);
   background-size:  contain;
}
```

其他属性：`auto`自动变化，`cover`覆盖div的大小

