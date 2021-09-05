---
title: jquery滚动事件实用
name: jquery-scroll-event
date: 2019-08-10
id: 0
tags: [flask,html]
categories: []
abstract: ""
---


想要在网页上加一个点击按钮，平时隐藏

在滑动的时候显示，又要求滑动到底部时按钮隐藏，于是试试jQuery的滚动事件


<!--more-->


想要在网页上加一个点击按钮，平时隐藏

在滑动的时候显示，又要求滑动到底部时按钮隐藏，于是试试jQuery的滚动事件

<!--more-->

### 一：一般判定条件

第一次这样写

```javascript
var go = $("#goto");
var h = f.offset().top;
go.show()
if(h>$(document).height()-20){
    go.hide()
}
```

使用`offset()`获取按钮go到顶部的距离，判断如果距离大于特定值(文档高度-20px)就隐藏按钮

发现问题：按钮一滑到底部判定点就自动隐藏了然后就不显示了，因为写的是网页加载时运行的代码

改进：

```javascript
$(document).scroll(function () {
    var go = $("#goto");
	var h = f.offset().top;
	go.show()
	if(h>$(document).height()-20){
    	go.hide()
	}
});
```

使用jQuery判断滚动事件，一旦窗口发生滚动就加载js代码

发现问题：按钮一滑到底部判定点就开始闪烁，因为初始态按钮是`show()`的，在判定点到达时按钮`hide()`，一旦滑动事件发生又会先显示再隐藏。

改进：

```javascript
$(document).scroll(function () {
    var go = $("#goto");
	var h = f.offset().top;
	if(h>$(document).height()-20){
    	go.hide()
	}
    else{
        go.show()
    }
});
```

这样每次滑动的时候判断高度，不会出现闪烁，但是会出现按钮到达底部后继续显示的问题，发现是因为按钮第一次隐藏后`display: none`进行相对高度差计算时`offset`就不存在

### 二：使用滚动条高度判断

不使用按钮与文档顶部距离的方式判断，改用判断滚动条是否到达底部来判断

```javascript
$(document).scroll(function () {
            var f = $("#goto");

            var height = $(document).height();
            var wheight = $(window).height();
            var cha = $(document).scrollTop();
            if(cha < height-wheight-20){
                f.show()
            }
            else {
                f.hide()
            }
});
```

通过计算文档的总体高度和窗口的高度，计算差值。`scrollTop()`是滚动条滚动到顶部的距离

