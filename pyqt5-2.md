---
title: PyQt5图形化布局
name: pyqt5-2
date: 2019-02-02
id: 0
tags: [python,pyqt5,学习笔记]
categories: []
abstract: ""
---
# 环境：python3

## 绝对布局

![k8RZxP.webp](/images/pyqt5-2-1.webp)
<!--more-->


# 环境：python3

## 绝对布局

![k8RZxP.webp](/images/pyqt5-2-1.webp)<!--more-->

```python
#布局管理
#使用绝对定位或者提供的layout类
import sys
from PyQt5.QtWidgets import QWidget,QLabel,QApplication

class ly1(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        label1=QLabel('test1',self)
        label1.move(15,10)

        label2=QLabel('test2',self)
        label2.move(35,40)

        self.setGeometry(300,200,600,500)
        self.setWindowTitle('absolute layout')
        self.show()

if __name__=='__main__':
    app=QApplication(sys.argv)
    ly1=ly1()
    sys.exit(app.exec_())

```

## 盒式布局

![k8Rnr8.webp](/images/pyqt-5-2-2.webp)

```python
# qhboxlayout

import sys
from PyQt5.QtWidgets import (QWidget,qApp,QApplication,QPushButton,QHBoxLayout,QVBoxLayout)

class ly2(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        okbutton=QPushButton('ok')
        cancelbutton=QPushButton('cancel')

        hbox=QHBoxLayout() # 定义一个水平布局，放置按钮
        hbox.addStretch(1) # 添加按钮间的弹性空间
        hbox.addWidget(okbutton)
        hbox.addWidget(cancelbutton)

        vbox=QVBoxLayout() # 新建一个垂直布局，把水平布局放在里面
        vbox.addStretch(1) # 放在右下角
        vbox.addLayout(hbox)

        self.setLayout(vbox)

        self.setGeometry(300,200,600,500)
        self.setWindowTitle('hbox layout')
        self.show()

if __name__=='__main__':
    app=QApplication(sys.argv)
    ly2=ly2()
    sys.exit(app.exec_())
```

## 表格布局

![k8RuqS.webp](/images/pyqt5-2-3.webp)

```python
# qgridlayout
# 栅格布局

import sys
from PyQt5.QtWidgets import (QGridLayout,QWidget,QApplication,QPushButton)

class ly3(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        grid=QGridLayout()
        self.setLayout(grid)

        names=['cls','bck','','close'
               ,'7','8','9','/',
               '4','5','6','*',
               '1','2','3','-',
               '0','.','=','+']
        positions=[(i,j) for i in range(5) for j in range(4)]
        for positions,name in zip(positions,names):
            if name=='':
                continue
            button=QPushButton(name)
            grid.addWidget(button,*positions)

        self.move(200,300)
        self.setWindowTitle('grid layout')
        self.show()

if __name__=='__main__':
    app=QApplication(sys.argv)
    ly3=ly3()
    sys.exit(app.exec_())


```

