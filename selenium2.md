---
title: selenium自动测试2
name: selenium2
date: 2019-02-09
id: 0
tags: [selenium,python]
categories: []
abstract: ""
---


# 模拟浏览器的前进后退

```python
import time
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.taobao.com')
browser.get('https://www.baidu.com')
browser.get('https://www.zhihu.com')

browser.back()
time.sleep(1)
browser.forward()
```

<!--more-->

# 模仿获取修改cookie

```python
# cookies

import time
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name':'name','domain':'www.zhihu.com','value':'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```

结果

```python
D:\Python3.7\python.exe D:/PyCharmProject/pylearn1/5/sele7.py
[{'domain': '.zhihu.com', 'httpOnly': False, 'name': 'l_n_c', 'path': '/', 'secure': False, 'value': '1'}, {'domain': 'www.zhihu.com', 'expiry': 1549705680.884314, 'httpOnly': False, 'name': 'tgw_l7_route', 'path': '/', 'secure': False, 'value': 'f2979fdd289e2265b2f12e4f4a478330'}, {'domain': '.zhihu.com', 'expiry': 1549706584, 'httpOnly': False, 'name': '__utmb', 'path': '/', 'secure': False, 'value': '51854390.0.10.1549704784'}, {'domain': '.zhihu.com', 'expiry': 1644312780.88437, 'httpOnly': False, 'name': 'q_c1', 'path': '/', 'secure': False, 'value': 'b81c79e2c604431d856561e9bd072f4a|1549704811000|1549704811000'}, {'domain': 'www.zhihu.com', 'httpOnly': False, 'name': '_xsrf', 'path': '/', 'secure': False, 'value': '6524dc593f67c2fc75a0daa912d3ad69'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884404, 'httpOnly': False, 'name': 'r_cap_id', 'path': '/', 'secure': False, 'value': '"ZGI4MTM3YmVkN2NjNGQ2NmI1Y2Q0ODQ0MjNmZGMxYTQ=|1549704811|288662099f8ab872df144b6afac66eab0fa6f5bd"'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884425, 'httpOnly': False, 'name': 'cap_id', 'path': '/', 'secure': False, 'value': '"MGJjNzFiMTE5ZGVhNGVlMDhjNTM1YTU0N2NiNzM4OTA=|1549704811|bbd5b33faa33b55f1eaf911585a94767c974d511"'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884445, 'httpOnly': False, 'name': 'l_cap_id', 'path': '/', 'secure': False, 'value': '"NTg1YTRiY2NhOWYyNDIwMmE5MGE4YTkyMTNjMzI1MjI=|1549704811|9a78db62999d39161a5a9150d69fcabbfe2b02c1"'}, {'domain': '.zhihu.com', 'httpOnly': False, 'name': 'n_c', 'path': '/', 'secure': False, 'value': '1'}, {'domain': '.zhihu.com', 'expiry': 1644312783.696351, 'httpOnly': False, 'name': 'd_c0', 'path': '/', 'secure': False, 'value': '"AIAjkGJ69A6PTtudL-q7P8p60dqmgl53jjc=|1549704813"'}, {'domain': '.zhihu.com', 'expiry': 1612776783, 'httpOnly': False, 'name': '_zap', 'path': '/', 'secure': False, 'value': '227b4bf9-0c06-41bf-a50d-37c90f02fb40'}, {'domain': '.zhihu.com', 'expiry': 1612776784, 'httpOnly': False, 'name': '__utma', 'path': '/', 'secure': False, 'value': '51854390.808437234.1549704784.1549704784.1549704784.1'}, {'domain': '.zhihu.com', 'httpOnly': False, 'name': '__utmc', 'path': '/', 'secure': False, 'value': '51854390'}, {'domain': '.zhihu.com', 'expiry': 1565472784, 'httpOnly': False, 'name': '__utmz', 'path': '/', 'secure': False, 'value': '51854390.1549704784.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none)'}, {'domain': '.zhihu.com', 'expiry': 1612776784, 'httpOnly': False, 'name': '__utmv', 'path': '/', 'secure': False, 'value': '51854390.000--|3=entry_date=20190209=1'}]
[{'domain': '.zhihu.com', 'httpOnly': False, 'name': 'l_n_c', 'path': '/', 'secure': False, 'value': '1'}, {'domain': 'www.zhihu.com', 'expiry': 1549705680.884314, 'httpOnly': False, 'name': 'tgw_l7_route', 'path': '/', 'secure': False, 'value': 'f2979fdd289e2265b2f12e4f4a478330'}, {'domain': '.zhihu.com', 'expiry': 1549706584, 'httpOnly': False, 'name': '__utmb', 'path': '/', 'secure': False, 'value': '51854390.0.10.1549704784'}, {'domain': '.zhihu.com', 'expiry': 1644312780.88437, 'httpOnly': False, 'name': 'q_c1', 'path': '/', 'secure': False, 'value': 'b81c79e2c604431d856561e9bd072f4a|1549704811000|1549704811000'}, {'domain': 'www.zhihu.com', 'httpOnly': False, 'name': '_xsrf', 'path': '/', 'secure': False, 'value': '6524dc593f67c2fc75a0daa912d3ad69'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884404, 'httpOnly': False, 'name': 'r_cap_id', 'path': '/', 'secure': False, 'value': '"ZGI4MTM3YmVkN2NjNGQ2NmI1Y2Q0ODQ0MjNmZGMxYTQ=|1549704811|288662099f8ab872df144b6afac66eab0fa6f5bd"'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884425, 'httpOnly': False, 'name': 'cap_id', 'path': '/', 'secure': False, 'value': '"MGJjNzFiMTE5ZGVhNGVlMDhjNTM1YTU0N2NiNzM4OTA=|1549704811|bbd5b33faa33b55f1eaf911585a94767c974d511"'}, {'domain': '.zhihu.com', 'expiry': 1552296780.884445, 'httpOnly': False, 'name': 'l_cap_id', 'path': '/', 'secure': False, 'value': '"NTg1YTRiY2NhOWYyNDIwMmE5MGE4YTkyMTNjMzI1MjI=|1549704811|9a78db62999d39161a5a9150d69fcabbfe2b02c1"'}, {'domain': '.zhihu.com', 'httpOnly': False, 'name': 'n_c', 'path': '/', 'secure': False, 'value': '1'}, {'domain': '.zhihu.com', 'expiry': 1644312783.696351, 'httpOnly': False, 'name': 'd_c0', 'path': '/', 'secure': False, 'value': '"AIAjkGJ69A6PTtudL-q7P8p60dqmgl53jjc=|1549704813"'}, {'domain': '.zhihu.com', 'expiry': 1612776783, 'httpOnly': False, 'name': '_zap', 'path': '/', 'secure': False, 'value': '227b4bf9-0c06-41bf-a50d-37c90f02fb40'}, {'domain': '.zhihu.com', 'expiry': 1612776784, 'httpOnly': False, 'name': '__utma', 'path': '/', 'secure': False, 'value': '51854390.808437234.1549704784.1549704784.1549704784.1'}, {'domain': '.zhihu.com', 'httpOnly': False, 'name': '__utmc', 'path': '/', 'secure': False, 'value': '51854390'}, {'domain': '.zhihu.com', 'expiry': 1565472784, 'httpOnly': False, 'name': '__utmz', 'path': '/', 'secure': False, 'value': '51854390.1549704784.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none)'}, {'domain': '.zhihu.com', 'expiry': 1612776784, 'httpOnly': False, 'name': '__utmv', 'path': '/', 'secure': False, 'value': '51854390.000--|3=entry_date=20190209=1'}, {'domain': 'www.zhihu.com', 'expiry': 2180424784, 'httpOnly': False, 'name': 'name', 'path': '/', 'secure': True, 'value': 'germey'}]
[]
```

