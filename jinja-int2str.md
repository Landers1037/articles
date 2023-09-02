---
title: jinja2模板把int参数转为str
name: jinja-int2str
date: 2019-08-26
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


使用Jinja2模板处理传入的参数时，如果是数字希望转换为字符串显示，可以不使用js函数处理

<!--more-->

使用Jinja2自带的转换符`~`

```html
<a href="{{ '/home/'~id }}"></a>
```

其中`id`是int型数据，使用`~`可以把前后的数据连接转化为字符串