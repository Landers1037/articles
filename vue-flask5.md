---
title: vue存储token信息
name: vue-flask5
date: 2020-01-15
id: 0
tags: [vue]
categories: []
abstract: ""
---


使用vuex来存储管理生成的token信息用于认证


<!--more-->


使用vuex来存储管理生成的token信息用于认证

<!--more-->

### vuex

```javascript
import Vuex from 'vuex';
const store = new Vuex.Store({
    state:{
        Authorization: localStorage.getItem('Authorization') ? localStorage.getItem('Authorization') : ''
    },
    mutations:{
        changeLogin (state, user) {
            state.Authorization = user.Authorization;
            localStorage.setItem('Authorization', user.Authorization);
        }
    },
})
```

定义vuex插件

### main.js

设置axios拦截器，在每次的请求里添加token信息

```javascript
// http request 拦截器
axios.interceptors.request.use(
    let token = localStorage.getItem('Authorization');
 config => {
  if (token) { // 判断是否存在token，如果存在的话，则每个http header都加上token
   config.headers.Authorization = token;
  }
  return config;
 },
 err => {
  return Promise.reject(err);
 });
 
// http response 拦截器
axios.interceptors.response.use(
 response => {
  return response;
 },
 error => {
  if (error.response) {
   switch (error.response.status) {
    case 401:
     // 返回 401 清除token信息并跳转到登录页面
     store.commit(types.LOGOUT);
     router.replace({
      path: 'login',
      query: {redirect: router.currentRoute.fullPath}
     })
   }
  }
  return Promise.reject(error.response.data) // 返回接口返回的错误信息
});
```

使用`localStorage.getItem('Authorization');`获取本地存储的token信息

### 后端认证

在后端我们需要从请求头中获取token并验证

```python
token = request.headers('Authorization')
```

### 拓展

除了将token放在请求头中还可以直接放在请求的信息里使用get方式获取

假如要传输的token为`1211211234567`，我们修饰请求地址为[http://127.0.0.1:8080/home?token=1211211234567]()

此时认证信息包含在key为token的请求地址里

```python
 token = request.args.get("token")
```

我们在后端获取key为`token`的值即为token信息，然后进行验证

