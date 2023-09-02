---
title: html设置元素水平对齐
name: html-horizon-align
date: 2019-06-30
id: 0
tags: [html,css]
categories: []
abstract: ""
---


假设行内有多个元素，需要其水平对齐

```html
    <input type="text" name="mess" autocomplete="off" required>
    <input class="btn" type="submit" name="submit" value="发布">
```

<!--more-->

只需要设置垂直属性

```css
input[type=text]{
    vertical-align: middle;
}
.btn {
    font-size: 28px;
    vertical-align: middle;
}
```

这样两个`input`标签就会水平中心对齐

