---
title: vue和flask跨域交互
name: vue-flask1
date: 2020-01-12
id: 0
tags: [vue,flask]
categories: []
abstract: ""
---


使用axios在vue项目中进行跨域请求


<!--more-->


使用axios在vue项目中进行跨域请求

<!--more-->

## 依赖

axios，flask-cors

## Vue部分

要使用axios，首先引入axios包并且定义api请求路径

**main.js**

```javascript
import axios from 'axios'

Vue.config.productionTip = false
axios.defaults.timeout = 5000 // 请求超时
axios.defaults.baseURL = '/'  // api 即上面 vue.config.js 中配置的地址
```

**vue.config.js**

```javascript
module.exports = {
    publicPath: process.env.NODE_ENV === 'production'
        ? '/production-sub-path/'
        : '/',
    runtimeCompiler: true,
    devServer: {
        port: 8080,
        proxy: {
            //默认的api路径就直接写根目录，后期调试在改为/api
            '/': {
                target: 'http://127.0.0.1:5000/',
                changeOrigin: true,
                ws: true,
                pathRewrite: {
                    '^/': ''
                }
            }
        }
    }
}
```

 🎯因为后端flask运行端口是5000，所以api的请求地址为http://127.0.0.1:5000/

## Flask部分

使用CORS插件允许服务器的跨域资源

```python
from flask_cors import CORS
test = Blueprint('test',__name__)

cors = CORS(test)
```

此时**test**蓝图的全部资源均可以进行跨域请求

## axios测试

```javascript
methods: {
            login() {
                //使用加密的密码校验方式
                var pass = Base64.encode(this.input2);
                var user = this.input1;
                //判断登录用户
                if (user === 'root'){
                    let user_data = {"user": user,"pass": pass };
                    console.log(user_data);
                    axios
                        .post('/login',user_data)
                        .then(res => {
                            console.log(res);
                        });

                }

            }
```

