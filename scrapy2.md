---
title: 使用scrapy特定爬虫爬取不同的网页
name: scrapy2
date: 2019-03-18
id: 0
tags: [scrapy]
categories: []
abstract: ""
---


在我的不断摸索下，发现当只想使用一个spider就想爬取不同的网站时，可以直接对parse方法进行条件判断<!--more-->

```python
# 常见的parse函数用于解析request的url
# starturls可以是一个列表，里面可以放入不同的网址，记住这个时候要把顶级的allowed domin注释掉

def parse(self, response):
            if response.url == 'https://my.ss8.fun/':
                return scrapy.Request(response.url,callback=self.parse_ss8)
            elif response.url == 'https://ttizi.com/':
                return scrapy.Request(response.url,callback=self.parse_ttizi)
            elif response.url == 'https://d.ishadowx.com/':
                return scrapy.Request(response.url,callback=self.parse_ishadow)


    def parse_ttizi(self,response):
        html = response.css('.card-body')
        for data in html:
            item = ImageItem()
            item['url'] = data.css('a:nth-child(2)::attr(href)').extract_first()
            yield item

    def parse_ishadow(self,response):
        html = response.css('.hover-text')
        for data in html:
            item = ImageItem()
            try:
                item['url'] = 'https://d.ishadowx.com/'+data.css('h4 a::attr(href)').extract_first()
                yield item
            except:
                continue

    def parse_ss8(self,response):
        html = response.css('article')
        for data in html:
            item = ImageItem()
            item['url'] = 'https://my.ss8.fun/'+data.css('a::attr(href)').extract_first()
            yield item
```

利用parse中的判断条件判断处理的url为哪一个网址，然后callback中的函数调用的就是专门处理这个网站的方法

注意方法start_requests（）方法的覆盖不能用于判断，它返回的必须是一个迭代器型的url列表