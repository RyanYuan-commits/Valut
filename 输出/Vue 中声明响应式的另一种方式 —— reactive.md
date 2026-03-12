---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
aliases:
  - reactive
---
---

**关键词**: Vue, 响应式, reactive

---

## 1	reactive 原理与特性

`reactive()` API  接收一个 JavaScript 对象, 返回一个 [JavaScript Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy), JavaScript Proxy 的行为和普通对象一致, 所以不需要像 [[在 Vue 中使用 ref 函数声明响应式状态|ref]] 那样使用 `.value` 来进行访问. 其对于对象的转化是 **深层的**, 对象中的嵌套对象也会被 `reactive()` 包装.

```vue
<script setup>

import { reactive } from 'vue'

const state = reactive({ count: 0 })

</script>


<template>

<button @click="state.count++">
  {{ state.count }}
</button>

</template>
```

特别的, 当 [[在 Vue 中使用 ref 函数声明响应式状态|ref]] 的值是一个对象时, [[在 Vue 中使用 ref 函数声明响应式状态|ref]] 在内部也会调用 `reactive()`, 与浅层  [[在 Vue 中使用 ref 函数声明响应式状态|ref]] 类似, `reactive()` 也提供了一个 `shallowRective()` 以退出深层响应式.

## 2	响应式对象 ≠ 原始对象

`reactive()` 返回的是一个对象的代理, 它和原始对象是不相等的, 修改原始对象并不会触发响应式更新.

```js
const raw = {}
const proxy = reactive(raw)

// 代理对象和原始对象不是全等的
console.log(proxy === raw) // false
```

这个规则对嵌套对象也适用, 响应式对象内的嵌套对象仍然是代理:

```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw

console.log(proxy.nested === raw) // false
```

## 3	reactive 的局限性

**仅能使用在对象类型**: `reactive()` 仅作用于对象类型, 即对象, 数组和集合类型, 不能处理如 string, number, boolean 这样的原始类型.

**不能替换对象**: 由于 Vue 的响应式跟踪是通过属性访问实现的, 因此我们必须始终保持对响应式对象的相同引用. 这意味着我们不能轻易地"替换"响应式对象, 因为这样的话与第一个引用的响应性连接将丢失.

```js
let state = reactive({ count: 0 })

// 上面的 ({ count: 0 }) 引用将不再被追踪
// (响应性连接已丢失！)
state = reactive({ count: 1 })
```

**对解构不友好**: 当我们将响应式对象的原始类型属性解构为本地变量时, 或者将该属性传递给函数时, 我们将丢失响应性连接.

```js
const state = reactive({ count: 0 })

// 当解构时，count 已经与 state.count 断开连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收到的是一个普通的数字
// 并且无法追踪 state.count 的变化
// 我们必须传入整个对象以保持响应性
callSomeFunction(state.count)
```

由于这些限制, 建议使用 [[在 Vue 中使用 ref 函数声明响应式状态|ref]] 作为声明响应式状态的主要 API.

---