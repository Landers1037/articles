---
title: selenium自动测试1
name: selenium1
date: 2019-02-08
id: 0
tags: [selenium,python]
categories: []
abstract: ""
---


# 测试工具：

## Chrome ，selenium

### 获取浏览器的网页信息<!--more-->

```python
# 使用selenium模拟浏览器来获取网页的信息
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait

browser=webdriver.Chrome() # 模拟Chrome
try:
    browser.get('https://www.baidu.com')
    input = browser.find_element_by_id('kw')
    input.send_keys('python')
    input.send_keys(Keys.ENTER)
    wait = WebDriverWait(browser,10)
    wait.until(EC.presence_of_all_elements_located((By.ID,'coment_left')))
    print(browser.current_url)
    print(browser.get_cookie())
    print(browser.page_source)
finally:
    browser.close()
```

### 获取淘宝网站的节点信息

[![kNETwd.png](https://s2.ax1x.com/2019/02/08/kNETwd.png)](https://imgchr.com/i/kNETwd)

```python
# 尝试获取淘宝的信息
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.taobao.com')
# input_first = browser.find_element_by_id('q')
# input_second = browser.find_element_by_css_selector('#q')
# input_third = browser.find_element_by_xpath('//*[@id="q"]')
# print(input_first,input_second,input_third)

# 获取多个节点
lis1=browser.find_element_by_css_selector('.service-bd li')
lis2=browser.find_elements_by_css_selector('.service-bd li')
print(lis1) # 返回第一个值
print(lis2) # 返回列表
browser.close()

```

[![kNERW6.png](https://s2.ax1x.com/2019/02/08/kNERW6.png)](https://imgchr.com/i/kNERW6)
根据html的源代码利用css选择器获取需要的信息

### 自动化测试，在输入框中填写信息

```python
browser = webdriver.Chrome()
# browser.get('https://www.taobao.com/')
# input = browser.find_element_by_id('q')
# input.send_keys('五三模拟')
# time.sleep(2)
# input.clear()
# input.send_keys('黄冈金卷')
# button = browser.find_element_by_class_name('btn-search')
# button.click()

# javascript模拟下拉进度条
url='https://landers1037.top'
browser.get(url)
# top=browser.find_element_by_class_name()
browser.execute_script('window.scrollTo(0, document.scrollHeight)')
browser.execute_script('alert("To Bottom")')
```

### 使用自带的方法属性获取节点的信息

```python
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
input=browser.find_element_by_class_name('zu-top-add-question')
print(input.text) # 文本属性
print(input.id)
print(input.tag_name)
print(input.location)
print(input.size)
```

### 自动化测试设置等待网页响应时间

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from  selenium.webdriver.support import expected_conditions as EC

browser=webdriver.Chrome()
browser.get('http://www.taobao.com/')
wait=WebDriverWait(browser,10) # 10s的最长等待时间
input=wait.until(EC.presence_of_element_located((By.ID,'q')))
button=wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR,'.btn-search')))
print(input,button)
```

