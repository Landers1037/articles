---
title: 学习electron时遇到问题
name: learn-from-electron
date: 2020-04-07
id: 0
tags: [electron]
categories: []
abstract: ""
---


在学习Electron中遇到的问题总结

<!--more-->

## 设计运行问题

### electron对象变量周期

electron定义的窗口渲染进程变量和主进程变量一开始应该定义为全局变量

在全局的`background.js`里

```javascript
// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win;

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({ width: 800, height: 500,
    backgroundColor: '#2f343f',
    webPreferences: {
      nodeIntegration: true,
  }
  });
}    
```

在`app`建立完成后调用`createWindow`初始化即可。

如果是在一个函数内部创建的窗口对象，则会在该窗口失效后自动回收该对象，导致窗口消失

### 白屏闪烁问题

因为web响应的问题，可能会出现创建窗口时出现窗口闪烁的情况

解决方案1：

为程序设计背景色(较深的背景色)，这样当页面未完成渲染的时候就会先显示带有背景色的空窗口

```javascript
function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({ width: 800, height: 500,
    backgroundColor: '#2f343f',
    webPreferences: {
      nodeIntegration: true,
  }
  });
}  
```

解决方案2：

使用**骨架屏**，在页面未渲染完成时优先显示程序的整体骨架样式。

同时可以使用懒加载，懒加载部分不需要优先显示的资源，优先加载需要用户最先看到的页面资源

解决方案3：

使用延时加载的方案：

对于多窗口的渲染进程，在一个主进程渲染时，可以使用延时函数先加载渲染页面，然后再显示。

对应的`show: false`属性需要添加

```javascript
function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({ width: 800, height: 500,
    show: false,
    backgroundColor: '#2f343f',
    webPreferences: {
      nodeIntegration: true,
  }
  });
}
...


createWindow();
setTimeout(()=>{
    win.show();
  },2500);
```

`win`创建后并不会显示，而是在2.5s后再显示。2.5s这段时间就是等待资源渲染的时间

`electron`为了保证窗口的加载不会出现白屏，还提供了窗口不同的事件监听方式，如`ready-to-show`

```javascript
const { BrowserWindow } = require('electron')
let win = new BrowserWindow({ show: false })
win.once('ready-to-show', () => {
  win.show()
})
```



### Frameless窗口

没有边框的窗口，使用electron提供的无窗API

```javascript
const { BrowserWindow } = require('electron')
let win = new BrowserWindow({ width: 800, height: 600, frame: false })
win.show()
```

即`frame`属性为false

这时的窗口是一个无边框的窗口且不能进行拖动和自由缩放功能

**添加自定义按钮后使用监听API**

在为窗口加入自定义样式的按钮后使用`IpcRenderrer`向主进程发送信号

```javascript
minisize(){
  ipcRenderer.send('min');
}
```

在主进程里监听信号

```javascript
ipcMain.on('min',()=>{
  win.minimize();
});
```

**解决拖动问题**

```css
.header{
    -webkit-app-region: drag
}
```

只需要在自定义的顶部栏上添加`-webkit-app-region`属性。对应的顶部栏里的按钮一定要添加**无法拖动属性**

```css
button {
  -webkit-app-region: no-drag;
}
```

有时为了禁止拖动时，选择到顶部栏的文本，需要添加`user-select`属性

```javascript
.header {
  -webkit-user-select: none;
  -webkit-app-region: drag;
}
```

**透明窗口**

```javascript
const { BrowserWindow } = require('electron')
let win = new BrowserWindow({ transparent: true, frame: false })
win.show()
```

### 访问白屏

**第一种情况**：打开调试模式查看提示module加载失败

在BrowserWindow建立时需要添加属性`nodeIntegration: true`。打包全部的node_module到运行环境里

**第二种情况**：Vue提示`unexpected token`，检查路由没有问题，node_module也全部打包。查看Vue-router的运行模式，需要使用`hash`模式。使用`history`模式会导致页面无法解析

## 打包问题

### 打包的名称

在`package.json`里定义的名称`name`，即程序的名称

可以使用`app.getName()`获取

使用electron-builder for Vue打包时，`vue.config.js`里定义的`produceName`会覆盖原本`package.json`里的名称。

> 注意：
>
> 虽然名称会覆盖，但是使用app.getName()获得的名称始终是package.json里的名称

使用`app.getPath('exe')`获取的是程序的可执行文件的路径(带有可执行文件名称)

如`C:\electron\test.exe`

使用`app.getAppPath()`获取的是electron打包编译的js入口文件路径，如`C:\electron\resources\app.asar`

### 打包图标

窗口图标使用`icon`属性定义，可以支持`jpg`，`png`，`ico`等图片。但是程序的图标需要是`ico`文件，且尺寸必须大于等于**256x256**。否则会打包失败

### 三方文件

当使用到外部的三方文件时，使用`extraFiles`指定文件，使其打包时包含在安装程序内

### 打包失败cannot commit change

有多方的原因

- 检查js文件是否写错，语法是否有问题。注意分隔符`;`
- 检查打包时`electron.exe`程序是否还在后台运行
- 缓存为即时更新，关闭打包命令窗口重新打包
- 使用`--dir`打包参数测试

### 打包文件过大

使用打包的参数`compression: "maximum"`

打包时会打包进全部的node-modules，查看**package.json**文件的`dependencies`，这些在打包后会全部包含。去掉无用的包。

### 页面打开白板

一般为路由错误，`loadURL()`默认在开发环境下使用`localhost:8080/xxx`的形式加载页面。而打包后使用的是加载本地html文件的方式

默认使用`win.loadURL(file://${__dirname}/app/index.html)`的形式加载，默认时同`app://./index.html`

当需要加载vue的路由页面时`html`后没有`/`，如`app://./index.html#/login`

### windows提示寻找新应用打开链接

在使用`win.loadURL(app://./index.html#/login)`加载页面时，需要提前定义链接地址交给electron处理而不是外部程序打开链接

```javascript
createProtocol('app')
// Load the index.html when not in development
win.loadURL('app://./index.html');
```



### 参考

[Electron Frameless Window](https://www.electronjs.org/docs/api/frameless-window)

[Electron BrowserWindow](https://www.electronjs.org/docs/api/browser-window)

[Electron Protocol](https://www.electronjs.org/docs/api/protocol)