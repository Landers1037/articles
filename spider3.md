---
title: python简易爬虫之动态爬取租房信息
name: spider3
date: 2019-02-03
id: 0
tags: [python,爬虫]
categories: []
abstract: ""
---


# 环境：Python3 

保存为json和csv文件<!--more-->

```python
# 尝试抓取中介信息
from selenium import webdriver
import json
import csv

list=[]
browser=webdriver.Chrome()
def geturl(offset):
    url='https://xy.zu.anjuke.com/fangyuan/xiangchengbc/p'+str(offset)
    browser.get(url)
    classnames=browser.find_elements_by_class_name('zu-info')
    # for classname in classnames:
    #     print(classname.text)


    tags=browser.find_elements_by_css_selector('.zu-info a')# 位置
    details=browser.find_elements_by_css_selector('.zu-info p') # 大小
    prices=browser.find_elements_by_css_selector('.zu-side p') # 价格
    for tag ,detail,price in zip(tags,details,prices):
        # print(tag.text+'\n'+detail.text+'\n'+price.text)
        data = {}
        data['位置']=tag.text
        data['大小'] = detail.text.replace('\n','') # 去掉里面的换行符号
        data['价格']=price.text.replace('元/月','')

        list.append(data)

        # with open('house.json','a',encoding='utf-8')as file:
        #     file.write(json.dumps(data,ensure_ascii=False))
def save():
    with open('list.json','w',encoding='utf-8')as file:
        file.write(json.dumps(list,indent=2,ensure_ascii=False))

    headers=['位置','大小','价格']
    with open('house.csv','w',encoding='utf-8')as file:
        file_csv=csv.DictWriter(file,headers)
        file_csv.writeheader()
        file_csv.writerows(list)
def main():
    for i in range(12):
        try:
            geturl(i)
        except Exception as e:
            print(e.args)


if __name__ == '__main__':
    main()
    save()
```

