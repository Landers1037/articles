---
title: electron高级配置使用
name: electron-advance-use
date: 2020-05-28
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron开发的高级使用方法


<!--more-->


Electron开发的高级使用方法

<!--more-->

## Builder配置

### 跨平台编译

Electron-Builder支持跨平台的编译，可以针对不同的平台指定编译方式

```javascript
builderOptions: {
    "win": {
    "target": [
      {
        "target": "nsis",
        "arch": [
          "x64",
          "ia32"
        ]
      }
    ]
  }
}
```



### NSIS配置

针对Win平台使用Nsis编译更加快速

```javascript
builderOptions: {
        nsis:{
          oneClick: false,
          allowToChangeInstallationDirectory: true,
          runAfterFinish: false,
          shortcutName: "Pan",
          deleteAppDataOnUninstall: true,
          preCompressedFileExtensions: [".mp4",".png",".ico",".webp"],
          warningsAsErrors: false,
          createStartMenuShortcut: true,
        },
}
```

`oneClick` 是否一键安装打开，默认安装在C盘

`allowToChangeInstallationDirectory` 在不使用一键安装时，此选项可以更改安装目录

`runAfterFinish` 是否在安装完成后打开

`deleteAppDataOnUninstall` 卸载后是否删除文档缓存，这个默认不开启

`preCompressedFileExtensions` 打包时排除的文件后缀

### 优化打包大小

```javascript
compression: 'maximum',
```

compression参数支持`strore,normal,maximum`，最快打包选择store一般在测试时使用

### asar优化使用

默认打包开启asar，会将编写的JavaScript文件统一打包到`app.asar`中。asar是electron专用的封装格式可以将代码封装为一个独立的包，在运行时从内部读取执行。

缺点：asar是虚拟环境的，在执行过程中和文件相关的操作API可能无法使用，例如`fs.OpenSync`。此外asar是只读的，如果有文件的修改工作不应在asar路径里执行。

对于需要增量更新的应用，有时只需要更新几个JS文件或者静态文件，此时重新安装会下载较大的安装包。可以通过`asarUnpack`的方式实现增量更新。

```javascript
asar: true,
asarUnpack:[
	'**/js/*',
	'**/css/*.css',
	'**/xxx.png'
],
```

**增量更新实现**

在`asarUnpack`参数中加入不需要打包进asar的文件，最终生成的目录下就会存在`app.asar`和`app.asar.unpacked`。增量更新的文件只需要直接放在`unpacked`目录下即可

> 注意：asarUnpack数组里的文件路径是来自于`app.asar`的，只需要填写文件在app.asar内部的相对路径即可
>
> 不使用asar可以将运行代码直接放在文件运行目录下，不过这样有暴露源码的风险
>
> 使用`asar :false`参数即可不使用asar打包

### 使用外部js作为配置

默认所有的js都会封装到asar里，如果需要一个外部js文件作为配置文件

**config.js**

```javascript
const a = '123'

export {a}
```

通过引入模块的方式引入 `import {a} from './config.js'`即可在其他地方使用该配置

把`config.js`文件放在public目录下就不被被编译进入app.js（vue适用）

在index.html中引入js文件

```html
<script src="config.js" type="module"></script>
```

> 注意： 这里的type一定要是`module`代表以模块方式导入，不能是`text/javascript`
>
> 如果以js导入，在使用时会提示`unexpected token`错误

## 渲染进程优化

### Ipcmain交互 IpcRenderer交互

当渲染进程需要和主进程交互时，渲染进程里使用IpcRenderer，主进程使用IpcMain

```javascript
ipcRenderer.send('asynchronous-message', 'ping')
//向主进程发送ping
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
```

如果使用sync监听则会阻塞进程，只有返回消息时才能继续执行



### Frameless窗口

使用**BrowserWindow**的`frameless`参数可以实现无边框的窗口，一般用在自定义菜单按钮

```javascript
const { BrowserWindow } = require('electron')
let win = new BrowserWindow({ width: 800, height: 600, frame: false })
win.show()
```

> 注意：无边框后不能触发拖拽移动，所以要自己实现拖拽移动的功能

```javascript
<body style="-webkit-app-region: drag">
</body>

button {
  -webkit-app-region: no-drag;
}
```

在允许拖拽的区域添加样式`-webkit-app-region: drag`，将内部的按钮设置为`-webkit-app-region: no-drag;`否则按钮将无法使用。

```javascript
.titlebar {
  -webkit-user-select: none;
  -webkit-app-region: drag;
}
```

对于可拖拽区域内的文字同样添加`-webkit-app-region: drag;`属性使其可以在被选中时拖动

### 跨域请求

默认浏览器都会开启跨域策略，禁止非同源的请求，这要在后端进行设置允许同源。

可以在创建进程时默认关闭同源策略，以允许任意非同源的访问

```javascript
webPreferences: {
    nodeIntegration: true,
    webSecurity: false,
    allowRunningInsecureContent: true,
    webgl: false,
}
```

设置`webSecurity`为false以禁用同源策略

## 渲染窗口对象

### webcontents

webcontents对象用于渲染以及控制web页面

#### 实例方法：

`webContents.getAllWebcontents()` 获取全部web页面的数组

`webContents.getFocusedWebContents()` 获取当前聚焦的窗口对象

`webContents.fromId(id)` 根据id获取web页面对象

`webContents.loadURL()` 根据URL加载页面

`webContents.loadFile()` 加载本地文件

`webContents.downloadURL()` 下载远程地址文件

`webContents.getURL()` 获取当前页面URL

`webContents.reload()` 重载当前页面

`webContents.reloadIgnoringcache()` 忽略缓存并强制重载

### session

管理浏览器会话，缓存，cookie，代理设置

```javascript
const { BrowserWindow } = require('electron')

let win = new BrowserWindow({ width: 800, height: 600 })
win.loadURL('http://github.com')

const ses = win.webContents.session
console.log(ses.getUserAgent())
```

#### 部分实例：

`ses.getCacheSize()` 获取cache大小

`ses.clearCache()` 清除cookie缓存

`ses.clearStroage()` 清除本地存储， 包括`appcache`, `cookies`, `filesystem`, `indexdb`, `localstorage`, `shadercache`, `websql`, `serviceworkers`, `cachestorage`.

## 参考资料

[webcontents](https://www.electronjs.org/docs/api/web-contents)

[browserwindow](https://www.electronjs.org/docs/api/browser-window)

[session](https://www.electronjs.org/docs/api/session)