---
title: electron右键菜单
name: electron-contextmenu
date: 2020-04-05
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron里配置右键菜单


<!--more-->


Electron里配置右键菜单

<!--more-->

## 概念

**主进程和渲染进程**

写好的web页面都会运行在渲染进程里，默认的`Menu`菜单属于主进程。

需要在渲染进程里添加菜单则需要使用`Remote`模块

### 菜单

**Menu**

Menu模块是一个菜单对象，可以用于创建原生窗口菜单和上下文菜单

**主进程**

```javascript
const { app, Menu } = require('electron')

const template = [
  // { role: 'appMenu' }
  ...(isMac ? [{
    label: app.name,
    submenu: [
      { role: 'about' },
      { type: 'separator' },
      { role: 'services' },
      { type: 'separator' },
      { role: 'hide' },
      { role: 'hideothers' },
      { role: 'unhide' },
      { type: 'separator' },
      { role: 'quit' }
    ]
  }] : []),
  // { role: 'fileMenu' }
  {
    label: 'File',
    submenu: [
      isMac ? { role: 'close' } : { role: 'quit' }
    ]
  },
  // { role: 'editMenu' }
  {
    label: 'Edit',
    submenu: [
      { role: 'undo' },
      { role: 'redo' },
      { type: 'separator' },
      { role: 'cut' },
      { role: 'copy' },
      { role: 'paste' },
      ...(isMac ? [
        { role: 'pasteAndMatchStyle' },
        { role: 'delete' },
        { role: 'selectAll' },
        { type: 'separator' },
        {
          label: 'Speech',
          submenu: [
            { role: 'startspeaking' },
            { role: 'stopspeaking' }
          ]
        }
      ] : [
        { role: 'delete' },
        { type: 'separator' },
        { role: 'selectAll' }
      ])
    ]
  },
  // { role: 'viewMenu' }
  {
    label: 'View',
    submenu: [
      { role: 'reload' },
      { role: 'forcereload' },
      { role: 'toggledevtools' },
      { type: 'separator' },
      { role: 'resetzoom' },
      { role: 'zoomin' },
      { role: 'zoomout' },
      { type: 'separator' },
      { role: 'togglefullscreen' }
    ]
  },
  // { role: 'windowMenu' }
  {
    label: 'Window',
    submenu: [
      { role: 'minimize' },
      { role: 'zoom' },
      ...(isMac ? [
        { type: 'separator' },
        { role: 'front' },
        { type: 'separator' },
        { role: 'window' }
      ] : [
        { role: 'close' }
      ])
    ]
  },
  {
    role: 'help',
    submenu: [
      {
        label: 'Learn More',
        click: async () => {
          const { shell } = require('electron')
          await shell.openExternal('https://electronjs.org')
        }
      }
    ]
  }
]

const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)
```

使用`Menu()`初始化一个菜单对象，新建一个菜单模板，使用`buildFromTemplate`为菜单应用模板

`Menu`对象的函数

#### [`Menu.setApplicationMenu(menu)`](https://www.electronjs.org/docs/api/menu#menusetapplicationmenumenu) 静态方法用于设置主程序的菜单，为null时隐藏菜单

#### [`Menu.buildFromTemplate(template)`](https://www.electronjs.org/docs/api/menu#menubuildfromtemplatetemplate) 从模板数组加载菜单

实例方法：

`menu.popup()` 显示菜单，在渲染进程里作为上下文菜单弹出

`menu.closePopup()` 隐藏菜单

`menu.append()` 向菜单对象中添加菜单项

`menu.insert()` 向菜单对象中添加指定pos位置的菜单项

### 右键菜单

在渲染进程里使用`remote`创建菜单对象

```javascript
const { remote } = require('electron')
const { Menu, MenuItem } = remote

const menu = new Menu()
menu.append(new MenuItem({ label: 'MenuItem1', click() { console.log('item 1 clicked') } }))
menu.append(new MenuItem({ type: 'separator' }))
menu.append(new MenuItem({ label: 'MenuItem2', type: 'checkbox', checked: true }))

window.addEventListener('contextmenu', (e) => {
  e.preventDefault()
  menu.popup({ window: remote.getCurrentWindow() })
}, false)
```

使用`MenuItem()`方法新建每一个菜单项

>  注意项：
>
> 在electron-vue的右键菜单配置时，使用添加监听器的方式在路由变化时会失效。使用
>
> ```javascript
>             window.oncontextmenu = function () {
>                 that.menu.popup({ window: remote.getCurrentWindow() });
>                 return false
>             }
> ```
>
> 

### 注意事项

#### 菜单的生命周期

在定义一个右键菜单或者托盘菜单时，如果使用`let menu1 = new Menu()`的形式定义菜单对象。

`menu1`对象必须在全局变量里定义，这样保证该变量不能随着垃圾回收机制被回收导致菜单失效

#### 菜单的监听

一般使用`click() >= {}`的方式监听菜单的点击。但是最好使用上下文菜单的方式添加一个新的菜单按钮监听，而不是直接在click中定义函数

```javascript
menu1.append(new MenuItem({label:"设置",click(){createsetting()}}));
```

避免使用模板里的`click()`

```javascript
{
  label: "设置",
  click: () => {
  win.webContents.openDevTools();
}
```



###  参考

 [Electron Menu](https://www.electronjs.org/docs/api/menu)

[Electron MenuItem](https://www.electronjs.org/docs/api/menu-item)