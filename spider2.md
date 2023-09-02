---
title: python简单爬虫之抓取头条图片
name: spider2
date: 2019-02-02
id: 0
tags: [python,爬虫]
categories: []
abstract: ""
---


# 环境：python3 

查看网页源代码得知，头条的网页使用ajax刷新获取网页资源，不能使用简单的html分析，定位网页的ajax请求,分析得到图片的cdn地址

保存时自动获取图片的md5值作为图片的名称避免命名重复，使用多线程池抓取图片提高效率<!--more-->

```python
#爬取今日头条的图片
#爬取data的图片的原地址数据

import requests
from urllib.parse import urlencode
import os
from hashlib import md5
from multiprocessing.pool import Pool

def get_page(offset):
    params={
        'offset':offset,
        'format':'json',
        'keyword':'街拍',
        'autoload':'true',
        'count':'20',
        'cur_tab':'3',
    }

    		      url='https://www.toutiao.com/api/search/content/?'+urlencode(params)
    try:
        response=requests.get(url)
        if response.status_code==200:
            return response.json() #encoding to json
    except requests.ConnectionError:
        return None

def get_images(json): #迭代器返回一个生成器
    if json.get('data'):
        for item in json.get('data'): #先找到页面的分类0-20
            title=item.get('title')
            images=item.get('image_list')
            for image in images:
                 yield {
                    'image_url':image.get('url'),
                    'title':title
                 }

def save_images(item):
    if not os.path.exists(item.get('title')):
        os.mkdir(item.get('title'))
    try:
        response=requests.get(item.get('image_url'))
        if response.status_code==200:
            file_path='{0}/{1}.{2}'.format(item.get('title'),md5(response.content).hexdigest(),'jpg')
            if not os.path.exists(file_path):
                with open(file_path,'wb')as f:
                    f.write(response.content)
            else:
                print('alreadly download',file_path)
    except requests.ConnectionError:
        print('Failed to save')

def main(offset):
    json=get_page(offset)
    for item in get_images(json):
        print(item)
        save_images(item)

GROUP_START = 1
GROUP_END = 20

if __name__=='__main__':
    pool=Pool()
    groups=([x*20 for x in range(GROUP_START,GROUP_END+1)])
    pool.map(main,groups)
    pool.close()
    pool.join()
```

