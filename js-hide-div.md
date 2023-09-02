---
title: javascript动态显示块元素
name: js-hide-div
date: 2019-08-03
id: 0
tags: [html]
categories: []
abstract: ""
---


块元素需要在加载时使用动画可以使用JavaScript实现
原生js写法繁琐，使用jQuery实现

<!--more-->

首先引入jQuery

```html
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
```

#### 设置需要加载动画的元素块id

```html
<div id="test" class="test">
    <span>加载动画</span>
</div>
<a id="button">点击显示</a>
```

#### 设置div块的初始样式

```css
.test{
	display: none;
	width: 200px;
}
```

#### js代码

```javascript
$("#button").hover(function{
                 $("#test").toggle();
                 });
```

上面代码实现的是鼠标移动到标签a上时id为`"test"`的块显示，移开时隐藏

改为点击

```javascript
$("#button").click(function{
                 $("#test").toggle();
                 });
```

#### 效果

原生的JavaScript支持的效果有`show()`,`hide()`,`fadein()`,`fadeup()`,`fadedown()`,`slide()`等

可以更改为滑入效果

```javascript
$("#button").hover(function{
                 $("#test").slideDown();
                 });
```

添加参数，可以修改动画的速度

`slideDown(1000)`表示整个动画耗时1000毫秒