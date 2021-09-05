---
title: Jinja2自定义过滤器使用
name: jinja-filter
date: 2019-08-27
id: 0
tags: [flask]
categories: []
abstract: ""
---


Jinja2自带的过滤器有时候不能满足需求，可以自定义过滤器实现复杂效果


<!--more-->


Jinja2自带的过滤器有时候不能满足需求，可以自定义过滤器实现复杂效果

<!--more-->

### 创建过滤器

```python
# 自定义过滤器
def time_fil(str):
    result = str.strftime("%Y-%m-%d %H:%M:%S")
    return result
```

传入的`str`是**datetime**对象，使用自定义过滤器转换为字符串

### 注册过滤器

```python
app.jinja_env.filters["time_fil"] = time_fil
```

### 调用

```html
#在html模板中调用
<span>{{ img.timestamp|time_fil }}</span>
```

### 常用过滤器

| 过滤器               | 说明                                       |
| -------------------- | :----------------------------------------- |
| safe                 | 渲染时不转义                               |
| tojson               | 字符串转换为json对象                       |
| wordcount            | 统计长字符的单词数                         |
| truncate(length=255) | 按长度截取字符串                           |
| capitalize           | 把值的首字母转换成大写，其他字母转换成小写 |
| lower                | 把值转换成小写形式                         |
| upper                | 把值转换成大写形式                         |
| title                | 把值中每个单词的首字母都转换成大写         |
| trim                 | 把值的首尾空格去掉                         |
| striptags            | 渲染前把值中所有的HTML标签都删掉           |
| reverse              | 反转值生成逆序列表                         |
| round                | 四舍五入取整                               |
| floor                | 取到小数点后两位                           |
| first                | 取列表里的第一个值                         |
| last                 | 取列表里最后一个值                         |
| length               | 返回列表长度                               |
| sum                  | 列表求和                                   |
| sort                 | 按照升序排列                               |
| join("符号")         | 合并为字符串，符号为空时没有间隔           |

[全部内置过滤器](http://docs.jinkan.org/docs/jinja2/templates.html#builtin-filters)

