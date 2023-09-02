---
title: electron如何下载文件
name: how-electron-download
date: 2020-05-28
id: 0
tags: [electron]
categories: []
abstract: ""
---


Electron实现原生的文件下载

<!--more-->

## 原生实现

浏览器构建下载对象blob后会调用浏览器自带的下载器进行下载，在渲染进程里添加下载任务后，会在主进程对其进行统一管理。

### webcontents类

在渲染进程中获取到下载文件的地址后，使用`IpcRenderer`发送给主进程，主进程可以调用`win.webContents.downloadURL(url);`触发下载。

```javascript
ipcMain.on('download',(event, args)=>{
  down.webContents.downloadURL(args);
  down.show();
});
```

### downloaditem对象

每一个下载任务都对应着一个`downloaditem`对象，控制来自远程资源的文件下载。

使用**webcontents**的`'will-download'`事件可以监听下载对象。

```javascript
const { BrowserWindow } = require('electron')
let win = new BrowserWindow()
win.webContents.session.on('will-download', (event, item, webContents) => {
  // 设置保存路径,使Electron不提示保存对话框。
  item.setSavePath('/tmp/save.pdf')

  item.on('updated', (event, state) => {
    if (state === 'interrupted') {
      console.log('Download is interrupted but can be resumed')
    } else if (state === 'progressing') {
      if (item.isPaused()) {
        console.log('Download is paused')
      } else {
        console.log(`Received bytes: ${item.getReceivedBytes()}`)
      }
    }
  })
  item.once('done', (event, state) => {
    if (state === 'completed') {
      console.log('Download successfully')
    } else {
      console.log(`Download failed: ${state}`)
    }
  })
})
```

监听**item**的`state`属性变化，即可知道当前文件的下载进度，主要包括`progressing`,`interrupted`,`completed`,`cancelled`

### 实例方法

#### [`downloadItem.setSavePath(path)`](https://www.electronjs.org/docs/api/download-item#downloaditemsetsavepathpath)  保存到指定下载路径，此语句未出现时会出现保存对话框

#### [`downloadItem.getSavePath()`](https://www.electronjs.org/docs/api/download-item#downloaditemgetsavepath) 在设置了savePath后可以获取到当前保存路径

#### [`downloadItem.pause()`](https://www.electronjs.org/docs/api/download-item#downloaditempause) 暂停下载任务

#### [`downloadItem.isPaused()`](https://www.electronjs.org/docs/api/download-item#downloaditemispaused) 判断当前下载任务是否暂停

#### [`downloadItem.resume()`](https://www.electronjs.org/docs/api/download-item#downloaditemresume) 重新开始下载任务

#### [`downloadItem.canResume()`](https://www.electronjs.org/docs/api/download-item#downloaditemcanresume) 判断当前下载任务能否恢复下载

#### [`downloadItem.cancel()`](https://www.electronjs.org/docs/api/download-item#downloaditemcancel) 取消下载操作

#### [`downloadItem.getURL()`](https://www.electronjs.org/docs/api/download-item#downloaditemgeturl) 获取下载文件的URL

#### [`downloadItem.getMimeType()`](https://www.electronjs.org/docs/api/download-item#downloaditemgetmimetype) 返回文件的MIME类型

#### [`downloadItem.getFilename()`](https://www.electronjs.org/docs/api/download-item#downloaditemgetfilename) 返回文件名称

#### [`downloadItem.getTotalBytes()`](https://www.electronjs.org/docs/api/download-item#downloaditemgettotalbytes) 如果获取到则返回下载文件总大小

#### [`downloadItem.getReceivedBytes()`](https://www.electronjs.org/docs/api/download-item#downloaditemgetreceivedbytes) 返回已经下载的文件大小

#### [`downloadItem.getState()`](https://www.electronjs.org/docs/api/download-item#downloaditemgetstate) 获取下载任务状态

**更多示例参考Electron API**

## Node实现

使用Node的`request`请求库可以在渲染进程实现文件下载

```bash
yarn add request
```

在渲染进程页面引入

```javascript
    const fs = require('fs');
    const os = require('os');
    const request = require('request');
    const path = require('path');

    download(index,url,name){
                let dir = localStorage.getItem('path')?localStorage.getItem('path'):os.homedir();
                let save_path = path.join(dir , name);
                let token = this.$store.state.token;
                if(fs.existsSync(save_path)){
                    this.$message.error('文件已存在');
                    return
                }
                if(!localStorage.getItem('path')){
                    this.$message('未定义保存路径,默认为' +os.homedir());
                }
                this.download_list[index].progress = 'downloading';
                const req = request({
                    method: 'GET',
                    uri: url + '&token=' +token
                });
                const writer = fs.createWriteStream(save_path);
                req.pipe(writer);
                req.on('end',()=>{
                    this.$message('任务下载完成');
                    this.download_list[index].progress = 'complete';
                    this.$store.commit('updateList',this.download_list);
                    this.refresh();
                });
                req.on('error',()=>{
                    this.$message.error('下载失败');
                });

            },
```

使用request请求文件对象，在响应中使用`req.pipe(writer)`方法向文件流内部写入数据

## 参考资料

[download-item](https://www.electronjs.org/docs/api/download-item)

[webcontents](https://www.electronjs.org/docs/api/web-contents)