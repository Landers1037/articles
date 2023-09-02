---
title: vue-cli3使用中的问题
name: vue-flask3
date: 2020-01-14
id: 0
tags: [vue,javascript]
categories: []
abstract: ""
---


使用**vue-cli3**开发工具时可能遇到的问题

<!--more-->

## 全局配置

### vue.config.js

新的CLI3使用`vue.config.js`来配置webpack内容，在使用脚手架创建项目时此文件不会自动创建，我们需要在`src`根目录下创建该文件

```javascript
module.exports = {
    lintOnSave: false,
    publicPath: process.env.NODE_ENV === 'production'
        ? '/production-sub-path/'
        : '/',
    runtimeCompiler: true,
    devServer: {
        // overlay: {
        //     warning: false,
        //     errors: false
        // },

        port: 8080,
        proxy: {
            //默认的api路径就直接写根目录，后期调试在改为/api
            '/': {
                target: 'http://127.0.0.1:5000/',
                changeOrigin: true,
                ws: false,
                pathRewrite: {
                    '^/': ''
                },
            },

        },
    },
}
```

在这里配置axios代理请求的后端地址，和ESLINT语法规范等

### 静态目录

新版的vue的静态文件夹由`static`改为`src/public`，所有的静态文件都可以放在该路径下，访问的格式为`"/logo.png"`,假设文件路径为`public/logo.png`

### 路由

在添加vue-router包后，如果使用的时vue的包管理器下载的包会直接在`src`下创建路径src/`view`作为存放vue组件的路径，否则使用npm安装后自行在src目录下创建组件路径

## 问题

### sockjs-node请求错误

使用调试工具查看后台请求时，会看到sockjs-node的请求失败错误，vue使用ws保证代码热更新后浏览器端可以正常重新加载。

当使用局域网和wifi时，ws请求的地址可能无法到达，就会造成请求失败。我们直接关闭ws

```javascript
proxy: {
        '/': {
             target: 'http://127.0.0.1:5000/',
             changeOrigin: true,
             ws: false,
             ...
```

在`vue.config.js`中禁止ws

### 浏览器地址无法访问

假如定义了路由`/home`，使用vue的路由跳转可以直接访问该地址。但是当在浏览器直接输入`ip地址/home`时就会提示不存在此路径。

因为路由的跳转由vue-router代理，当router检测到有路由变更时才会产生响应。我们可以关闭history模式来保证浏览器可以直接访问地址。

```javascript
const router = new VueRouter({
  // mode: 'history',
  routes
});
```

### 图片动态加载

假如使用src设置图片的地址，浏览器可以正确访问到资源。当使用`:src="img_url"`时我们使用v-bind绑定src属性为img_url，在data中设定

```javascript
data: {
    img_url: "../assets/1.jpg"
}
```

此时浏览器无法加载出图片，因为v-bind绑定的地址在经过vue打包渲染后并不会指向原本的图片位置。我们需要使用`require`引入图片资源

```javascript
data: {
    img_url: require("../assets/1.jpg");
}
```

另一种解决方法：

使用`background:url("")`的形式加载图片，url内引入的图片可以是图片的相对路径

### 全局样式

`.vue`单文件中的`<style scoped>`定义的样式属于局部样式，只能作用于本vue组件，如果想要应用全局的style样式就在`.vue`文件中添加`<style>`标签

### this获取

在方法中想要使用this获取数据或对象时，有时会报错`undefined`，在方法定义的前面指定this

```javascript
test(){
    let _this = this;
    _this.data = xxx;
}
```

