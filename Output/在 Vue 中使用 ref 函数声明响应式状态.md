---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
aliases:
  - ref
  - Ref
---
---

**关键词**: Vue, 响应式

---

## 1	ref 函数介绍

在组合式 API 中, 推荐使用 [ref](https://cn.vuejs.org/api/reactivity-core#ref) 函数来声明响应式状态, `ref` 接收一个 JavaScript 对象, 将其封装成一个带有 `.value` 属性的响应式对象后返回.

```js
import { ref } from 'vue'

const count = ref(0)
```

要在模板中使用响应式对象, 需要在 `setup` 函数中声明并返回它们:

```js
import { ref } from 'vue'

export default {
  // `setup` 是一个特殊的钩子，专门用于组合式 API。
  setup() {
    const count = ref(0)

    // 将 ref 暴露给模板
    return {
      count
    }
  }
}
```

可以使用 `<script setup>` 语法糖来简化上面的操作, 在 `<script setup>` 中导入或声明 的 变量和函数可以在模板中直接使用.

## 2	ref 的深层响应性

`ref` 可以持有任何类型的值, 包括深层嵌套的对象, 数组或者 JavaScript 内置的数据结构, 比如 Map. ref 会使它的值具有深层响应性. 这意味着即使改变嵌套对象或数组时, 变化也会被检测到:

```js
import { ref } from 'vue'

const obj = ref({
 nested: { count: 0 },
 arr: ['foo', 'bar']
})

function mutateDeeply() {
 // 以下都会按照期望工作
 obj. value. nested. count++
 obj. value. arr. push('baz')
}
```

非原始值将通过 `reactive()` 转换为响应式代理, 也可以通过 `shallow ref` 来放弃深层响应性. 对于浅层 `ref`, 只有 `.value` 的访问会被追踪. 浅层 `ref` 可以用于避免对大型数据的响应性开销来优化性能, 或者有外部库管理其内部状态的情况.

## 3	DOM 更新时机

在 Vue 中, DOM 的更新并不是实时的, 而是会在 "next tick" 周期中缓冲所有状态的修改, 保证不管修改了多少次, 最终都会作为一次更新.

如果需要等待 DOM 更新完成后再执行某些代码, 可以使用 [nextTick()](https://cn.vuejs.org/api/general.html#nexttick) 全局 API:

```js
import { nextTick } from 'vue'

async function increment() {
  count.value++
  await nextTick()
  // 现在 DOM 已经更新了
}
```

---

[官方文档 - 响应式基础](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html)