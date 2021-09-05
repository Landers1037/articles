---
title: electron多窗口
name: electron-multiwindow
date: 2020-04-05
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron的多窗口


<!--more-->


Electron的多窗口

<!--more-->

## 多窗口

可以有两种模式的多窗口，主进程里的多窗口和渲染进程里创建的多窗口

### 主进程新建多窗口

```javascript
let win

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({
    icon:"./build/left.png", width: 800, height: 600,
    webPreferences: {
    nodeIntegration: true
  },
    defaultFontFamily:{
      standard: "微软雅黑",
      sansSerif: "微软雅黑",
      serif: "微软雅黑",
      monospace: "Consolas",
    }
  });

  if (process.env.WEBPACK_DEV_SERVER_URL) {
    // Load the url of the dev server if in development mode
    win.loadURL(process.env.WEBPACK_DEV_SERVER_URL)
    if (!process.env.IS_TEST) win.webContents.openDevTools()
  } else {
    createProtocol('app')
    // Load the index.html when not in development
    win.loadURL('app://./index.html');
  }

  win.on('closed', () => {
    win = null
  })
}
```

electron-vue的页面创建窗口使用`createWindow()`定义了一个主页面

窗口对象win的`win.loadURL`方法，加载了vue的单页面`index.html`于是窗口的页面会自动打开vue里定义的`/`路由对于的页面。

如果我们想要在主进程里额外打开一个窗口，比如项目介绍页面

```javascript
let intro;

function createIntro () {
  // Create the browser window.
  intro = new BrowserWindow({
    width: 800, height: 600,
    webPreferences: {
    nodeIntegration: true
  }
  });

  if (process.env.WEBPACK_DEV_SERVER_URL) {
    // Load the url of the dev server if in development mode
    intro.loadURL(process.env.WEBPACK_DEV_SERVER_URL)
    if (!process.env.IS_TEST) win.webContents.openDevTools()
  } else {
    createProtocol('app')
    // Load the index.html when not in development
    intro.loadURL('https://github.com');
  }

  intro.on('closed', () => {
    intro = null
  })
}
```

这里定义了一个新窗口`intro`，它打开的是`'https://github.com'`网页。

#### 实例化窗口

```javascript
app.on('ready', async () => {
  if (isDevelopment && !process.env.IS_TEST) {
    // Install Vue Devtools
    // Devtools extensions are broken in Electron 6.0.0 and greater
    // See https://github.com/nklayman/vue-cli-plugin-electron-builder/issues/378 for more info
    // Electron will not launch with Devtools extensions installed on Windows 10 with dark mode
    // If you are not using Windows 10 dark mode, you may uncomment these lines
    // In addition, if the linked issue is closed, you can upgrade electron and uncomment these lines
    // try {
    //   await installVueDevtools()
    // } catch (e) {
    //   console.error('Vue Devtools failed to install:', e.toString())
    // }

  }
  createWindow();
  createIntro();
  });
});
```

在`app`加载完毕后初始化窗口，打开应用就会自动打开主页和介绍页面

### 主进程里加载vue路由页面

```javascript
let test;

function createTest () {
  // Create the browser window.
  test = new BrowserWindow({
    width: 800, height: 600,
    webPreferences: {
    nodeIntegration: true
  }
  });

  if (process.env.WEBPACK_DEV_SERVER_URL) {
    // Load the url of the dev server if in development mode
    test.loadURL(process.env.WEBPACK_DEV_SERVER_URL)
    if (!process.env.IS_TEST) win.webContents.openDevTools()
  } else {
    // Load the index.html when not in development
    test.loadURL('app://./index.html#/test');
  }

  test.on('closed', () => {
    test = null
  })
}
```

加入我们的**vue-router**里定义了页面`Test`，路由为`/test`

在开发环境下，`loadUrl()`加载的url为`http://127.0.0.1:8080/#/test`，即你在vue.config里设置的开发服务器地址

在生产环境下，加载的url为`app://./index.html#/test`，打包后默认的文件夹为app所以不需要更改

> **注意**：
>
> vue-router里的mode必须为普通hash模式，不能是history模式
>
> 所以loadUrl里的所有url均有#前缀
>
> 注意index.html#/test，而不是index.html/#/test

第二种方法即使用vue多页面构建的方法



### 渲染进程新建窗口

在渲染进程里新建窗口需要和主进程通信，所以会用到`ipc`,`remote`模块

**渲染进程**

```javascript
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

**主进程**

```javascript
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})
```

**在electron vue里使用**

```javascript
const { ipcRenderer } = require('electron')
methods:{
    test(){
        ipcRenderer.send('openlog')
    }
}

在主进程文件里添加监听
ipcMain.on('openlog', (event, arg) => {
  console.log('打开窗口')
  ...
  createTest();
})
```



### 参考

[Electron BrowserWindow](https://www.electronjs.org/docs/api/browser-window)

[Electron Ipcmain](https://www.electronjs.org/docs/api/ipc-main)