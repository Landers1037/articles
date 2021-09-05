---
title: Vuex使用
name: learn-vuex
date: 2020-04-18
id: 0
tags: [vue]
categories: []
abstract: ""
---


Vuex插件的使用


<!--more-->


Vuex插件的使用

<!--more-->

## Vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 [devtools extension](https://github.com/vuejs/vue-devtools)，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

> 来自Vuex官网

### 状态管理模式

状态管理包含基础的三个部分，`state`用于驱动应用的数据源，`view`以声明的方式把state映射到视图，`actions`响应视图上的用户输入导致的状态变化。

Vuex将这个状态管理抽离出来，作为一个全局的单例模式

### 核心

`store`是Vuex的核心，store就像一个容器存储你的Vue应用的大部分状态。

`store`的特点：

- `store`管理的状态是响应式的，当store里的状态发生变化，Vue里的所有组件都会响应这个变化
- `store`里的状态不能随意改变，只要通过显式的提交commit操作才能修改，这个操作称为**mutation**

### 示例

```javascript
//创建一个vuex实例
import Vue from "vue"
import Vuex from 'vuex'
Vue.use(Vuex);

const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment (state) {
            state.count++
        }
    }
});

export default store
```

定义一个`state`计数count，定义一个`mutation`为计数增加函数increment

此时通过访问`store.state.count`即可访问状态count，使用`store.commit('increment')`即可提交更改使计数增加1

在Vue引用中引入后使用`this.$store.state.count`操作状态

```javascript
//定义一个方法add用于更改计数
        methods:{
            add(){
                this.count = this.$store.state.count;
                console.log(this.$store.state.count);
                this.$store.commit('increment');
            }
        }
```

每次按下按钮触发add函数，`count`状态就会改变

### state

Vuex的状态管理使**全局的单例模式**，当一个组件需要获取状态时可以使用`store.state.count`，所以可以使用计算属性来直接获取状态值

```javascript
<div>{{getCount}}</div>
...
  computed: {
    getCount () {
      return store.state.count
    }
```

当每个模块化的组件需要频繁地获取和修改状态值时，就会导致`store`对象被频繁地引用。为了解决这个问题Vue提供了从根组件注入的方式

```javascript
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
```

这样我们可以通过`this.$store`来全局访问状态

### mapstate

`mapstate`是辅助函数，当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性，让你少按几次键。

```javascript
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

当state里的状态名称和当前组件定义的状态名称相同时可以简写

```javascript
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

使用对象展开运算符可以简化计算属性的编写

```javascript
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```

### mutation

由一个字符串type和一个回调函数组成，回调函数第一个参数接收`state`

```javascript
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
```

触发此回调函数不能直接调用，只能使用`commit`提交的方式

**提交载荷payload**

即传入额外的参数

```javascript
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

这里第二个参数n代表递增的量，执行`store.commit('increment', 10)`即可传入该变量

除了使用`type`字符串提交还可以通过对象方式提交

```javascript
store.commit({
  type: 'increment',
  amount: 10
})
```

指定提交的mutation字符串和参数

**遵守规则**

定义Vuex状态需要遵循对象设计的规则

- 在使用前最好在store实例里定义好所有用到的state

- 动态地增加state应该遵循对象添加的方式，使用`Vue.set(obj, 'newProp', 123)`或者使用对象展开运算符

- ```javascript
    state.obj = { ...state.obj, newProp: 123 }
    ```

**提交mutation**

在组件里使用`this.$store.commit()`提交或者使用`mapMutation`辅助函数将methods映射为`store.commit`

```javascript
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

> 所有的mutation都是同步事务，使用action可以操作异步事务

### action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

```javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。

### module

为解决store过大问题，Vuex 允许我们将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态对象**。

```javascript
const moduleA = {
  state: { count: 0 },
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

> 更多深入使用参考vuex官网

## 参考

[Vuex](https://vuex.vuejs.org/zh/)

