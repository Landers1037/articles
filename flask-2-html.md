---
title: 如何使用js把字符串转变为html文本
name: flask-2-html
date: 2019-07-30
id: 0
tags: [html,flask]
categories: []
abstract: ""
---


当数据需要动态从数据库或者存储文件中调用并传入flask中，路由传入的是字符串类型，如果希望传入的数据在html文件中作为一段内容显示，需要使用JavaScript对字符串里的html标签转义


<!--more-->


当数据需要动态从数据库或者存储文件中调用并传入flask中，路由传入的是字符串类型，如果希望传入的数据在html文件中作为一段内容显示，需要使用JavaScript对字符串里的html标签转义

<!--more-->

### 添加容器

在html文件中为传入的字符创建一个容器

```html
<div class="post">
    <p class="text">
        {{ text }}
    </p>
</div>
```

text是传入的带有html标签的字符串，给p标签添加类名"text"

### 字符串

假设传入一段字符串为

```html
"这是一段测试代码<a href="#">点击</a>"
```

不经过处理直接显示出来是

```html
这是一段测试代码<a href="#">点击</a>
```

我们需要的样式如下

这是一段测试代码<a href="#">点击</a>

### 使用js处理

添加JavaScript函数

```javascript
<script>
    window.onload=function () {
        var list = document.getElementsByClassName("text");
        for (var i = 0; i < list.length; i++) {
            text = list[i].innerText;
            list[i].innerHTML = text;
        }
    }
</script>
```

使用css过滤器筛选出所有的类名为"text"的元素，得到的是一个节点列表
对节点列表按照长度进行遍历，每一个元素都使用`innerHTML()`函数将文本重新格式化

