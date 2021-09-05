---
title: Scrapy爬取考研成绩分数线
name: spider6
date: 2019-03-03
id: 0
tags: [爬虫,scrapy]
categories: []
abstract: ""
---


github源代码：[github](https://github.com/Landers1037/PySpider/tree/master/%E5%A4%A7%E5%AD%A6%E8%80%83%E7%A0%94%E6%88%90%E7%BB%A9%E5%88%86%E6%95%B0%E7%BA%BF)

## 蜘蛛部分
<!--more-->


github源代码：[github](https://github.com/Landers1037/PySpider/tree/master/%E5%A4%A7%E5%AD%A6%E8%80%83%E7%A0%94%E6%88%90%E7%BB%A9%E5%88%86%E6%95%B0%E7%BA%BF)

## 蜘蛛部分<!--more-->

```python
import scrapy
from kaoyan.items import KaoyanItem

class KaoyanlineSpider(scrapy.Spider):
    name = 'kaoyanline'
    allowed_domains = ['www.kaoshidian.com']
    start_urls = ['http://www.kaoshidian.com/kaoyan/fs-13-0-0-0-0.html']

    def parse(self, response):
        html = response.css('tr')
        for data in html:
            item = KaoyanItem()
            item['years'] = data.css('td[class="tc"]::text').extract_first()
            item['school'] = data.css('td[class="tc"]:nth-child(2) a::text').extract_first()
            item['cate'] = data.css('td[class="tc"]:nth-child(3)::text').extract_first()
            item['profession'] =data.css('td[class="tc"]:nth-child(5) a::text').extract_first()
            item['score'] = data.css('td[class="tc"]:nth-child(6)::text').extract_first()
            yield item

    def start_requests(self):
        base_url = 'http://www.kaoshidian.com/kaoyan/fs-13-0-0-0-'
        for page in range(1,self.settings.get('MAX_PAGE')+1):
            url = base_url + str(page) + '.html'
            yield scrapy.Request(url,self.parse)
```

## item部分

```python
import scrapy
from scrapy import Field

class KaoyanItem(scrapy.Item):
    years = Field()
    school = Field()
    cate = Field()
    profession = Field()
    score = Field()
```

## pipeline部分

```python
import psycopg2

class KaoyanPipeline(object):
    def process_item(self, item, spider):
        return item

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
        # 先新建一个表格存放大学排名
        self.cursor = self.db.cursor()

    def process_item(self,item,spider):
        data = item

        sql = 'insert into score (years,school,cate,profession,score) values (%s,%s,%s,%s,%s)'

        try:
            self.cursor.execute(sql,(data['years'],data['school'],data['cate'],data['profession'],data['score']))
            self.db.commit()
        except:
            self.db.rollback()

        return item

    def close_spider(self,spider):
        self.db.close()
```

## settings部分

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119\
        Safari/537.36'
MAX_PAGE = 344

SQL_HOST = 'localhost'
SQL_USER = 'postgres'
SQL_PASSWORD = '123456'
SQL_DATABASE = 'spiders'
SQL_PORT = '5432'
```



[![kO9Krd.png](https://s2.ax1x.com/2019/03/03/kO9Krd.png)](https://imgchr.com/i/kO9Krd)