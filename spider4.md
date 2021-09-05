---
title: Scrapy实现京东商城爬虫
name: spider4
date: 2019-03-02
id: 0
tags: [scrapy,爬虫]
categories: []
abstract: ""
---


# Python3.7 

github源码地址：[github](https://github.com/Landers1037/PySpider/tree/master/%E4%BA%AC%E4%B8%9C%E5%95%86%E5%9F%8E%E5%95%86%E5%93%81%E7%88%AC%E5%8F%96/jd)
<!--more-->


# Python3.7 

github源码地址：[github](https://github.com/Landers1037/PySpider/tree/master/%E4%BA%AC%E4%B8%9C%E5%95%86%E5%9F%8E%E5%95%86%E5%93%81%E7%88%AC%E5%8F%96/jd)<!--more-->

## 蜘蛛部分

```python
# -*- coding: utf-8 -*-
'''
对京东的商品爬取尝试对一个关键字列表进行遍历，获取其对应的全部商品信息
'''
import scrapy
from jd.items import shopItem
from scrapy import Request

class JdspiderSpider(scrapy.Spider):
    name = 'jdspider'
    allowed_domains = ['jd.com']
    start_urls = ['https://search.jd.com/Search?keyword=iphone&enc=utf-8&wq=iphone&page=']

    def parse(self, response):
        with open('1.html','w',encoding='utf-8')as f:
            f.write(response.text)
        goods = response.css('.gl-item')
        for shop in goods:
            item = shopItem()
            item['title'] = shop.css('.p-name-type-2 a em::text').extract_first()
            item['img'] = shop.css('.p-img img::attr(source-data-lazy-img)').extract_first()
            item['colors'] = shop.css('.ps-item a::attr(title)').extract_first()
            item['price'] = shop.css('.p-price i::text').extract_first()
            yield item

    def start_requests(self):
        base_url = 'https://search.jd.com/Search?keyword=iphone&enc=utf-8&wq=iphone&page='
        for page in range(1,self.settings.get('MAX_PAGE')+1):
            url = base_url+str(page)
            yield Request(url,self.parse) # 递归

```

## item部分

```python
import scrapy
from scrapy import Item,Field

class JdItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

class shopItem(Item):
    title = Field()
    img = Field()
    colors = Field()
    price = Field()
```

## pipelines部分

```python
import pymysql
from scrapy.exceptions import DropItem
from scrapy import Request

class JdPipeline(object):
    def process_item(self, item, spider):
        return item

class MysqlPipeline():
    def __init__(self,host,database,user,password,port):
        self.host = host
        self.database = database
        self.user = user
        self.password = password
        self.port = port

        self.db = pymysql.connect(self.host, self.user, self.password, self.database, port=self.port)
        self.cursor = self.db.cursor()

    @classmethod
    def from_crawler(cls,crawler):
        return cls(
            host=crawler.settings.get('MYSQL_HOST'),
            database=crawler.settings.get('MYSQL_DATABASE'),
            user=crawler.settings.get('MYSQL_USER'),
            password=crawler.settings.get('MYSQL_PASSWORD'),
            port=crawler.settings.get('MYSQL_PORT'),
        )

    def open_spider(self,spider):
        self.db = pymysql.connect(self.host, self.user, self.password, self.database, port=self.port)
        self.cursor = self.db.cursor()


    def process_item(self,item,spider):
        data = item
        sql = 'insert ignore into jd (标题,颜色,图片,价格) values (%s,%s,%s,%s)'
        self.cursor.execute(sql,(data['title'],data['colors'],data['img'],data['price']))
        self.db.commit()
        return item

    def close_spider(self,spider):
        self.db.close()
```

## settings部分

```python
BOT_NAME = 'jd'

SPIDER_MODULES = ['jd.spiders']
NEWSPIDER_MODULE = 'jd.spiders'
ROBOTSTXT_OBEY = False

ITEM_PIPELINES = {
   'jd.pipelines.JdPipeline': 300,
    'jd.pipelines.MysqlPipeline': 301,
}
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119\
        Safari/537.36'

MYSQL_HOST = 'localhost'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '123456'
MYSQL_PORT = 3306
MYSQL_DATABASE = 'spiders'

MAX_PAGE = 100 # 限制抓取页数
```

