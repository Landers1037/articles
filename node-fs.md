---
title: node的fs模块
name: node-fs
date: 2020-04-05
id: 0
tags: [javascript]
categories: []
abstract: ""
---


Node.js里的`fs`模块常用api

<!--more-->

### 打开文件

```javascript
fs.open(path, flags[, mode], callback)

fs.open('test', 'r', (err, fd) => {
  console.log(fd)
})
```

支持多种模式`r`,`r+`,`w`,`w+`

### 读取内容

```javascript
fs.read(fd, buffer, offset, length, position, callback)

fs.open('./package.json', 'r', function (err, fd) {
  var readBuffer = new Buffer(1024),
    offset = 0,
    len = readBuffer.length,
    filePostion = 5;
  fs.read(fd, readBuffer, offset, len, filePostion, function (err, readByte) {
    console.log('读取数据总数：' + readByte + ' bytes'); // ==>读取数据总数：2 bytes 
    console.log(readBuffer.slice(0, readByte)); //数据已被填充到readBuffer中 
  })
})
```

### 读取文件

```javascript
fs.readFile(path[,options],callback)

fs.readFile('./package.json', function (err, data) {
	console.log(data);
  })
```

### 写入文件

```javascript
fs.write(fd, buffer, offset, length[, position], callback)

fs.open('./test.txt', 'a', function (err, fd) { 
  var writeBuffer = new Buffer('这是要写入的字符串'), 
  offset = 0, 
  len = writeBuffer.length, 
  filePostion = null; 
  fs.write(fd, writeBuffer, offset, len, filePostion, function(err, readByte){ 
    console.log('写数据总数：'+readByte+' bytes' );     //  写数据总数：27 bytes 
  })
})
```

### 关闭文件

```javascript
fs.close(fd, callback)
```

### 判断文件是否存在

```javascript
fs.stat()

fs.stat('./test.txt', function(err, stat){
    if(stat&&stat.isFile()) {
    console.log('文件存在');
    } else {
    console.log('文件不存在或不是标准文件');
    }
});
```

### 判断是否是文件

```javascript
stat.isFile()
```

### 检查访问权限

```javascript
fs.access()

fs.access('./test.txt', function(err) {
    console.log(err ? '文件存在' : '文件不存在');
});

//检查是否可写
fs.access('/etc/passwd', fs.R_OK | fs.W_OK, function(err) {
    console.lo(err ? '不可操作!' : '可以读/写');
});
```

支持多种模式

`fs.F_OK` 文件对于进程是否可见

`fs.R_OK` 文件是否可读

`fs.W_OK` 文件是否可写

`fs.X_OK` 文件是否可执行

### 文件是否存在

```javascript
fs.exists()
fs.existsSync(file)
```

### 删除文件

```javascript
fs.unlink()
```

### 创建文件夹

```javascript
fs.mkdir(path[,model],callback)
```

### 重命名

```javascript
fs.rename(oldPath,newPath,callback)
```

### 删除文件夹

```javascript
fs.rmdir(path,callback)
```

### 读取路径信息

```javascript
fs.readdir(path,callback)
```

### 参考

[简书](https://www.jianshu.com/p/eec2cae9d93a)

[Nodejs API](http://nodejs.cn/api/fs.html)