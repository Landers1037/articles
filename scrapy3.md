---
title: scrapy的打包技巧
name: scrapy3
date: 2019-03-18
id: 0
tags: [python,scrapy]
categories: []
abstract: ""
---


传统的爬虫脚本是直接运行于服务器上，如果写了一个小的爬虫想要随时使用，可以使用scrapy自带的打包方法
在scrapy爬虫的根目录下（例如爬虫项目名为test）test文件夹下，新建run.py文件
<!--more-->


传统的爬虫脚本是直接运行于服务器上，如果写了一个小的爬虫想要随时使用，可以使用scrapy自带的打包方法
在scrapy爬虫的根目录下（例如爬虫项目名为test）test文件夹下，新建run.py文件<!--more-->

```python
from scrapy.utils.project import get_project_settings
from scrapy.crawler import CrawlerProcess
def run():

    project_settings = get_project_settings()
    process = CrawlerProcess(project_settings)
    process.crawl(testSpider)# 自己定义的爬虫
    process.start()

if __name__=='__main__':
    run()
```

scrapy使用以上的process类启动爬虫
导入spider文件夹下自己定义的蜘蛛.py文件中的爬虫类

```python
from test.spiders.bot import testSpider
```

**在最后的运行时，可能出现缺少*****modle test.spider.bot 不存在的错误，注意自己根目录的位置**

导入所需要的scrapy包

```python
import scrapy.pipelines.images # 如果有图片的下载用到images类就要单独导入
import urllib.robotparser
import scrapy.spiderloader
import scrapy.statscollectors
import scrapy.logformatter
import scrapy.dupefilters
import scrapy.squeues
import scrapy.extensions.spiderstate
import scrapy.extensions.corestats
import scrapy.extensions.telnet
import scrapy.extensions.logstats
import scrapy.extensions.memusage
import scrapy.extensions.memdebug
import scrapy.extensions.feedexport
import scrapy.extensions.closespider
import scrapy.extensions.debug
import scrapy.extensions.httpcache
import scrapy.extensions.statsmailer
import scrapy.extensions.throttle
import scrapy.core.scheduler
import scrapy.core.engine
import scrapy.core.scraper
import scrapy.core.spidermw
import scrapy.core.downloader
import scrapy.downloadermiddlewares.stats
import scrapy.downloadermiddlewares.httpcache
import scrapy.downloadermiddlewares.cookies
import scrapy.downloadermiddlewares.useragent
import scrapy.downloadermiddlewares.httpproxy
import scrapy.downloadermiddlewares.ajaxcrawl
import scrapy.downloadermiddlewares.chunked
import scrapy.downloadermiddlewares.decompression
import scrapy.downloadermiddlewares.defaultheaders
import scrapy.downloadermiddlewares.downloadtimeout
import scrapy.downloadermiddlewares.httpauth
import scrapy.downloadermiddlewares.httpcompression
import scrapy.downloadermiddlewares.redirect
import scrapy.downloadermiddlewares.retry
import scrapy.downloadermiddlewares.robotstxt
import scrapy.spidermiddlewares.depth
import scrapy.spidermiddlewares.httperror
import scrapy.spidermiddlewares.offsite
import scrapy.spidermiddlewares.referer
import scrapy.spidermiddlewares.urllength
import scrapy.pipelines
import scrapy.core.downloader.handlers.http
import scrapy.core.downloader.contextfactory

from test.pipelines import* # 自己定义的项目文件
from test.settings import *
# 缺什么import什么
from test import *
```

使用pyinstaller 打包，在cmd cd到项目的根目录，记住scrapy的项目目录是两层的同名文件夹，进入里层的test文件夹

**运行 `pyinstaller -F run.py`直接生成单文件，方便拷贝使用**

最后会在项目目录下的dist文件夹下找到exe文件，此时运行一定会报错
一般问题是缺少scrapy版本信息

进入python的库里找到scrapy库，拷贝`mime.types,VERSION`文件，在exe所在的目录下新建`scrapy`文件夹，把版本文件放进去，再次运行即可