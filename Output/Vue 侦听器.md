---
type: permanent
banner: Assets/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
---

**关键词**: vue, watch

---

## 1	基本示例

`watch()` 的第一个参数可以是不同形式的 "数据源": 一个 ref, 一个响应式对象, 一个 getter 函数, 多个数据源组成的数组.

```vue
<script setup>
import { ref, watch } from 'vue'

const question = ref('你是男生吗?')
const answer  = ref('yes')

watch(question, async (newQuestion) => {
  answer.value = ''
  answer.value = '思考中'
  try {
    const res = await fetch('https://yesno.wtf/api')
    answer.value = (await res.json()).answer
  } catch (error) {
    answer.value = '访问 API 异常' + error
  }
})
</script>

<template>
  <p>
    询问一个可以用 yes 或 no 回答的问题:
    <input v-model="question" />
  </p>
  <p>{{ answer }}</p>
</template>
```

## 2	特殊场景

### 2.1	创建后立刻执行一次回调

`watch()` 默认是懒执行的, 当监听的数据第一次发生变化时, 回调才会执行, 可以通过 `immediate` 设置立刻执行.

```js
// 立即执行，且当 source 改变时再次执行
watch(source, (newValue, oldValue) => {}, { immediate: true })
```

### 2.2	一次性侦听器

```js
// 当 source 变化时, 仅触发一次.
watch(source, (newValue, oldValue) => {}, { once: true })
```

## 3	watchEffect

watchEffect 会在副作用发生期间**追踪依赖**. 它会在同步执行过程中, **自动追踪**所有能访问到的**响应式属性**. 这更方便, 而且代码往往更简洁, 但有时其响应性依赖关系会不那么明确.

```js
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  data.value = await response.json()
})
```

---