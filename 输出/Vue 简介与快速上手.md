---
type: permanent
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
---
---

**关键词**: Vue

---

## 1	什么是 Vue?

Vue 是用于构建用户界面的 JavaScript 界面, 基于标准的 HTML, CSS 和 JavaScript 构建, 并提供了一套[[命令范式 —— 命令式 vs 声明式|声明式]]的, **组件化**的编程模型.

```js
import { createApp, ref } from 'vue'

createApp({
  setup() {
    return {
      count: ref(0)
    }
  }
}).mount('#app')
```

```html
<div id="app">
  <button @click="count++">
    Count is: {{ count }}
  </button>
</div>
```

上面的示例展示了 Vue 的两个核心功能:

- **[[命令范式 —— 命令式 vs 声明式|声明式]]渲染**: Vue **基于标准 HTML 拓展了一套模板语法**, 使得我们可以[[命令范式 —— 命令式 vs 声明式|声明式]]地描述最终输出的 HTML 和 JavaScript 状态之间的关系.
	
- **响应性**: Vue 会自动跟踪 JavaScript 状态并在其发生变化时**响应式**地更新 DOM.

## 2	渐进式框架

Vue 注重**灵活和可逐步集成**的特点, 可以在以下场景使用:

- 无需构建步骤, 渐进式增强静态的 HTML
	
- 在任何页面中作为 Web Components 嵌入
	
- 单页应用 (SPA)
	
- 全栈 / 服务端渲染 (SSR)
	
- Jamstack / 静态站点生成 (SSG)
	
- 开发桌面端, 移动端, WebGL, 甚至是命令行终端中的界面

## 3	单文件组件

在启用了构建工具的 Vue 项目中, 可以使用一种类似 HTML 格式的方式来书写 Vue 组件. 它被称为单文件组件 (Single-File Components, SFC), 将 HTML, CSS 和 JavaScript 封装在一个文件中.

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

## 4	API 风格

### 4.1	选项式 API

用包含多个选项的对象来描述组件逻辑, 如 `data`, `method`. 定义的选项都会暴露在 `this` 中.

```vue
<script>
export default {
  // data() 返回的属性将会成为响应式的状态
  // 并且暴露在 `this` 上
  data() {
    return {
      count: 0
    }
  },

  // methods 是一些用来更改状态与触发更新的函数
  // 它们可以在模板中作为事件处理器绑定
  methods: {
    increment() {
      this.count++
    }
  },

  // 生命周期钩子会在组件生命周期的各个不同阶段被调用
  // 例如这个函数就会在组件挂载完成后被调用
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### 4.2	组合式 API

使用导入的 API 函数来描述组件的数据和行为, 组合式 API 通常与 `<script setup>` 搭配使用, 用以简化配置.

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

## 5	快速上手

### 5.1	创建 Vue 应用

```bash
npm create vue@latest

pnpm create vue@latest

yarn create vue@latest

bun create vue@latest
```

### 5.2	通过 CDN 使用 Vue

可以直接借助 Script 标签来导入 Vue 的 js 文件:

```js
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

---