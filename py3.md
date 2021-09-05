---
title: Python图片资源处理
name: py3
date: 2019-02-17
id: 0
tags: [python]
categories: []
abstract: ""
---


# python图片资源的处理

## 转化为png.py文件加载
<!--more-->


# python图片资源的处理

## 转化为png.py文件加载<!--more-->

```python
import base64
 
def pic2py(picture_name):
    """
    将图像文件转换为py文件
    :param picture_name:
    :return:
    """
    open_pic = open("%s" % picture_name, 'rb')
    b64str = base64.b64encode(open_pic.read())
    open_pic.close()
    # 注意这边b64str一定要加上.decode()
    write_data = 'img = "%s"' % b64str.decode()
    f = open('%s.py' % picture_name.replace('.', '_'), 'w+')
    f.write(write_data)
    f.close()
 
if __name__ == '__main__':
    pics = ["one.png", "two.png"]
    for i in pics:
        pic2py(i)
```

## 在文件中引用

```python
from png import img as one
...
tmp = open('one.png', 'wb')
tmp.write(base64.b64decode(one))
tmp.close()
os.remove(‘one.png’)
```

