---
title: Scrapy统计哔哩哔哩番剧榜单
name: spider7
date: 2019-03-05
id: 0
tags: [python,爬虫]
categories: []
abstract: ""
---


# 哔哩哔哩番剧榜单

源代码地址 [github](https://github.com/Landers1037/PySpider/tree/master/%E5%93%94%E5%93%A9%E5%93%94%E5%93%A9%E7%95%AA%E5%89%A7%E6%A6%9C%E5%8D%95/bilibili)<!--more-->

## 分析网页源码

根据网页链接提取，发现网页使用ajax异步加载数据，找到了对应的接口后发现里面的数据是json，所以使用json模块进行分析

## 蜘蛛

```python
# -*- coding: utf-8 -*-
import scrapy
import json
from bilibili.items import fanItem

class BilibotSpider(scrapy.Spider):
    name = 'bilibot'
    allowed_domains = ['www.bilibili.com']
    start_urls = ['https://bangumi.bilibili.com/media/web_api/search/result?season_version=-1&area=-1&is_finish=-1&copyright=-1&season_status=-1&season_month=-1&pub_date=-1&style_id=-1&order=3&st=1&sort=0&page=1&season_type=1&pagesize=20']

    # def parse(self, response):
    #     data = response.text
    #     print(data)

    def parse(self,response):
        jsonBody = json.loads(response.body)
        result = jsonBody['result']
        data = result['data']
        for dict in data:
            modelItem = fanItem()
            modelItem['title'] = dict['title']
            modelItem['follow'] = dict['order']['follow']
            modelItem['play'] = dict['order']['play']
            try:
                modelItem['score'] = dict['order']['score']
            except Exception :
                modelItem['score'] = '没有数据'
            modelItem['cover'] = dict['cover']
            yield modelItem
        #     modelItems.append(modelItem)
        # return modelItems

    def start_requests(self):
        base_url = 'https://bangumi.bilibili.com/media/web_api/search/result?season_version=-1&area=-1&is_finish=-1&copyright=-1&season_status=-1&season_month=-1&pub_date=-1&style_id=-1&order=3&st=1&sort=0&page='
        for i in range(1,self.settings.get('MAX_PAGE')+1):
            url = base_url + str(i) + '&season_type=1&pagesize=20'
            yield scrapy.Request(url,callback=self.parse)

```

## item

```python
import scrapy
from scrapy import Field

class BilibiliItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

class fanItem(scrapy.Item):
    title = Field()  # 名字
    follow = Field() # 追番
    play = Field()   # 播放量
    score = Field()  # 评分
    cover = Field()  # 封面
```

## 数据库连接

```python
class pgPipeline():
    def __init__(self,host,database,user,password,port):
        self.host = host
        self.database = database
        self.user = user
        self.password = password
        self.port = port

        self.db = psycopg2.connect(host=self.host, user=self.user, password=self.password, database=self.database, port=self.port)
        self.cursor = self.db.cursor()

    @classmethod
    def from_crawler(cls,crawler):
        return cls(
            host=crawler.settings.get('SQL_HOST'),
            database=crawler.settings.get('SQL_DATABASE'),
            user=crawler.settings.get('SQL_USER'),
            password=crawler.settings.get('SQL_PASSWORD'),
            port=crawler.settings.get('SQL_PORT'),
        )

    def open_spider(self,spider):
        self.db = psycopg2.connect(host=self.host, user=self.user, password=self.password, database=self.database, port=self.port)
        self.cursor = self.db.cursor()

    def process_item(self,item,spider):
        data = item

        sql = 'insert into fanju (番剧,追番人数, 播放量,评分,封面) values (%s,%s,%s,%s,%s)'

        try:
            self.cursor.execute(sql,(data['title'],data['follow'],data['play'],data['score'],data['cover']))
            self.db.commit()
        except:
            self.db.rollback()

        return item

    def close_spider(self,spider):
        self.db.close()
```

问题分析：部分番剧没有评分或者观看记录，在使用json提取数据的时候使用try语句进行提取返回错误时直接对字典的键赋值为“null”

[![kjYqdH.png](https://s2.ax1x.com/2019/03/05/kjYqdH.png)](https://imgchr.com/i/kjYqdH)