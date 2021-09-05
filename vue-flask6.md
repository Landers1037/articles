---
title: vue的props传值
name: vue-flask6
date: 2020-02-16
id: 0
tags: [vue]
categories: []
abstract: ""
---


Vue中子组件的props传值


<!--more-->


Vue中子组件的props传值

<!--more-->

 vue使用的是单向数据流，即数据由父组件传入子组件后，在子组件内修改的数据不会引起父组件的改变

## props

使用props可以在父子间传递数据

### 静态数据

子组件示例

```javascript
<template>
    <div class="container">{{ title }}</div>
</template>
<script>
    export default {
        name: "son",
    	props: ['title'],
}
</script>
```

父组件示例

```javascript
<template>
    <son title="hello"></son>
</template>
<script>
    export default {
        name: "father",
		components: {son}
}
</script>
```

此时传入的数据是静态的

### 动态传值

子组件示例

```javascript
<template>
    <div class="container">
        <p>your name: {{name}}</p>
		<p>your age: {{age}}</p>
    </div>
</template>
<script>
    export default {
        name: "son",
    	props: ['name','age'],
}
</script>
```

父组件示例

```javascript
<template>
    <son :name="name" :age="age"></son>
</template>
<script>
    export default {
        name: "father",
		components: {son},
        data(){
            return{
                name: "pig",
                age: 20
            }
        }    
}
</script>
```

使用v-bind可以向子组件绑定动态的数据

### props类型

一般定义props的方式

```javascript
props: ["p1","p2"]
```

你可以指定传值的类型

```javascript
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
//来自vuejs官方文档
```

### props数据流使用

作为官方文档的补充

当我们在子组件中需要使用传入的props，并且对它进行修改或者引用时。需要现在`data`中初始化

```javascript
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
//来自vuejs官网的示例
```

加入我们需要使用这个传入的`initialCounter`数据，就像

```javascript
<template>
    <div>
    <p>couter now is:{{ counternow }}</p>
	</div>
</template>
<script>
        ...
    computed: {
        counternow: function(){
            return this.counter++
        }
    }
</script>        
```

### props类型检查

根据官网示例，在定义props时可以指定其类型和是否是必须属性

```javascript
props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
}
```

当 prop 验证失败的时候，(开发环境构建版本的) Vue 将会产生一个控制台的警告。