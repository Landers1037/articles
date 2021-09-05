---
title: Scrapy+selenium动态网页抓取
name: spider8
date: 2019-03-07
id: 0
tags: [爬虫,scrapy]
categories: []
abstract: ""
---


# scrapy+selenium

源代码地址 [github](https://github.com/Landers1037/PySpider/tree/master/bilibili%E9%AC%BC%E7%95%9C%E8%A7%86%E9%A2%91%E8%AF%A6%E6%83%85%E7%88%AC%E5%8F%96/kichiku)
<!--more-->


# scrapy+selenium

源代码地址 [github](https://github.com/Landers1037/PySpider/tree/master/bilibili%E9%AC%BC%E7%95%9C%E8%A7%86%E9%A2%91%E8%AF%A6%E6%83%85%E7%88%AC%E5%8F%96/kichiku)<!--more-->

## 网页解析

```html
<div class="l-item">
    <div class="l">
        <div class="spread-module">
            <a href="/video/av2247" target="_blank">
                <div class="pic">
                    <div class="lazy-img"><img alt="【ドナルド】Dr.McDo-みさおと一緒-" src="//i2.hdslb.com/bfs/archive/364ab57dda0f61015ddb974285bb82e659fc590a.jpg@200w_125h.webp">
                    </div><i class="icon medal "></i>
                    <div class="cover-preview-module"><!---->
                        <div class="progress-bar"><span style="width: 0%;"></span>
                        </div>
                    </div>
                    <div class="mask-video"></div>
                    <div class="danmu-module"></div>
                    <span class="dur">01:27</span>
                    <!----><!---->
                    <div class="watch-later-trigger w-later"></div>
                </div>
                <p title="【ドナルド】Dr.McDo-みさおと一緒-" class="t">【ドナルド】Dr.McDo-みさおと一緒-</p><p class="num"><span class="play">
```

根据网页结构发现渲染的html中信息包含在li节点内部，于是我们使用css选择器进行提取

## 蜘蛛

```python
import scrapy
import json
from kichiku.items import KichikuItem
import selenium.webdriver
from selenium.webdriver.chrome.options import Options

class GuichuSpider(scrapy.Spider):
    name = 'guichu'
    allowed_domains = ['www.bilibili.com']
    base_url = 'https://www.bilibili.com/v/kichiku/guide/?spm_id_from=333.334.b_7072696d6172795f6d656e75.67#/all/default/0/'

    def __init__(self):
        chrome_options = Options()
        chrome_options.add_argument('--headless')  # 指定使用无头的模式
        self.driver = selenium.webdriver.Chrome(chrome_options=chrome_options)


    # def closed(self,spider):
    #     self.driver.close()

    def parse(self, response):
        datas = response.css('.vd-list li')
        for data in datas:
            item = KichikuItem()

            item['title'] = data.css('a p::text').extract_first()
            item['play'] = data.css('.play::text').extract_first()

            item['danmu'] = data.css('.danmu::text').extract_first()

            item['up'] = data.css('.up-info a::text').extract_first()

            item['time'] = data.css('.pic span::text').extract_first()

            yield item

    def start_requests(self):
        for page in range(1,self.settings.get('PAGE')+1):
            url = self.base_url+str(page)
            yield scrapy.Request(url=url,callback=self.parse,dont_filter=True)


```

​	因为selenium在模拟打开网页的时候总是每次新建一个标签页，并且不会主动关闭浏览器，我们在蜘蛛的init函数中新建一个模拟实体然后定义一个closed函数，在每次爬虫结束的时候关闭浏览器

## selenium模拟部分

放在middleware.py中

```python
class jsmiddleware(object):
    def process_request(self,request,spider):
        if request.url != 'https://s1.hdslb.com/bfs/static/jinkela/subchannel/subchannel.a5960552c0624e434750f62452fcdb8a47eb3bf4.js':
            spider.driver.get(request.url)
            html = spider.driver.page_source
            return scrapy.http.HtmlResponse(url=request.url,body=html.encode('utf-8'),encoding='utf-8',request=request)

```

因为哔哩哔哩在每次渲染的时候会有一个提前加载的哔哩哔哩图标页面，我们用if语句排除这个网址的渲染

## 反爬虫策略

```python
SQL_HOST = 'localhost'
SQL_USER = 'postgres'
SQL_PASSWORD = '123456'
SQL_DATABASE = 'spiders'
SQL_PORT = '5432'

user_agent_list = [\
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 "
        "(KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",
        "Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 "
        "(KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 "
        "(KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.6 "
        "(KHTML, like Gecko) Chrome/20.0.1090.0 Safari/536.6",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.1 "
        "(KHTML, like Gecko) Chrome/19.77.34.5 Safari/537.1",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 "
        "(KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",
        "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/536.5 "
        "(KHTML, like Gecko) Chrome/19.0.1084.36 Safari/536.5",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_0) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.0 Safari/536.3",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 "
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 "
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24"
       ]

USER_AGENT = random.choice(user_agent_list)
PAGE = 3000
```

使用random函数对每次的浏览器头进行伪装，这个还不够，最好使用代理服务器，每次抓取时自动切换服务器

## item

```python
import scrapy
from scrapy import Field

class KichikuItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = Field()
    play = Field()
    danmu = Field()
    up = Field()
    time = Field()
```

最终的结果保存到数据库中

## 遇到的问题

因为一共有差不多4000多页的数据，scrapy每次请求的次数太多，在爬取到400页的时候就出现了远程服务器拒绝连接的情况，所以降低爬取的线程数，使用代理是最好的办法