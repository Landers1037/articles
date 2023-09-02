---
title: markdown图片并列显示
name: let2pic-parallel
date: 2019-05-24
id: 0
tags: [hexo]
categories: []
abstract: ""
---


markdown中使用html标签让两张图片并排显示

```html
<table><tr>
<td><img src=pic1.png border=0></td>
<td><img src=pic2.png border=0></td>
</tr></table>
```

<!--more-->

当使用本地图片资源时满足markdown语法

```html
<table><tr>
<td>![1](/images/alipay.png)</td>
<td>![2](/images/wechatpay.png)</td>
</tr></table>
```

补充居中方法

图片居中

```html
<div style="text-align: center"><img src=pic1.png border=0></div>
```

a标签居中

因为a标签的特殊性，是一个带有跳转的链接，需要先转换为行内块元素

```css
a {
    display: inline-block;
    font-size: 12px;
    font-weight: bold;
    color: black;
    text-decoration: none;
}
```

```html
<div style="text-align: center"><a href="#">test</a></div>
```

