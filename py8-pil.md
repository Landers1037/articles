---
title: PIL库的图片处理使用
name: py8-pil
date: 2019-04-11
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


## PIL库
<!--more-->


## PIL库<!--more-->

```python
from PIL import ImageFont
from PIL import Image
from PIL import ImageDraw
```

```python
imageFile = "pinyin.png" #使用的图片资源	
    try:
            word_css = "pinyin.ttf"
            word_size = 138  # 文字大小
            font = ImageFont.truetype(word_css, word_size)

            # 分割得到数组
            im1 = Image.open(imageFile)  # 打开图片
            draw = ImageDraw.Draw(im1)
            time = datetime.datetime.now().strftime('%Y-%m-%d-%H-%M-%S')
            draw.text((x, y), pstr, (0, 0, 0), font=font)  # 设置位置坐标 文字 颜色 字体
            im1.save(r'C:\\Users\Administrator\Desktop\\{}.png'.format(time))
        except:
            print('找不到字体文件')

        del draw  # 删除画笔
        im1.close()  # 关闭图片
```

```python
blank = Image.new("RGB",[1024,768],"white") #创建图片或者导入图片
drawObj = ImageDraw.Draw(blank) #新建一个图片操纵对象

# 创建一个正方形。 [x1,x2,y1,y2]或者[(x1,x2),(y1,y2)]  fill代表的为颜色
drawObj.line([100,100,100,600],fill='red')
drawObj.line([100,100,600,100],fill='red')
drawObj.line([600,100,600,600],'black')
drawObj.line([100,600,600,600],'red')

# 弧形 [x1,x2,y1,y2]  弧度 颜色
drawObj.arc([100,100,600,600],0,360,fill='black')
drawObj.arc([200,100,500,600],0,360,fill='red')

# 画半圆 [x1,x2,y1,y2]  弧度 outline弦线颜色 fill填充颜色
drawObj.chord([100,100,600,600],0,360,outline=125)
drawObj.chord([100,100,600,600],0,90,outline=158)
drawObj.chord([100,100,600,600],90,180,outline=99,fill='red')

# 扇形 [x1,x2,y1,y2]  弧度 outline弦线颜色 fill填充颜色
drawObj.pieslice([100,100,600,600],180,210,outline=255)
drawObj.pieslice([100,100,600,600],30,80,fill=255)

# 多边形
drawObj.polygon([10,23,45,6,77,87],outline='red')
drawObj.polygon([10,20,30,40,50,90,70,80,90,100],fill='red')

# 矩形
drawObj.rectangle((200,200,500,500),outline = "red")
drawObj.rectangle((250,300,450,400),fill = 128)

# 文字
text = 'i\'m very happy'
# 颜色
drawObj.ink = 0 + 0 * 256 + 255 * 256 * 256
# 加载到图片上
drawObj.text([300,500],text)
```

参考文章：[csdn](https://blog.csdn.net/dxyna/article/details/81128297)

