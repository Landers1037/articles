---
title: PyQt5图形化窗口界面学习
name: pyqt5-1
date: 2019-02-02
id: 0
tags: [pyqt5,python,学习笔记]
categories: []
abstract: ""
---


# PyQt5图形化学习

## 一个带有图标的窗口界面
<!--more-->
# PyQt5图形化学习

## 一个带有图标的窗口界面

<!--more-->


# PyQt5图形化学习

## 一个带有图标的窗口界面<!--more-->

![k8cgsS.webp](/images/pyqt5-1-1.webp)

```python
'''带有图标的面向对象编程'''
#因为之前是面向过程的函数
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QToolTip, QPushButton, QMessageBox, QDesktopWidget
from PyQt5.QtGui import QIcon
from  PyQt5.QtGui import QFont
from  PyQt5.QtCore import QCoreApplication
#qtooltip的作用是放在上面的一个提示框，这个提示框用html语言进行换行

#新建一个类继承qwidget
class demo1(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        QToolTip.setFont(QFont('Sanserif',10))
        self.setToolTip('this is a <b>QWidget</b>widget')
        btn=QPushButton('Button',self)
        btn.setToolTip('this is a <b>QPushButton</b>widget')
        btn.resize(btn.sizeHint()) #自动默认按钮大小
        btn.move(50,50)

        qbtn=QPushButton('click close',self)#后面的self就是放置的父亲组件没有的时候就是一个顶层窗口
        qbtn.clicked.connect(QCoreApplication.instance().quit)
        qbtn.move(100,100)
        qbtn.resize(qbtn.sizeHint())


        self.setWindowTitle('GUI with icon')
        # self.setGeometry(300,300,600,600)
        '''设置窗口居中'''
        self.center()
        self.setWindowIcon(QIcon('icon.webp'))
        self.show()

    def closeEvent(self, event):
        reply=QMessageBox.question(self,'message','are u sure to quit',QMessageBox.Yes|QMessageBox.No)
        if reply==QMessageBox.Yes:
            event.accept()
        else:
            event.ignore()

    def center(self):
        qr=self.frameGeometry() #获取当前屏幕的尺寸
        cp=QDesktopWidget().availableGeometry().center() #获取屏幕的可用尺寸的中心像素位置
        qr.moveCenter(cp) #把qr移动到中心位置
        self.move(qr.topLeft()) #把窗口组件移动到qr定义的左上角零点位置

if __name__=='__main__':
    app=QApplication(sys.argv)
    demo1=demo1() #实例化对象
    sys.exit(app.exec_())
```

## 状态栏窗口

![k8chIs.webp](/images/pyqt-5-1-2.webp)

```python
'''菜单和工具栏'''
import sys
from PyQt5.QtWidgets import QApplication,QMainWindow
#qmainwindow is a main window used to put others
class demo2(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.statusBar().showMessage('ready')
        self.setGeometry(300,300,600,600)
        self.setWindowTitle('demo2 statusbar')
        self.show()


if __name__=='__main__':
    app=QApplication(sys.argv)
    demo2=demo2()
    sys.exit(app.exec_())
```

## 菜单栏窗口

![k8cjo9.webp](/images/pyqt-5-1-3.webp)

```python
'''menubar'''
import sys
from PyQt5.QtWidgets import QApplication,QMainWindow,qApp,QAction
from PyQt5.QtGui import QIcon

class demo3(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        exitAct=QAction(QIcon('icon.webp'),'&Exit',self)
        openAct=QAction('&Open',self)
        exitAct.setShortcut('Ctrl+Q')
        exitAct.setStatusTip('exit application')
        exitAct.triggered.connect(qApp.quit) #所有的信号槽调用关闭方法都没有（）
        openAct.triggered.connect(qApp.quit)

        self.statusBar()

        menubar=self.menuBar() #是最上面的顶层菜单
        filemenu=menubar.addMenu('&File') #设置顶层菜单
        openmenu=menubar.addMenu('&Open')
        filemenu.addAction(exitAct) #下面的选项菜单
        filemenu.addAction(openAct)

        self.setGeometry(300,300,600,600)
        self.setWindowTitle('menu')
        self.show()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    demo3 = demo3()
    sys.exit(app.exec_())

```

## 二级菜单

![k8czJ1.webp](/images/pyqt-5-1-4.webp)

```python
#二级子菜单
import sys
from PyQt5.QtWidgets import QMainWindow,QAction,QMenu,QApplication

class demo4(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        menubar=self.menuBar()
        filemenu=menubar.addMenu('file')

        impMenu=QMenu('import',self)
        impAct=QAction('import mail',self) #动作的监听
        impMenu.addAction(impAct)

        newAct=QAction('new',self)
        filemenu.addAction(newAct)
        filemenu.addMenu(impMenu)

        self.setGeometry(300,200,600,500)
        self.setWindowTitle('demo4')
        self.show()

if __name__=='__main__':
    app=QApplication(sys.argv)
    demo4=demo4()
    sys.exit(app.exec_())

```

## 勾选式菜单栏

![k8gpz6.webp](/images/pyqt-5-1-5.webp)

```python
#勾选菜单栏
import sys
from PyQt5.QtWidgets import QMainWindow,QAction,QMenu,QApplication

class demo5(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.statusBar=self.statusBar()
        self.statusBar.showMessage('ready')
        menubar=self.menuBar()
        viewMenu=menubar.addMenu('view')

        viewStatAct=QAction('view statusbar',self,checkable=True)
        viewStatAct.setStatusTip('view statusbar')
        viewStatAct.setChecked(True)
        viewStatAct.triggered.connect(self.toggleMenu)

        viewMenu.addAction(viewStatAct)


        self.setGeometry(300,200,600,500)
        self.setWindowTitle('demo5')
        self.show()

    def toggleMenu(self):
        if state:
            self.statusBar.show()
        else:
            self.statusBar.hide()

if __name__=='__main__':
    app=QApplication(sys.argv)
    demo5=demo5()
    sys.exit(app.exec_())

```

## 右键菜单

![k8gAdH.webp](/images/pyqt-5-1-6.webp)

```python
#右键菜单
import sys
from PyQt5.QtWidgets import QMainWindow,QAction,QMenu,qApp,QApplication

class demo6(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setGeometry(300,200,600,500)
        self.setWindowTitle('demo4')
        self.show()


    def contextMenuEvent(self, QContextMenuEvent):
        cmenu=QMenu(self)
        newAct=cmenu.addAction('new')
        opnAct=cmenu.addAction('open')
        quitAct=cmenu.addAction('quit')
        action=cmenu.exec_(self.mapToGlobal(QContextMenuEvent.pos()))

        if action==quitAct:
            qApp.quit()

if __name__=='__main__':
    app=QApplication(sys.argv)
    demo6=demo6()
    sys.exit(app.exec_())

```

## 图标工具栏

![k8gZFA.webp](/images/pyqt-5-1-7.webp)

```python
#工具栏
import sys
from PyQt5.QtWidgets import QMainWindow,QAction,QMenu,QApplication,qApp
from PyQt5.QtGui import QIcon


class demo7(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        exitAct=QAction(QIcon('icon.webp'),'exit',self)
        exitAct.triggered.connect(qApp.quit)

        self.toolbar=self.addToolBar('exit')
        self.toolbar.addAction(exitAct)

        self.setGeometry(300,200,600,500)
        self.setWindowTitle('demo4')
        self.show()

if __name__=='__main__':
    app=QApplication(sys.argv)
    demo7=demo7()
    sys.exit(app.exec_())

```

## 主窗口

[![k8g3wQ.md.webp](/images/pyqt-5-1-8.webp)](https://imgchr.com/i/k8g3wQ)

```python
#主窗口的设计
from PyQt5.QtWidgets import QMainWindow,QTextEdit,QAction,QApplication
from PyQt5.QtGui import QIcon
import sys

class demo8(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()


    def initUI(self):
        textEdit=QTextEdit()
        self.setCentralWidget(textEdit)

        exitAct=QAction(QIcon('icon.webp' ),'EXIT',self)
        exitAct.setShortcut('Ctrl+q')
        exitAct.setStatusTip('exit')
        exitAct.triggered.connect(self.close)

        self.statusBar()

        menubar=self.menuBar()
        filemenu=menubar.addMenu('&File')
        filemenu.addAction(exitAct)

        toolbar=self.addToolBar('Exit')
        toolbar.addAction(exitAct)

        self.setGeometry(300,200,800,600)
        self.setWindowTitle('main window')
        self.show()
#这个文本编辑框是自带右键菜单的
if __name__=='__main__':
    app=QApplication(sys.argv)
    demo8=demo8()
    sys.exit(app.exec_())
```

