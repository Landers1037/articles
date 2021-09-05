---
title: Scrapy常见错误一
name: scrapy1
date: 2019-03-08
id: 0
tags: [scrapy]
categories: []
abstract: ""
---


## 通用爬虫爬取页面深层链接

出现以下错误

```cmd
'str' object has no attribute 'iter'
```


<!--more-->


## 通用爬虫爬取页面深层链接

出现以下错误

```cmd
'str' object has no attribute 'iter'
```

<!--more-->查看css选择器的部分

```python
Rule(LinkExtractor(allow=r'/photo.*',restrict_css='li div a::attr(href)'),callback='parse_item')
```

解析：
scrapy的css提取链接不需要指定属性，如果链接存在于属性href中，则只需要提取包含它的元素a

```python
Rule(LinkExtractor(allow=r'/photo.*',restrict_css='li div a'),callback='parse_item')
```

