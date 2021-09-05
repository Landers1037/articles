---
title: vue路由传参的方式
name: vue-flask8
date: 2020-02-18
id: 0
tags: [vue]
categories: []
abstract: ""
---


vue-router是vue的路由管理插件，使用vue-router可以向路由传递参数


<!--more-->


vue-router是vue的路由管理插件，使用vue-router可以向路由传递参数

<!--more-->

## vue-router注册

```javascript
import VueRouter from 'vue-router'
Vue.use(VueRouter);

const router = new VueRouter({
  routes:[
    {
      path: '/',
      component: ()=>import("./components/index")
    },
    {
      path: '/test',
      name: 'test',
      component: ()=>import("./components/test")
    }
  ]
});

new Vue({
  router,
  render: h => h(App),
}).$mount('#app')

```

我们定义了一个`router`实例，需要在Vue中使用该实例

## 传参方式

### 路由变量

```javascript
goto(id) {
    this.$router.push({
      path: `/test/${id}`,
})
```

此时的routers规则应该为

```javascript
{
     path: '/test/:id',
     name: 'test',
     component: test
}
```

在`test`的组件中，使用`this.$route.params.id`来获取id参数

### params

```javascript
this.$router.push({
      name: 'test',
      params: {
      	id: id
      }
})
```

在`test`组件上，使用`this.$route.params.id`来获取id参数

### query

```javascript
this.$router.push({
      name: 'test',
      query: {
      	id: id
      }
})
```

在`test`组件上使用`this.$route.query.id`获取参数

### 区别

`params`传递参数是隐式传递，`query`是显示传递

使用`query`传参数时，参数会跟在url的后面

```html
//go to test
http://127.0.0.1:8080/test?id=id
```

而使用params传递时，参数不会显示在地址栏

```html
http://127.0.0.1:8080/test
```

<h4 style="color:red">注意</h4>

在浏览器刷新时，缓存中的`params`参数会重置，所以如需页面刷新时数据不会丢失，应该使用`query`传递