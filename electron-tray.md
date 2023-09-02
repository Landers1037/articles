---
title: electron设计tray托盘
name: electron-tray
date: 2020-04-05
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron自定义托盘图标和托盘功能

<!--more-->

**托盘的建立在主进程进行**

### Tray

```javascript
const { app, Menu, Tray } = require('electron')

let tray = null
app.on('ready', () => {
  tray = new Tray('/path/to/my/icon')
  const contextMenu = Menu.buildFromTemplate([
    { label: 'Item1', type: 'radio' },
    { label: 'Item2', type: 'radio' },
    { label: 'Item3', type: 'radio', checked: true },
    { label: 'Item4', type: 'radio' }
  ])
  tray.setToolTip('This is my application.')
  tray.setContextMenu(contextMenu)
})
```

> 注意：
>
> 因为是主进程里初始化，所以tray对象不能是一个局部变量，否则托盘在建立后不久就会被系统回收导致托盘消失

tray有自带的监听事件，如`click`，`double-click`。但是click监听的函数，最好写在上下文菜单里而不是写在菜单模板里。否则可能会无法正常执行函数

```javascript
trayContextMenu.append(new MenuItem({
      label:"测试",
      click(){
          func();
      }
  }));
//如果下面这种写法肯可能无法执行
const contextMenu = Menu.buildFromTemplate([
    { label: 'Item1', type: 'radio' },
    { label: 'Item2', type: 'radio' },
    { label: 'Item3', type: 'radio', checked: true },
    { label: 'Item4', type: 'radio' }，
    {label: '测试',click: ()=>{func();},
  ])
```

#### 托盘图标

推荐使用`ico`图标格式保证图标的显示，当图片尺寸过大时可能会显示失败。

#### 内置方法

`tray.destroy()` 销毁托盘实例

`tray.setImage(image)` 设置托盘的图标

`tray.setPressedImage(image)` 设置托盘按下时的图标

`tray.setToolTip(toolTip)` 设置鼠标悬停在托盘时的提示文字

`tray.setTitle(title)` 设置托盘图标旁的文字标题

`tray.popUpContextMenu([menu, position\])` 弹出菜单

`tray.setContextMenu(menu)` 设置托盘菜单



 

### 参考

[Electron Tray](https://www.electronjs.org/docs/api/tray)