---
title: selenium自动测试3
name: selenium3
date: 2019-02-09
id: 0
tags: [python,selenium]
categories: []
abstract: ""
---


# 环境：py3.7    selenium

## 选项卡的管理

```python
import time
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(2)
browser.switch_to_window(browser.window_handles[0])
```

<!--more-->我们打开两个网页并且在两个选项卡之间切换
解释器有一句提示

```python
D:\Python3.7\python.exe D:/PyCharmProject/pylearn1/5/sele8.py
['CDwindow-43E60C0E5F0F40D681C031FD97565C7C', 'CDwindow-11F12BB1C815706EAC8AB211D6A7BA53']
D:/PyCharmProject/pylearn1/5/sele8.py:9: DeprecationWarning: use driver.switch_to.window instead
  browser.switch_to_window(browser.window_handles[1])
D:/PyCharmProject/pylearn1/5/sele8.py:12: DeprecationWarning: use driver.switch_to.window instead
  browser.switch_to_window(browser.window_handles[0])
```

可见selenium库的函数有些许变化改变标签建议使用`driver.switch_to.window`

## 异常处理

```python
# 异常处理
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException,TimeoutException

browser=webdriver.Chrome()
browser.get('https://www.baidu.com')
try:
    browser.find_element_by_id('hello')
except TimeoutException:
    print('time out')
except NoSuchElementException:
    print('no such element')
finally:
    browser.close()
```

