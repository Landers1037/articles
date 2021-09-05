---
title: vue动态组件
name: vue-dynamic-components
date: 2020-04-17
id: 0
tags: [vue]
categories: []
abstract: ""
---


Vue的动态组件使用


<!--more-->


Vue的动态组件使用

<!--more-->

## 动态组件需求

假设一个框架页面`home.vue`，在这个框架里有三个功能`login.vue`，`user.vue`，`setting.vue`

当用户登录后会跳转到user页面里，也可以进入setting页面进行全局设计。当我们需要在一个home框架里完成这些功能而没有路由的跳转时，就会用到**动态组件**

即在home框架里根据需要动态加载子组件的方式

### 使用动态子组件

```javascript
<div class="home">
  <button @click="change">切换组件</button>
  <component :is="myComponent"></component>
</div>
```

使用`is`属性选择当前要加载的组件，`myComponent`是一个组件对象，可以使用函数返回一个组件对象

```javascript
...
  components: {user,setting,login},
  methods:{
    change(){
      if(this.index>=1){
          this.index = 0
      }else{
          this.index = 1
      }
    },
    myComponent(){
        if(this.index===0){return 'user'}else{
            return 'setting'
        }
         
    }    
  }
```

当**change**按钮按下时，组件就会在`user`和`setting`中切换

### keep-alive缓存

使用`keep-alive`标签可以缓存动态组件，而不是在切换组件后就销毁它们

这在需要有历史信息的页面中非常有用

```javascript
<keep-alive>
    <component is:"myComponent"></component>
</keep-alive>
```

使用`include`和`exclude`有条件地缓存组件

```javascript
<keep-alive include="user">
    <component is:"myComponent"></component>
</keep-alive>

//或者使用数组的形式
<keep-alive include="user,setting">
    <component is:"myComponent"></component>
</keep-alive>

<keep-alive :include="['user','setting']">
    <component is:"myComponent"></component>
</keep-alive>
```

`include`表示只缓存包含在里面的组件，`exclude`表示排除包含在数组里的组件

