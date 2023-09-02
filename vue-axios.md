---
title: vue使用axios
name: vue-axios
date: 2020-02-02
id: 0
tags: [vue,javascript]
categories: []
abstract: ""
---


在Vue项目中使用axios

<!--more-->

## 全局引入

```javascript
import axios from 'axios'
import VueAxios from 'vue-axios'

Vue.use(VueAxios,axios);
```

使用

```javascript
this.axios.get()
```



### 注册为属性

```javascript
import axios from 'axios'
Vue.prototype.$http = axios
```

使用

```javascript
this.$http.get()
```

### 局部使用

```javascript
import axios from 'axios';
axios.get()
```

## axios配置

```javascript
axios.defaults.baseURL = 'http://127.0.0.1/4000'
```

使用时会自动拼接

如`axios.get('/api/xxx')`即请求的实际地址为`http://127.0.0.1/4000/api/xxx`

### 跨域代理

#### 设置代理地址

```javascript
    devServer: {
        port: 8080,
        proxy: {
            '/api': {
                target: 'http://127.0.0.1:5000',
                changeOrigin: true,
                ws: false,
                pathRewrite: {
                    '^/api': ''
                },
            },

        },
    },
```

`proxy`里添加后端的api地址，使用`/api`来代替地址`http://127.0.0.1:5000`即当访问使用

`axios.get('/api/xxx')`时，会自动请求5000后端地址。

`pathRewrite`地址重写，对请求的地址重写已保证请求到正确的地址

示例：

```javascript
'^/api': '/api'
请求为 http://127.0.0.1:5000/api/xxx
'^api':'/'
请求为 http://127.0.0.1:5000/xxx
```

示自己后端的api编写修改