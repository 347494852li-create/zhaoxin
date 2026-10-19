# 第 3 章：Vue 核心概念

> **这一章的目标**：把 Vue 的 6 个核心概念讲透：组件、模板语法、响应式、生命周期、computed、props。
>
> **学完能做什么**：看懂项目里任意一个 `.vue` 文件，能自己写一个简单页面。
>
> **预计**：3 小时

---

## 本章地图

| 节 | 概念 | 用一句话说清 |
|---|---|---|
| 3.1 | 什么是组件 | 一个自带 HTML+JS+CSS 的"零件" |
| 3.2 | 模板语法 | 把数据"渲染"到 HTML 上的语法 |
| 3.3 | **响应式（最重要）** | 改数据 → 界面自动变 |
| 3.4 | 生命周期 | 组件从生到死的钩子 |
| 3.5 | computed（计算属性） | 派生的数据，自动更新 |
| 3.6 | props（父子通信） | 父组件给子组件传值 |
| 3.7 | 事件（子传父） | 子组件通知父组件 |
| 3.8 | 组件注册（全局 vs 局部） | 项目里那 13 个全局组件怎么来的 |
| 3.9 | 完整例子 | 手写一个列表页 |

---

# 3.1 什么是组件

## 痛点

传统开发一个页面：`user-list.html` + `user-list.js` + `user-list.css`，三个文件分开。
改一处要开三个文件，而且 HTML 和 JS 的关系看不出来。

## 解决：组件（Component）

**把一个"零件"的 HTML、JS、CSS 写在一个文件里**，扩展名 `.vue`。

**类比**：像 Java 的一个类——**数据（字段）+ 行为（方法）**封装在一起。

```vue
<!-- UserCard.vue  ← 一个组件就是一个文件 -->
<template>
  <!-- ① HTML 结构 -->
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.age }} 岁</p>
    <button @click="sayHi">打招呼</button>
  </div>
</template>

<script>
// ② JS 逻辑
export default {
  data() {
    return {
      user: { name: '张三', age: 20 }
    }
  },
  methods: {
    sayHi() {
      alert('你好，我是 ' + this.user.name)
    }
  }
}
</script>

<style scoped>
/* ③ CSS 样式；scoped = 只作用于这个组件 */
.user-card {
  border: 1px solid #ccc;
  padding: 16px;
}
</style>
```

## ⭐ 三段式的分工

| 块 | 作用 | 类比 |
|---|---|---|
| `<template>` | 结构（HTML） | JSP / Thymeleaf 模板 |
| `<script>` | 逻辑（JS） | Java 类 |
| `<style scoped>` | 样式（CSS） | — |

**`scoped` 是什么？** 让样式**只作用于当前组件**，不会污染别人的 `.user-card`。

**怎么做到的？** 编译时给元素加唯一属性，选择器也加上这个属性：

```html
<!-- 编译后 -->
<div class="user-card" data-v-7ba5bd90>
```
```css
.user-card[data-v-7ba5bd90] { ... }
```

**类比**：像 Java 的 `private`——限定作用域。

## 项目里的组件在哪

```
zxcm-ui/src/
├── views/               ← 页面级组件（一个页面 = 一个组件）
│   ├── login.vue
│   ├── index.vue
│   └── contract/contractDraft/index.vue
├── components/          ← 可复用的公共组件
│   ├── Pagination/      分页
│   ├── DictTag/         字典标签
│   ├── FileUpload/      文件上传
│   └── MoneyInput/      金额输入
├── layout/              ← 布局组件
│   ├── index.vue        后台外壳
│   └── components/
│       ├── Sidebar/     侧边栏
│       ├── Navbar/      顶栏
│       └── AppMain/     内容区
└── App.vue              ← 根组件
```

**区分 `views` 和 `components`**：

| 目录 | 放什么 | 特点 |
|---|---|---|
| `views/` | **页面**（路由对应的） | 一个页面一个组件，不会被别处复用 |
| `components/` | **公共组件**（可复用） | 多个页面都能用 |

**类比**：`views` 像一个 Controller，`components` 像 Service/Util。

---

# 3.2 模板语法（把数据渲染到 HTML）

## 六种语法

```html
<template>
  <div>
    <!-- ① 插值：显示变量 -->
    <p>{{ message }}</p>
    <p>{{ user.name }}</p>
    <p>{{ 1 + 1 }}</p>              <!-- 可以写表达式 -->

    <!-- ② 绑定属性（v-bind 简写为 : ） -->
    <img :src="avatarUrl" />
    <el-input :value="name" />
    <el-table :data="list">

    <!-- ③ 绑定事件（v-on 简写为 @ ） -->
    <button @click="handleClick">点我</button>
    <el-input @input="handleInput" />

    <!-- ④ 双向绑定 -->
    <el-input v-model="form.username" />

    <!-- ⑤ 条件渲染 -->
    <div v-if="loading">加载中...</div>
    <div v-else-if="error">出错了</div>
    <div v-else>内容</div>
    <div v-show="visible">显示/隐藏（display:none）</div>

    <!-- ⑥ 循环渲染 -->
    <div v-for="(item, index) in list" :key="item.id">
      {{ index }}: {{ item.name }}
    </div>
  </div>
</template>
```

## ⭐ `v-if` vs `v-show`（面试常问）

| | `v-if` | `v-show` |
|---|---|---|
| 怎么实现 | **销毁/重建 DOM** | 改 CSS `display: none` |
| 初始渲染开销 | 条件为假时不渲染 | **总是渲染** |
| 切换开销 | 大（要重建） | **小**（只改样式） |
| 用在哪 | 条件**很少变** | 条件**频繁切换** |

**项目里的选择**：

```html
<!-- 搜索框：用 v-show（经常显示/隐藏，切换快） -->
<el-form v-show="showSearch" ref="queryForm" :model="queryParams">

<!-- 弹窗：用 v-if（很少打开，不需要一直渲染） -->
<el-dialog v-if="showApplyForm" ...>
```

## ⭐ `:key` 为什么必须有

```html
<div v-for="(item, index) in list" :key="item.id">
```

**`key` 是 Vue 用来"识别每个元素身份"的标记。**

**不写会怎样？** Vue 会用"索引"当 key，导致**复用错误**：

```javascript
// 原列表
[{ id: 1, name: 'A' }, { id: 2, name: 'B' }]

// 删除 A 后
[{ id: 2, name: 'B' }]
```

**如果按索引当 key**：Vue 以为"第 0 个变了内容"（不是"删除了一个"），
会**复用**第 0 个元素的 DOM，导致输入框里的内容错乱。

**用 `item.id` 当 key**：Vue 知道"id=1 的被删了"，正确地移除 DOM。

**⭐ 规则：`v-for` 一定要加 `:key`，且用**唯一 id**，不要用索引。**

## 项目里的真实例子

```html
<!-- src/views/contract/contractDraft/index.vue 的搜索表单 -->
<el-form
  v-show="showSearch"
  ref="queryForm"
  :model="queryParams"
  size="small"
  :inline="true"
  label-width="68px"
>
  <el-col :span="6">
    <el-form-item label="合同名" prop="contractName">
      <el-input
        v-model="queryParams.contractName"
        placeholder="请输入合同名"
        clearable
        @keyup.enter.native="handleQuery"
      />
    </el-form-item>
  </el-col>
</el-form>
```

| 代码 | 含义 |
|---|---|
| `v-show="showSearch"` | 搜索区可折叠 |
| `ref="queryForm"` | 起个引用名，`this.$refs.queryForm` 能拿到 |
| `:model="queryParams"` | 表单绑定到查询参数对象 |
| `:inline="true"` | 行内布局（多个输入框一行） |
| `label-width="68px"` | 标签宽度 |
| `v-model="queryParams.contractName"` | 双向绑定 |
| `@keyup.enter.native="handleQuery"` | 回车就搜索 |

**⭐ `clearable` 是什么？** Element UI 的属性，输入框显示清空按钮。

**⭐ `@keyup.enter.native`** = 监听原生 keyup 事件，且只响应 Enter 键。

---

# 3.3 响应式（⭐ 最重要的一节）

## 一句话

> **改数据 → 界面自动变。你不需要操作 DOM。**

## 对比：传统 vs Vue

### 传统 JS（手动操作 DOM）

```javascript
// 拿到数据后，手动改界面
const data = await fetch('/api/list').then(r => r.json())

const tbody = document.getElementById('list-body')
tbody.innerHTML = ''                           // 清空
data.forEach(item => {
  const tr = document.createElement('tr')
  tr.innerHTML = `<td>${item.name}</td><td>${item.age}</td>`
  tbody.appendChild(tr)                        // 逐行插入
})
```

**问题**：数据变了要重新写一遍这套操作，容易漏、容易错。

### Vue（改数据，界面自动变）

```html
<el-table :data="list">
  <el-table-column prop="name" label="姓名" />
  <el-table-column prop="age" label="年龄" />
</el-table>
```

```javascript
async loadData() {
  this.list = await fetch('/api/list').then(r => r.json())
  // ↑ 只改这一行，表格自动重绘
}
```

**⭐ 这就是响应式**：Vue 知道 `list` 被改了，自动重新渲染用到了 `list` 的地方。

## 类比

**Excel**：你改单元格的值，图表自动重画。你不需要"手动去画图"。

**Vue**：你改 `data` 里的变量，界面自动重绘。你不需要"手动操作 DOM"。

## ⭐ 只有 `data()` 里的字段才是响应式的

```javascript
data() {
  return {
    count: 0,              // ✅ 响应式
    list: [],              // ✅ 响应式
    user: { name: '' }     // ✅ 响应式（嵌套的也是）
  }
}
```

```javascript
methods: {
  add() {
    this.count++           // ✅ 界面会更新
  }
}
```

**⚠️ 不在 `data()` 里的变量不是响应式的**：

```javascript
data() {
  return { count: 0 }
},
created() {
  this.notReactive = 100       // ❌ 不是响应式，改了界面不变
}
```

**原因**：Vue 在 `data()` 返回时，用 `Object.defineProperty`（Vue 2）或 `Proxy`（Vue 3）
把每个字段都改造成"getter/setter"。**只有 return 里的字段被改造了。**

**你写代码时：所有要显示在界面上的数据，都必须放在 `data()` 里。**

## ⚠️ 坑 1：数组/对象的"新增属性"不响应

**Vue 2 的限制**：

```javascript
data() {
  return { user: { name: 'Tom' } }
},
methods: {
  addAge() {
    this.user.age = 20       // ❌ Vue 2 检测不到（新增属性）
  }
}
```

**为什么？** Vue 2 用 `Object.defineProperty` 只能监听**已存在的属性**，
新增的属性没有 getter/setter，Vue 不知道它变了。

**三种解决**：

```javascript
// ① Vue.set（推荐）
this.$set(this.user, 'age', 20)

// ② 整体替换（小对象可行）
this.user = { ...this.user, age: 20 }

// ③ 初始化时就声明
data() {
  return { user: { name: 'Tom', age: null } }    // 先占位
}
```

**⚠️ 但项目里用的是 Vue 2.6，所以你会遇到这个问题。**

**好消息**：**数组的 `push`/`pop`/`splice` 是响应式的**（Vue 重写了这 7 个方法）；

```javascript
this.list.push(newItem)      // ✅ 响应式
this.list[0] = newItem        // ❌ 不是（按索引赋值）
```

## ⚠️ 坑 2：`this` 丢失

```javascript
methods: {
  load() {
    // ❌ 普通函数：this 不是组件
    setTimeout(function() {
      this.list = data        // 报错：Cannot set property of undefined
    }, 100)

    // ✅ 箭头函数：this 是组件
    setTimeout(() => {
      this.list = data
    }, 100)
  }
}
```

**⭐ 这就是项目里到处用箭头函数的原因**（见第 1 章 1.3）。

## 项目里的响应式例子

```javascript
// src/views/contract/contractDraft/index.vue
getList() {
  this.loading = true                                            // ① 表格转圈
  listContractDraft(this.queryParams).then((response) => {
    this.contractDraftList = response.rows                        // ② 数据赋值
    this.total = response.total                                   // ③ 总数
    this.loading = false                                          // ④ 关转圈
  })
}
```

**对应模板**：

```html
<el-table v-loading="loading" :data="contractDraftList">
  <el-table-column prop="contractName" label="合同名" />
</el-table>

<pagination :total="total" v-show="total > 0" />
```

**四个变量一改，界面全部自动更新**：

| 变量 | 界面变化 |
|---|---|
| `loading` | 表格显示/隐藏遮罩 |
| `contractDraftList` | 表格行变化 |
| `total` | 分页组件显示"共 53 条" |

---

# 3.4 生命周期（组件从生到死）

## 什么是生命周期

一个组件会经历：**创建 → 挂载 → 更新 → 销毁**。
Vue 在每个阶段都提供了"钩子函数"，让你插代码。

**类比**：Java Bean 的 `@PostConstruct` / `@PreDestroy`。

## 八个钩子（只需记 4 个）

```
beforeCreate → created → beforeMount → mounted → beforeUpdate → updated
                                                  → beforeDestroy → destroyed
```

| 钩子 | 时机 | 能做什么 |
|---|---|---|
| `beforeCreate` | 实例刚创建，数据还没有 | 几乎不用 |
| **`created`** | **数据初始化完成，还没渲染到 DOM** | **发请求拿数据**（最常用） |
| `beforeMount` | 模板编译完，还没插入 DOM | 几乎不用 |
| **`mounted`** | **已渲染到浏览器 DOM** | **操作 DOM、初始化第三方库、拿 `$refs`** |
| `beforeUpdate` | 数据变了，DOM 还没更新 | 少用 |
| `updated` | DOM 更新完成 | 少用 |
| `beforeDestroy` | 销毁前 | **清理定时器、解绑事件**（重要） |
| `destroyed` | 销毁完成 | 少用 |

## ⭐ `created` vs `mounted`（面试常问）

| | `created` | `mounted` |
|---|---|---|
| DOM 存在吗 | ❌ 不存在 | ✅ 存在 |
| 能拿到 `$refs` 吗 | ❌ 不能（还没渲染） | ✅ 能 |
| 能操作 DOM 吗 | ❌ 不能 | ✅ 能 |
| 发请求 | ✅ **推荐**（早一点，快一点） | ✅ 也可以 |
| 初始化图表/ECharts | ❌ 不行（没有容器） | ✅ **必须在这** |

**⭐ 项目里的规律**：**发请求一律在 `created`；需要操作 DOM 的放 `mounted`。**

## 项目里的真实例子

```javascript
// src/views/contract/contractDraft/index.vue
created() {
  this.getList()                                  // ① 请求列表数据
  this.getContractType()                          // ② 请求合同类型（下拉框）

  // ③ 监听跨标签页消息
  this.refreshChannel = new BroadcastChannel('refresh-channel')
  this.refreshChannel.onmessage = (event) => {
    if (event.data === 'collateUpdate') {
      this.getList()
    }
  }

  // ④ 监听事件总线
  this.$EventBus.$on("reloadPage", (data) => {
    this.showApplyForm = false
    this.getList()
  })

  // ⑤ 如果 URL 带 openId，自动打开审批单
  if (this.$route.query.openId) {
    this.showApplyForm = true
    this.$nextTick(() => {
      this.$refs.applyForm.init(this.$route.query.openId)
    })
  }
},

beforeDestroy() {
  // ⑥ ★ 清理：关掉 BroadcastChannel，防止内存泄漏
  if (this.refreshChannel) {
    this.refreshChannel.close()
  }
}
```

## ⭐ 为什么 `beforeDestroy` 必须清理

**问题**：`BroadcastChannel`、`setInterval`、`addEventListener` 这些
**不会随着组件销毁自动消失**。

```javascript
// ❌ 不清理的后果
created() {
  setInterval(() => { this.getList() }, 5000)      // 每 5 秒请求一次
}
// 用户离开这个页面了，定时器还在跑，还在请求接口 → 内存泄漏 + 无用请求
```

```javascript
// ✅ 正确做法
data() {
  return { timer: null }
},
created() {
  this.timer = setInterval(() => { this.getList() }, 5000)
},
beforeDestroy() {
  clearInterval(this.timer)
  this.timer = null
}
```

**⭐ 面试常问："怎么避免内存泄漏？"** 答案就是：**在 `beforeDestroy` 清理所有外部注册**。

## `this.$nextTick()` 是什么

```javascript
if (this.$route.query.openId) {
  this.showApplyForm = true
  this.$nextTick(() => {
    this.$refs.applyForm.init(this.$route.query.openId)    // ← 等 DOM 更新后再拿 refs
  })
}
```

**问题**：改 `showApplyForm = true` 之后，DOM **不会立刻更新**
（Vue 是异步更新 DOM 的，为了批量优化）。

**所以立刻拿 `this.$refs.applyForm` 会是 `undefined`**（组件还没渲染）。

**`$nextTick(callback)`** = 等 DOM 更新完成后再执行 callback。

**类比**：`await Vue.nextTick()` ≈ Java 里 `SwingUtilities.invokeLater(...)`（等 UI 线程处理完）。

---

# 3.5 computed（计算属性）

## 痛点

```html
<!-- 想在模板里显示"全名" -->
<p>{{ user.firstName + ' ' + user.lastName }}</p>
```

**问题**：如果模板里很多地方用，要写很多遍；
而且每次重新渲染都会**重新计算**。

## 解决：computed

```javascript
computed: {
  fullName() {
    return this.user.firstName + ' ' + this.user.lastName
  }
}
```

```html
<p>{{ fullName }}</p>
<p>你好，{{ fullName }}</p>
```

## ⭐ 三个特点

### ① 用起来像属性（不带括号）

```html
{{ fullName }}        <!-- ✅ 不是 fullName() -->
```

### ② 有缓存（最重要）

**依赖不变时，直接用上次的结果，不重新计算。**

```javascript
computed: {
  expensiveResult() {
    console.log('计算了一次')          // 只会在依赖变化时打印
    return this.list.reduce(...)      // 耗时的计算
  }
}
```

**如果访问 10 次，但 `list` 没变 → 只计算 1 次。**

**对比 `methods`**：

| | `computed` | `methods` |
|---|---|---|
| 缓存 | ✅ 有 | ❌ 没有（每次都执行） |
| 用法 | `{{ fullName }}` | `{{ getFullName() }}` |
| 适合 | **派生数据** | **事件处理 / 有副作用** |

### ③ 依赖变了自动重算

```javascript
computed: {
  total() {
    return this.list.length          // 依赖 list
  }
}
// list 变了 → total 自动更新 → 界面自动更新
```

**你不需要手动"通知"它更新。**

## 项目里的真实例子

```javascript
// src/layout/components/Sidebar/index.vue
computed: {
  ...mapState(["settings"]),
  ...mapGetters(["userInfo"]),
  ...mapGetters(["sidebarRouters", "sidebar"]),

  activeMenu() {
    const route = this.$route
    const { meta, path } = route
    if (meta.activeMenu) {
      return meta.activeMenu
    }
    return path
  },

  isCollapse() {
    return !this.sidebar.opened
  }
}
```

**`activeMenu` 的作用**：侧边栏高亮当前菜单项。

```html
<el-menu :default-active="activeMenu">
```

**`isCollapse`**：侧边栏是否折叠（`!` 取反，因为 store 里叫 `opened`）。

## ⚠️ computed 里不能有副作用

```javascript
// ❌ 错误：computed 里发请求 / 改数据
computed: {
  list() {
    this.fetchData()          // 不该在这里发请求
    return this.data
  }
}

// ✅ 正确：computed 只做"计算"
computed: {
  total() {
    return this.list.length
  }
}
```

**规则：computed 必须是"纯函数"——相同输入永远相同输出，不能改外部状态。**

---

# 3.6 props（父传子）

## 痛点

一个"用户卡片"组件，想显示**不同的人**。
但组件里的 `data` 是写死的。

## 解决：props

**父组件传值，子组件接收。**

```vue
<!-- 子组件 UserCard.vue -->
<template>
  <div>
    <h3>{{ name }}</h3>
    <p>{{ age }} 岁</p>
  </div>
</template>

<script>
export default {
  props: {
    name: String,          // 声明接收一个叫 name 的字符串
    age: Number
  }
}
</script>
```

```vue
<!-- 父组件 -->
<template>
  <div>
    <UserCard name="张三" age="20" />
    <UserCard name="李四" age="30" />
  </div>
</template>

<script>
import UserCard from './UserCard.vue'
export default {
  components: { UserCard }      // 注册后才能用
}
</script>
```

## ⭐ 数据流向是单向的

```
父组件 ──props──► 子组件          ✅ 可以
子组件 ──改props──► 父组件        ❌ 不允许（会警告）
```

**为什么？** 如果子组件能直接改父组件的数据，多个子组件同时改就乱了。
**这叫"单向数据流"**，是 Vue 的核心理念。

**子组件想改怎么办？** 用 `$emit` 通知父组件，让父组件改（见 3.7）。

## 项目里的真实例子

```html
<!-- src/views/contract/contractDraft/index.vue -->
<el-table :data="contractDraftList" v-loading="loading">
```

**`<el-table>` 就是一个组件，`:data` 就是给它传 props。**

```html
<pagination
  :total="total"
  :page.sync="queryParams.pageNum"
  :limit.sync="queryParams.pageSize"
/>
```

| props | 含义 |
|---|---|
| `:total` | 总条数 |
| `:page.sync` | 当前页码（`.sync` 是双向绑定的语法糖） |
| `:limit.sync` | 每页条数 |

**⭐ `.sync` 修饰符**：让 props 可以"双向"——本质是"子组件 emit('update:page')，父组件自动更新"。

## props 的三种写法

```javascript
// ① 数组（简单，但不能校验类型）
props: ['name', 'age']

// ② 对象（推荐，能校验）
props: {
  name: String,
  age: {
    type: Number,
    default: 0,
    required: false
  }
}

// ③ 复杂校验
props: {
  age: {
    type: Number,
    validator: value => value >= 0 && value <= 150
  }
}
```

---

# 3.7 事件（子传父）

## 痛点

子组件想告诉父组件"用户点了确定按钮"，怎么通知？

## 解决：`$emit`

```vue
<!-- 子组件 ConfirmDialog.vue -->
<template>
  <div>
    <button @click="handleOk">确定</button>
  </div>
</template>

<script>
export default {
  methods: {
    handleOk() {
      this.$emit('ok', '这是传给父组件的数据')      // ★ 触发事件
    }
  }
}
</script>
```

```vue
<!-- 父组件 -->
<template>
  <ConfirmDialog @ok="onOk" />
</template>

<script>
export default {
  methods: {
    onOk(data) {
      console.log('子组件说:', data)
    }
  }
}
</script>
```

## 命名约定

| 位置 | 习惯 |
|---|---|
| 子组件 `$emit('ok')` | 小写 / kebab-case（`update-user`） |
| 父组件 `@ok` / `@update-user` | 和 emit 的名字对应 |

## 项目里的真实例子

```html
<!-- src/views/contract/contractDraft/index.vue 里用了 applyForm 子组件 -->
<applyForm ref="applyForm" @reload="refresh" />
```

```javascript
methods: {
  refresh() {
    this.showApplyForm = false
    this.handleQuery()          // 重新查询列表
  }
}
```

**含义**：`applyForm` 这个子组件在"提交成功"后 `$emit('reload')`，
父组件监听到就重新加载列表。

## ⭐ 三种通信方式对比

| 场景 | 方式 | 项目里的例子 |
|---|---|---|
| 父 → 子 | `props` | `<el-table :data="list">` |
| 子 → 父 | `$emit` | `<applyForm @reload="refresh">` |
| 跨层级/任意组件 | `$EventBus` / Vuex | `this.$EventBus.$emit('reloadPage')` |

**⭐ 项目里的 `$EventBus`**（`main.js:85` 创建）：

```javascript
Vue.prototype.$EventBus = new Vue()
```

**用法**：

```javascript
// A 组件发（不关心谁收）
this.$EventBus.$emit('reloadPage', { id: 1 })

// B 组件收（不关心谁发）
created() {
  this.$EventBus.$on('reloadPage', (data) => {
    this.getList()
  })
},
beforeDestroy() {
  this.$EventBus.$off('reloadPage')      // ★ 记得解绑！
}
```

**⚠️ 事件总线的问题**：

| 问题 | 说明 |
|---|---|
| 难追踪 | 不知道谁发的 |
| 易泄漏 | 忘了 `$off` 就一直在 |
| 难调试 | 数据流向混乱 |

**Vue 官方推荐用 Vuex 替代**。但项目里还在用（历史遗留）。

---

# 3.8 组件注册（全局 vs 局部）

## ⭐ 这个项目里注册了 13 个全局组件

```javascript
// src/main.js:93-106
Vue.component("DictTag", DictTag)
Vue.component('ZnModal', ZnModal)
Vue.component('ZnInfoPage', ZnInfoPage)
Vue.component("Pagination", Pagination)
Vue.component("RightToolbar", RightToolbar)
Vue.component("Editor", Editor)
Vue.component("FileUpload", FileUpload)
Vue.component("FileUploadLocal", FileUploadLocal)
Vue.component("FileUploadContract", FileUploadContract)
Vue.component("ImageUpload", ImageUpload)
Vue.component("ImageUploadLocal", ImageUploadLocal)
Vue.component("ImagePreview", ImagePreview)
Vue.component("JTable", JTable)
Vue.component("MoneyInput", MoneyInput)
```

**所以页面里可以直接写**：

```html
<pagination :total="total" />
<dict-tag :options="dicts" :value="status" />
<file-upload v-model="form.fileUrl" />
```

**不需要 import，不需要 `components: {}`。**

## 全局注册的原理

**`Vue.component()` 把组件登记到 `Vue.options.components`（构造函数上的静态属性）。**

**为什么所有页面都能查到？** 因为组件查找走**原型链**：

```
页面组件的 $options.components  →  原型链  →  Vue.options.components
        ↑ 找不到 Pagination                      ↑ 有 ✅
```

**⭐ 命名转换**：`Vue.component("Pagination", ...)` 注册后，
模板里可以写 `<Pagination />` 或 `<pagination />`（Vue 会自动做 kebab-case 转换）。

**HTML 不区分大小写**，所以习惯写 kebab-case。

## 局部注册（更推荐的方式）

```vue
<script>
import Pagination from '@/components/Pagination'
import MyDialog from './MyDialog.vue'

export default {
  components: { Pagination, MyDialog }     // ★ 注册
}
</script>
```

**⭐ 两种方式对比**：

| | 全局注册 | 局部注册 |
|---|---|---|
| 写法 | `Vue.component()` 一次 | 每个页面 `components: {}` |
| 作用范围 | **所有组件** | 仅当前组件 |
| 打包 | ⚠️ **无法 tree-shaking**，全进 bundle | ✅ 按需打包 |
| 可追溯 | ⚠️ 看页面代码不知道从哪来 | ✅ 一眼看到 import |
| 适合 | **几乎所有页面都用**（分页、字典标签） | **只在少数页面用** |

## ⭐ 全局注册的代价（面试重点）

**Vue 官方文档明确警告**：

> 全局注册的组件**无法被 tree-shaking 优化**。
> 即使某个组件从未被使用，它仍然会被打包进最终的 bundle 里。

**为什么？** 因为 `Vue.component("X", X)` 是一句**有副作用的运行时代码**，
打包工具（webpack/rollup）无法静态判断"这个组件到底用没用"，所以**不敢删**。

**这个项目里的实际情况**：

| 组件 | 到处都用吗 | 全局注册合理吗 |
|---|---|---|
| `Pagination` | ✅ 几乎每个列表页 | 合理 |
| `DictTag` | ✅ 大量使用 | 合理 |
| `FileUploadContract` | ⚠️ 只在合同相关页面 | 其实该局部注册 |
| `ZnModal` / `ZnInfoPage` | ⚠️ 少数页面 | 其实该局部注册 |

**所以首屏 bundle 会包含一些用不到的组件** —— 这是"管理后台场景"下的合理权衡
（用户一天开一次，多几百 KB 换开发便利，划算）。

**如果是 C 端首屏敏感的场景，应该全部改成局部注册 + 按需加载。**

---

# 3.9 完整例子：手写一个列表页

**把本章学的都用上。** 这个例子和项目里 `contractDraft/index.vue` 结构一致。

```vue
<template>
  <div class="app-container">
    <!-- ① 搜索区（用 v-show 控制折叠） -->
    <el-form
      v-show="showSearch"
      ref="queryForm"
      :model="queryParams"
      size="small"
      :inline="true"
    >
      <el-form-item label="姓名" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入姓名"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- ② 工具栏 -->
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          v-hasPermi="['system:user:add']"
          type="primary"
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
        >新增</el-button>
      </el-col>
    </el-row>

    <!-- ③ 表格（用 v-loading 和响应式数据） -->
    <el-table v-loading="loading" :data="userList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="姓名" prop="name" :show-overflow-tooltip="true" />
      <el-table-column label="年龄" prop="age" />
      <el-table-column label="状态" prop="status">
        <template slot-scope="scope">
          <!-- ④ v-if 条件渲染 -->
          <el-tag v-if="scope.row.status === 1" type="success">启用</el-tag>
          <el-tag v-else type="danger">停用</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            v-hasPermi="['system:user:edit']"
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
          >修改</el-button>
          <el-button
            v-hasPermi="['system:user:remove']"
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- ⑤ 分页（全局组件，直接写标签） -->
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />
  </div>
</template>

<script>
// ⑥ 局部导入：api 函数 + 自定义组件
import { listUser, delUser } from '@/api/system/user'
import UserDialog from './UserDialog.vue'

export default {
  name: 'UserList',                            // 组件名（keep-alive 缓存靠它）
  components: { UserDialog },                   // 局部注册

  // ⑦ data：所有响应式数据
  data() {
    return {
      loading: true,                            // 表格转圈
      showSearch: true,                         // 搜索区显示
      userList: [],                             // 列表数据
      total: 0,                                 // 总条数
      ids: [],                                  // 选中的 id
      single: true,                             // 非单个禁用
      multiple: true,                           // 非多个禁用
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        name: undefined,
        status: undefined
      }
    }
  },

  // ⑧ 生命周期：创建后立即拿数据
  created() {
    this.getList()
  },

  // ⑨ computed：派生数据
  computed: {
    hasSelection() {
      return this.ids.length > 0
    }
  },

  // ⑩ methods：所有方法
  methods: {
    getList() {
      this.loading = true
      listUser(this.queryParams).then(response => {
        this.userList = response.rows
        this.total = response.total
        this.loading = false
      })
    },

    handleQuery() {
      this.queryParams.pageNum = 1
      this.getList()
    },

    resetQuery() {
      this.resetForm('queryForm')
      this.handleQuery()
    },

    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length !== 1
      this.multiple = !selection.length
    },

    handleAdd() {
      this.$refs.dialog.add()
    },

    handleUpdate(row) {
      this.$refs.dialog.edit(row.id)
    },

    handleDelete(row) {
      const ids = row.id || this.ids
      this.$modal.confirm('是否确认删除？').then(() => {
        this.loading = true
        return delUser(ids)
      }).then(() => {
        this.getList()
        this.$modal.msgSuccess('删除成功')
      }).finally(() => {
        this.loading = false
      })
    }
  }
}
</script>
```

## 这个例子里用到了本章所有概念

| 概念 | 在哪 |
|---|---|
| 组件三段式 | 整个文件 |
| 模板语法（插值/绑定/事件/双向/条件/循环） | `<template>` 里 |
| 响应式 | `loading` / `userList` / `total` 改了界面自动变 |
| 生命周期 | `created() { this.getList() }` |
| computed | `hasSelection` |
| 局部注册 | `components: { UserDialog }` |
| 全局组件 | `<pagination />`（没 import） |

## ⭐ 几个新手常问的点

### `slot-scope="scope"` 是什么

```html
<el-table-column label="状态" prop="status">
  <template slot-scope="scope">
    <el-tag v-if="scope.row.status === 1">启用</el-tag>
  </template>
</el-table-column>
```

**`scope.row` 就是"当前这一行的数据"。**

`el-table` 渲染每一行时会把这个行数据通过"作用域插槽"传出来，
`slot-scope="scope"` 就是接住它。

**Vue 2.6+ 的新写法**：

```html
<template v-slot:default="scope">
  <!-- 或简写 -->
</template>
```

### `this.$refs.dialog.add()` 是什么

```html
<UserDialog ref="dialog" />
```

```javascript
this.$refs.dialog.add()          // 调用子组件的方法
```

**`$refs` 让你能直接调子组件的方法**（虽然不推荐，但项目里大量使用）。

**⚠️ 注意**：`$refs` 在 `mounted` 之前拿不到（DOM 还没渲染）。

### `this.$modal` / `this.resetForm` 是什么

**这两个是 `main.js` 里挂到全局的工具**：

```javascript
// src/main.js:77
Vue.prototype.resetForm = resetForm

// src/projects/plugins 里注册的
Vue.prototype.$modal = { confirm, msgSuccess, msgError, ... }
```

**所以任何组件里都能用 `this.$modal.confirm(...)`。**

---

# 本章自测

1. `.vue` 文件的三个块分别是什么？`scoped` 有什么用？
2. `v-if` 和 `v-show` 怎么选？
3. `v-for` 为什么必须加 `:key`？为什么不能把 index 当 key？
4. 什么样的数据是"响应式"的？
5. Vue 2 里给对象**新增属性**为什么界面不更新？怎么解决？
6. `created` 和 `mounted` 的关键区别？
7. `beforeDestroy` 里要做什么？
8. `computed` 和 `methods` 的三个区别？
9. `props` 的数据流方向？子组件能改 props 吗？
10. 子组件怎么通知父组件？
11. 全局注册组件和局部注册，各有什么代价？
12. `this.$nextTick()` 什么时候必须用？
13. `$refs` 什么时候才能拿到？
14. `slot-scope="scope"` 里的 `scope.row` 是什么？

<details>
<summary>答案</summary>

1. `<template>` 结构、`<script>` 逻辑、`<style scoped>` 样式；`scoped` 让样式只作用于当前组件（编译时加唯一属性）
2. 条件**很少变**用 `v-if`（销毁/重建 DOM）；**频繁切换**用 `v-show`（只改 `display`）
3. `key` 是 Vue 识别元素身份的标记。用 index 当 key 时，删除中间元素会导致 Vue 误判"内容变了"而非"删除了"，复用错误的 DOM
4. 只有 `data()` **return 里**的字段才是响应式（Vue 会给它们加 getter/setter）
5. Vue 2 用 `Object.defineProperty` 只能监听**已存在**的属性。解决：`this.$set(obj, key, val)` 或整体替换 `this.obj = {...this.obj, key: val}`
6. `created` 时 DOM **还没渲染**（拿不到 `$refs`）；`mounted` 时 DOM 已渲染（能操作 DOM）
7. 清理所有外部注册：定时器（`clearInterval`）、事件监听（`$off`、`removeEventListener`）、`BroadcastChannel.close()`，防内存泄漏
8. ① `computed` 有缓存，`methods` 没有；② `computed` 用起来像属性（不带括号）；③ `computed` 适合派生数据，`methods` 适合事件处理
9. **单向**：父 → 子。子组件**不能**直接改 props（会警告）
10. `this.$emit('事件名', 数据)`，父组件用 `@事件名="处理函数"` 接收
11. 全局：**无法 tree-shaking**，用不到的也进 bundle；局部：每个页面要写 import + `components: {}`，但能按需打包
12. 改了数据后**立刻要操作 DOM** 时必须用（因为 Vue 异步更新 DOM）
13. `mounted` 之后（`created` 里拿不到，DOM 还没渲染）
14. **当前这一行的数据对象**（`el-table` 通过作用域插槽把行数据传出来）
</details>

---

# 下一步

👉 **[第 4 章：项目骨架](04-项目骨架.md)**

Vue 概念学完了。下一章看**这个项目是怎么把这些概念组织起来的**：
入口文件、目录结构、路由表、Layout 外壳。
