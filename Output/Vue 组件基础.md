---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: Vue, 组件

---

## 1	组件介绍

组件允许我们将 UI 划分为独立的, 可重用的部分, 和嵌套 HTML 元素类似, 组件常常被组织成一个层层嵌套的树状结构.

![[Vue 组件.png|600]]
## 2	定义一个组件

使用构建步骤时候, 组件通过单 `.vue` 文件定义:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>
```

不使用构建步骤时, 组件以包含 Vue 特定选项的 JavaScript 对象来定义:

```js
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return { count }
  },
  template: `
    <button @click="count++">
      You clicked me {{ count }} times.
    </button>`
  // 也可以针对一个 DOM 内联模板：
  // template: '#my-template-element'
}
```

## 3	使用组件

```vue
<script setup>
import ButtonCounter from './ButtonCounter.vue'
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

## 4	传递 props

**场景**: 构建一个博客网站, 需要一个表示博客文章的组件, 我们可能希望所有的博客文章共享同一套布局, 需要向组件中传递数据.

props 是一种特别的 Attributes, 使用前需要现在组件中定义该 prop:

```vue
<script setup>
const props = defineProps(['title'])
console.log(props.title)
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

在父组件中k

---