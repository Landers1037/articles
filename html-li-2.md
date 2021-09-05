---
title: html中li节点并排方法
name: html-li-2
date: 2019-06-30
id: 0
tags: [html,css]
categories: []
abstract: ""
---


`<li></li>`节点默认是不能并排的

可以使用block块或者悬浮样式的方法让其并排显示


<!--more-->


`<li></li>`节点默认是不能并排的

可以使用block块或者悬浮样式的方法让其并排显示

<!--more-->

### 示例

```html
<ul> 
<li>测试</li> 
<li>测试</li> 
<li>测试</li> 
</ul> 
```



### 方法一

使用行内块元素，调整css样式

```css
li{ display:inline}
```

### 方法二

使用悬浮样式

```css
li{ 
    float:left; 
    list-style:none
}
```

`list-style`是节点样式，不去除就会显示标签节点

`float:left`表示左对齐

### 间距调整

并排显示后需要调整间距使两个`li`节点间有空隙，添加`margin`属性

```css
li{
    margin-right：10px
}
```

或

```css
border-right: 背景颜色
```

设置与背景颜色一致达到类似空格的效果