---
title: Scrapy爬取世界500强大学排名
name: spider5
date: 2019-03-02
id: 0
tags: [python,爬虫]
categories: []
abstract: ""
---


# 环境：python3.7

github源代码地址：[github](https://github.com/Landers1037/PySpider/tree/master/%E4%B8%96%E7%95%8C500%E5%BC%BA%E5%A4%A7%E5%AD%A6%E6%8E%92%E5%90%8D/rank)
<!--more-->


# 环境：python3.7

github源代码地址：[github](https://github.com/Landers1037/PySpider/tree/master/%E4%B8%96%E7%95%8C500%E5%BC%BA%E5%A4%A7%E5%AD%A6%E6%8E%92%E5%90%8D/rank)<!--more-->

## 蜘蛛部分

```python
# -*- coding: utf-8 -*-
import scrapy
from rank.items import RankItem

class UnirankSpider(scrapy.Spider):
    name = 'unirank'
    allowed_domains = ['http://www.shanghairanking.com']
    start_urls = ['http://www.shanghairanking.com/ARWU2018.html']

    def parse(self, response):
        # with open('1.html','w',encoding='utf-8')as f:
        #     f.write(response.text)
        campus = response.css('tr')
        # campus1 = response.css('.bgf5')

        for data in campus:
            item = RankItem()
            item['years'] = str(response.url).replace('http://www.shanghairanking.com/ARWU','').replace('.html','')
            item['ranks'] = data.css('td::text').extract_first()
            item['uni'] = data.css('.left a::text').extract_first()
            item['location'] = data.css('td a::attr(title)').extract_first()
            item['score']= data.css('td:nth-child(9) div::text').extract_first()
            yield item

    def start_requests(self):
        base_url = 'http://www.shanghairanking.com/ARWU'
        for i in range(1,self.settings.get('YEAR')+1):
            parse = 2019-i
            url = base_url+str(parse)+'.html'
            yield scrapy.Request(url,self.parse)
```

## item部分

```python
import scrapy
from scrapy import Field

class RankItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    years = Field()
    ranks = Field()
    uni = Field()
    location = Field()
    score = Field()
```

## pipeline部分

```python
import pymysql

class RankPipeline(object):
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
        # 先新建一个表格存放大学排名
        self.cursor = self.db.cursor()
        # sql = 'create table if not exists unirank(years varchar(255) not null,ranks varchar(255) not null,uni \
        # varchar(255) not null,location varchar(255) not null,score varchar(255) not null,primary key(years,uni))'
        # self.cursor.execute(sql)

    def process_item(self,item,spider):
        data = item

        sql = 'insert into unirank (years,ranks,uni,location,score) values (%s,%s,%s,%s,%s)'

        self.cursor.execute(sql,(data['years'],data['ranks'],data['uni'],data['location'],data['score']))
        self.db.commit()

        return item

    def close_spider(self,spider):
        self.db.close()
```

## settings

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119\
        Safari/537.36'

MYSQL_HOST = 'localhost'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '123456'
MYSQL_PORT = 3306
MYSQL_DATABASE = 'spiders'

YEAR = 10 # 爬取10年的数据
```

## 爬取结果

## [![kqttYV.png](https://s2.ax1x.com/2019/03/02/kqttYV.png)](https://imgchr.com/i/kqttYV)

