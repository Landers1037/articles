---
title: Pyqt5中的QDialog使用
name: pyqt5-3
date: 2019-04-15
id: 0
tags: [pyqt5]
categories: []
abstract: ""
---


# Pyqt5环境

## QDialog

QDialog类的子类主要有QMessageBox，QFileDialog，QColorDialog，QFontDialog，QInputDialog等

qt库中自带的对话框为了提高人机交互的效率<!--more-->

## QMessagebox

```python
QMessageBox.information 信息框
QMessageBox.question 问答框
QMessageBox.warning 警告
QMessageBox.ctitical危险
QMessageBox.about 关于

reply = QMessageBox.question(self, 'Message',"Are you sure to quit?",QMessageBox.Yes,QMessageBox.No)
		if reply == QMessageBox.Yes:
			event.accept()
		else:
			event.ignore()
            
QMessageBox.about(self, "About Search System",
						  "The Search System demonstrates how to search images")
QMessageBox.warning(self, "warning", "The save directory is empty",
										QMessageBox.Cancel)
QMessageBox.information(self,"Tips","Save the current frame successfully!",
									QMessageBox.Yes)
QMessageBox.aboutQt(self,"About Qt")  
QMessageBox.critical(self,"Critical",self.tr("错误!"))  
```

## QFileDialog

```python
#打开文件
fileName_choose, filetype = QFileDialog.getOpenFileName(self,  
                                    "选取文件",  
                                    self.cwd, # 起始路径 
                                    "All Files (*);;Text Files (*.txt)")   
#设置文件扩展名过滤,用双分号间隔
#打开多文件
files, filetype = QFileDialog.getOpenFileNames(self,  
                                    "多文件选择",  
                                    self.cwd, # 起始路径 
                                    "All Files (*);;PDF Files (*.pdf);;Text Files (*.txt)")  
#保存文件
fileName_choose, filetype = QFileDialog.getSaveFileName(self,  
                                    "文件保存",  
                                    self.cwd, # 起始路径 
                                    "All Files (*);;Text Files (*.txt)")  
#保存文件时，fileName_choose返回一个文件的地址+名称
#使用with开辟一块空间保存
with open(fileName_choose,'w')as f:
    f.write(data)
#注意文件的编码问题，二进制文件只有wb
```

参考文章：

[1](https://blog.csdn.net/humanking7/article/details/80546728)

[2](https://blog.csdn.net/lingpy/article/details/80118597)

