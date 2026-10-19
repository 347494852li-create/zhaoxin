# 第 1 章 · 第一个 Vue 应用与核心理念

> **这一章解决什么问题**：不装任何环境，用一个 HTML 文件跑起 Vue，理解「声明式渲染」「数据驱动」「MVVM」这三个词到底是什么意思。
>
> **学完标准**：你能独立写出计数器、待办列表；能解释「为什么改数据页面就变了」；能看懂 `src/main.js` 最后那 6 行。

---

## 一、准备：一个能跑 Vue 的 HTML 文件

新建一个文件夹，里面放 `index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <title>第一个 Vue 应用</title>
</head>
<body>
  <div id="app">
    <h1>{{ message }}</h1>
  </div>

  <!-- 引入 Vue 2.6.12 -->
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.js"></script>
  <script>
    new Vue({
      el: '#app',
      data: {
        message: '你好，Vue'
      }
    })
  </script>
</body>
</html>
```

双击用浏览器打开，页面上显示：**你好，Vue**。

这一小段代码已经包含了 Vue 的全部核心机制，下面逐个拆。

---

## 二、拆第一段代码

### 2.1 `new Vue({ ... })`：创建一个 Vue 实例

```js
new Vue({
  el: '#app',
  data: {
    message: '你好，Vue'
  }
})
```

| 选项 | 作用 |
|---|---|
| `el: '#app'` | 告诉 Vue：接管页面上 id 为 `app` 的那个元素 |
| `data: { ... }` | 这个实例的**数据**；`message` 是其中一个字段 |

`new Vue(...)` 返回一个**实例**，你可以把它理解成「这个页面区域的大脑」。

### 2.2 `{{ message }}`：插值表达式

```html
<h1>{{ message }}</h1>
```

`{{ }}` 叫**插值（interpolation）**：把里面表达式的值，作为文本显示出来。

它不只能放变量：

```html
{{ 1 + 2 }}              <!-- 3 -->
{{ message + '！' }}      <!-- 你好，Vue！ -->
{{ message.length }}      <!-- 7 -->
```

> 只能放**表达式**，不能放语句。`{{ if (x) { ... } }}` 是错的。

### 2.3 数据变，页面自动变

打开浏览器控制台（F12 → Console），输入：

```js
// 注意：需要先让实例暴露到全局才能这样访问
```

更直观的做法是把代码改成：

```js
const vm = new Vue({
  el: '#app',
  data: { message: '你好，Vue' }
})

window.vm = vm
```

然后在控制台执行：

```js
vm.message = '数据变了'
```

页面上的文字**立刻**变成「数据变了」。

你没有写任何「找到 h1、修改它的文字」的代码。这就是 Vue 的第一条核心：**数据驱动**。

---

## 三、计数器：把「行为」接上

完整例子：

```html
<div id="app">
  <h1>计数器：{{ count }}</h1>
  <button v-on:click="increment">加 1</button>
  <button v-on:click="decrement">减 1</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.js"></script>
<script>
new Vue({
  el: '#app',
  data: {
    count: 0
  },
  methods: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    }
  }
})
</script>
```

### 3.1 `methods`：实例的方法

```js
methods: {
  increment() {
    this.count++
  }
}
```

- `methods` 里的函数可以直接在模板里绑定给事件；
- 函数里的 `this` 指向 Vue 实例，所以 `this.count` 就是 `data.count`。

### 3.2 `v-on:click`：事件绑定

```html
<button v-on:click="increment">加 1</button>
```

意思是：**点击这个按钮时，调用 `increment` 方法**。

`v-on:` 可以简写成 `@`：

```html
<button @click="increment">加 1</button>
```

项目里到处是这种写法，例如 `position/index.vue:74` 的 `@click="handleAddEdit()"`。

### 3.3 发生了什么

```
用户点击按钮
  → Vue 调用 increment()
  → this.count++ 修改数据
  → Vue 检测到 count 变了
  → 重新渲染模板里用到 count 的地方（{{ count }}）
  → 页面显示新数字
```

**你只写了「数据怎么变」，没写「界面怎么改」。** 这是和传统 DOM 操作最大的区别。

---

## 四、对比：命令式 vs 声明式

假设要实现「点击按钮，让标题变成当前计数」。

### 4.1 传统写法（命令式：一步步指挥 DOM）

```html
<h1 id="title">计数器：0</h1>
<button id="btn">加 1</button>

<script>
let count = 0
document.getElementById('btn').onclick = function() {
  count++
  document.getElementById('title').textContent = '计数器：' + count
}
</script>
```

你必须自己：

1. 找到按钮；
2. 绑定点击事件；
3. 修改计数；
4. **再找到标题，手动改它的文字**。

界面元素一多，这类「找元素 → 改元素」的代码会迅速膨胀，而且容易漏改。

### 4.2 Vue 写法（声明式：只描述结果）

```html
<h1>计数器：{{ count }}</h1>
<button @click="count++">加 1</button>
```

你只声明：**标题显示 count 的值**。count 怎么变、什么时候重新渲染，Vue 负责。

> **声明式渲染**：你描述「界面应该长什么样」，而不是「怎么一步步把界面改成那样」。

这就是 Vue 官方文档反复强调的「声明式渲染（Declarative Rendering）」。

---

## 五、三个必懂的概念

### 5.1 MVVM

| 字母 | 含义 | 在 Vue 里对应 |
|---|---|---|
| **M**odel | 数据 | `data` |
| **V**iew | 视图 | `<template>` |
| **V**iew**M**odel | 连接数据和视图的中间层 | Vue 实例本身 |

```
      数据变化
data  ──────────>  视图自动更新
  ^                     |
  |      用户输入        |
  └─────────────────────┘
```

Vue 实例就是那个「中间人」：它盯着数据，也盯着视图，两边任何一边变了，它负责同步另一边。

### 5.2 数据驱动

**不要直接操作 DOM，只操作数据。**

```js
// 不推荐（在 Vue 里）
document.querySelector('.title').textContent = '新标题'

// 推荐
this.title = '新标题'
```

项目里你几乎看不到 `document.querySelector`，原因就在这里。

### 5.3 响应式

**Vue 能感知数据变化，并自动更新用到它的界面。**

你在 `data` 里声明的属性，Vue 会把它们改造成「可观察的」：读取时记录「谁用了我」，修改时通知「用我的人更新」。

这就是第 10 章要讲的响应式原理。现在只需要知道：

- 在 `data` 里声明过的属性，修改会触发更新；
- 后来**直接新增**的属性、以及**直接用下标修改数组**，有时不会触发更新（第 10 章讲原因和解法）。

---

## 六、回到本项目：根实例是怎么创建的

### 6.1 `src/main.js` 的最后 6 行

```js
new Vue({
  el: "#app",
  router,
  store,
  render: (h) => h(App),
})
```

| 选项 | 作用 |
|---|---|
| `el: "#app"` | 接管 `public/index.html:148` 的 `<div id="app">` |
| `router` | 装上 Vue Router（第 5 章） |
| `store` | 装上 Vuex（第 6 章） |
| `render: (h) => h(App)` | 用 render 函数渲染根组件 `App` |

### 6.2 为什么用 `render` 而不是 `template`

根实例这里没有写 `template: '<App />'`，而是用了：

```js
render: (h) => h(App)
```

`h` 是 Vue 传进来的**创建元素函数**（createElement）。`h(App)` 的意思是「创建一个 App 组件实例」。

两种写法效果相同：

```js
// 写法一：render 函数
render: (h) => h(App)

// 写法二：模板（需要完整版 Vue，含编译器）
components: { App },
template: '<App />'
```

本项目用写法一，因为它不依赖模板编译器，运行时更小，也是 Vue CLI 项目的常见写法。

> 你现在不需要会写 render 函数，只要认得 `h(App)` 是「渲染根组件」。

### 6.3 `App.vue` 是根组件

`src/App.vue:1-5`：

```vue
<template>
  <div id="app">
    <router-view />
  </div>
</template>
```

`<router-view />` 是一个占位符：**当前路由匹配到哪个页面组件，就在这里显示哪个**。

所以整条链路是：

```
index.html 的 <div id="app">
  → main.js 的 new Vue 接管
  → 渲染 App 组件
  → App 里的 <router-view /> 显示当前路由页面
  → 当前路由页面（如 position/index.vue）显示具体内容
```

第 5 章会把这条链路完整走一遍。

### 6.4 还有一堆 `Vue.use` 和 `Vue.prototype`

`main.js` 前半部分做了两类「全局安装」：

```js
Vue.use(Element, { size: ... })       // 安装 Element UI 组件库
Vue.component('Pagination', Pagination) // 全局注册组件
Vue.prototype.parseTime = parseTime     // 挂全局方法，任何组件里 this.parseTime 可用
```

这些在第 4、7 章分别讲。现在只需要知道：**`main.js` 是全站的公共装配车间**。

---

## 七、为什么组件里的 `data` 必须是函数

先埋一个伏笔。根实例里 `data` 是对象：

```js
new Vue({
  data: { count: 0 }
})
```

但组件里必须是函数：

```js
export default {
  data() {
    return { count: 0 }
  }
}
```

原因：组件会被复用多份。如果 `data` 是对象，所有组件实例会**共享同一个数据对象**，改一个全变；用函数则每次创建实例都返回一个**新对象**，彼此隔离。

第 4 章会结合项目代码再讲一遍。

---

## 八、动手练习

### 练习 1：计数器

不看上面的代码，从零写一个计数器，要求：

- 显示当前数字；
- 「+1」「-1」「归零」三个按钮；
- 数字小于 0 时显示红色。

提示：红色可以用 `:class`（第 2 章详解，这里可以先试）。

### 练习 2：待办列表

要求：

- 一个输入框 + 一个「添加」按钮；
- 输入内容后点添加，加入列表；
- 列表用 `v-for` 渲染（第 2 章会讲，先照猫画虎）；
- 每项后面有「删除」按钮。

```html
<div id="app">
  <input v-model="text" />
  <button @click="add">添加</button>
  <ul>
    <li v-for="(item, index) in todos" :key="index">
      {{ item }}
      <button @click="remove(index)">删除</button>
    </li>
  </ul>
</div>
```

### 练习 3：解释给外行听

用三句话向一个没学过前端的人解释：

1. 什么是 Vue？
2. 什么是数据驱动？
3. 为什么改数据页面会变？

### 练习 4：对照项目

打开 `src/main.js:134-139` 和 `src/App.vue`，画出：

```
index.html 的 div#app
  → new Vue
  → App 组件
  → <router-view>
  → 某个页面
```

这条链路。画不出来就回看 §6。

---

## 九、本章自测清单

- [ ] `new Vue({ el, data, methods })` 里三个选项分别是什么？
- [ ] `{{ }}` 里能放什么，不能放什么？
- [ ] `v-on:click` 和 `@click` 是什么关系？
- [ ] 什么是声明式渲染？它和直接操作 DOM 有什么区别？
- [ ] MVVM 三个字母分别对应 Vue 里的什么？
- [ ] 数据驱动是什么意思？
- [ ] 响应式是什么意思？它保证了什么？
- [ ] `render: (h) => h(App)` 在做什么？
- [ ] `App.vue` 里的 `<router-view />` 是什么？
- [ ] 为什么组件里的 `data` 必须是函数？

---

## 十、下一章前瞻

第 2 章会把模板里能用的语法一次讲全：

- `{{ }}` 的完整规则；
- `v-bind` / `:`、`v-on` / `@`、`v-model`；
- `v-if` / `v-show`、`v-for` / `:key`；
- 事件修饰符、表单修饰符；
- 项目模板里那些 `slot-scope`、`v-loading`、`v-hasPermi` 到底是什么。

> 这一章最重要的一句话：**Vue 让你只改数据，界面自动跟着变；你描述结果，不指挥过程。**
