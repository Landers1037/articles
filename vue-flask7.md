---
title: vue组件生命周期
name: vue-flask7
date: 2020-02-16
id: 0
tags: [vue]
categories: []
abstract: ""
---


需要注意的事项，vue组件的生命周期在注册时的区别


<!--more-->


需要注意的事项，vue组件的生命周期在注册时的区别

<!--more-->

# 生命周期

 `created` 钩子可以用来在一个实例被创建之后执行代码：

```java
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
//来自vuejs官方示例    
```

生命周期钩子的 `this` 上下文指向调用它的 Vue 实例，值得注意的是不能使用**箭头**函数，**箭头**函数的寻找this指向可能会导致错误

### 其他周期

- **beforeCreate**
- **created**
- **beforeMount**
- **mounted**
- **beforeUpdate**
- **updated**
- **beforeDestroy**
- **destroyed**

### 生命周期图

![cycle](http://file.mgek.cc/images/blog/lifecycle.webp)