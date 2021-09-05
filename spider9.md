---
title: Scrapy爬取烂番茄的年度电影榜单
name: spider9
date: 2019-03-08
id: 0
tags: [python,爬虫,scrapy]
categories: []
abstract: ""
---


分析网页结构后，发现年度电影数据最早到1950年，使用一个for循环对url进行构造
源码地址 [github](https://github.com/Landers1037/PySpider/tree/master/%E7%83%82%E7%95%AA%E8%8C%84%E7%94%B5%E5%BD%B1%E7%BD%91%E7%AB%99%E5%B9%B4%E5%BA%A6%E6%9C%80%E4%BD%B3%E7%94%B5%E5%BD%B1%E6%A6%9C%E5%8D%95/rottentomatoes)
<!--more-->


分析网页结构后，发现年度电影数据最早到1950年，使用一个for循环对url进行构造
源码地址 [github](https://github.com/Landers1037/PySpider/tree/master/%E7%83%82%E7%95%AA%E8%8C%84%E7%94%B5%E5%BD%B1%E7%BD%91%E7%AB%99%E5%B9%B4%E5%BA%A6%E6%9C%80%E4%BD%B3%E7%94%B5%E5%BD%B1%E6%A6%9C%E5%8D%95/rottentomatoes)<!--more-->

## 蜘蛛部分

```python
# -*- coding: utf-8 -*-
import scrapy
from rottentomatoes.items import RottentomatoesItem

class TomatoSpider(scrapy.Spider):
    name = 'tomato'
    allowed_domains = ['www.rottentomatoes.com']
    start_urls = ['https://www.rottentomatoes.com/']

    def parse(self, response):
        datas = response.css('.table tr')
        for data in datas:
            item = RottentomatoesItem()
            item['movieyear'] = response.url.replace('https://www.rottentomatoes.com/top/bestofrt/?year=','')
            item['rank'] = data.css('td:nth-child(1)::text').extract_first()
            item['movie'] = str(data.css('td a::text').extract_first()).strip()
            item['tomatometer'] = str(data.css('.tMeterScore::text').extract_first()).replace('\xa0','')
            yield item

    def start_requests(self):
        base_url = 'https://www.rottentomatoes.com/top/bestofrt/?year='
        for page in range(1950,2019):
            url = base_url + str(page)
            yield scrapy.Request(url,callback=self.parse)
```

## pipeline部分

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

        sql = 'insert into tomato(movieyear,rank,movie,tomatometer) values (%s,%s,%s,%s)'


        try:
            self.cursor.execute(sql,(data['movieyear'],data['rank'],data['movie'],data['tomatometer']))
            self.db.commit()
        except:
            self.db.rollback()

        return item

    def close_spider(self,spider):
        self.db.close()
```

