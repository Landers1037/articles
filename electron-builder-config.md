---
title: electron-builder配置
name: electron-builder-config
date: 2020-04-05
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron Builder的配置文件说明


<!--more-->


Electron Builder的配置文件说明

<!--more-->

### 测试打包

添加--dir参数，仅仅打包测试的文件，不会生成系统安装包

```bash
electron:build --dir
```

## 正式打包

### 公共配置

使用vue-electron-builder时，打包文件配置集成在`vue.config.js`

使用electron-builder正常打包时，配置文件写在`package.json`里

**vue.config.js**

```javascript
pluginOptions: {
    electronBuilder: {
      builderOptions: {
        // options placed here will be merged with default configuration and passed to electron-builder
        appId: "",
        productName: "",
        icon: "icon.ico",
        nsis:{

        },
        extraFiles:[
        ]
      }
    }
  },
```

`appId` 程序的默认ID，如com.xxx.xxx

`productName` 程序名称，生成的exe文件名

`copyright` 版权信息

**以下meta信息 写在package.json里**

`name` 程序名称

`description` 程序描述

`homepage` 程序的主页，URL地址

`author` 程序作者



### 文件配置

程序需要依赖的三方程序文件，或其他文件资源

默认会打包`node_modules`下的全部文件

 `extraResources` 打包时复制三方依赖到`resoureces`目录下(resources目录即安装程序后的需要加载的编译文件目录)

`extraFiles` 打包时复制三方文件到程序的根目录下的目录内

定义from和to

```javascript
extraFiles: {
          "./build/update.exe"
}
```

以上配置，会打包根目录下的`build/update.exe`。同时复制到安装目录下的`build/update.exe`

如需自定义from和to选择

```javascripts
{
    "from": "path/to/source",
    "to": "path/to/destination",
}
```

### WINDOWS NSIS打包

```javascript
nsis:{
      oneClick: false,
      allowToChangeInstallationDirectory: true,
      runAfterFinish: false,
      shortcutName: ""
}
```

`oneClick` 是否开启一键安装，默认True

`perMachine` 是否针对每台设备安装，或者针对每个用户安装，默认false

`allowElevation` 允许高权限安装，默认false

`allowToChangeInstallationDirectory` 是否允许安装自定义路径，默认false

### 参考

[Electron Buider](https://www.electron.build/configuration/configuration)