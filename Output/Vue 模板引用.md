---
type: permanent
banner: Assets/Banner/pexels-8kspain-21564213.jpg
---
---

**关键词**: Vue, ref

---

适用于需要访问底层 DOM 元素的场景, 可以使用特殊的 ref 属性:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 声明一个 ref 来存放该元素的引用
// 必须和模板里的 ref 同名
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

模板引用也可以被用在一个子组件上, 这种情况下引用中获得的值是组件实例.

可以在 v-for  中使用, 在挂载完成后获取到的是一个数组, 但是顺序不保证和 v-for 中的数据相同.

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

---