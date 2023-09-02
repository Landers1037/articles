---
title: css设置元素自动居中
name: html-auto-center
date: 2019-07-05
id: 0
tags: [html]
categories: []
abstract: ""
---


使用css设置html网页中的元素居中

<!--more-->

## 第一种居中方法

使用`text-align: center`

假如网页结构如下

```html
    <div class="container">
        <td class="test"><img src="/static/alipay.png"></td>
    </div>
```

设置`.container`的属性`text-align: center`

对于特殊标签`a`,`li`需要标注其为块元素

```css
a{
    display: inline-block;
}
```

## 

## 第二种居中方法

使用`margin: 0 auto`自动确定位置

```html
<div class="foot">
    <li class="love">❤️</li>
    <li class="zan">👍</li>
</div>
```



```css
.foot{
    margin: 0 auto;
    margin-bottom: 20px;
}
```

