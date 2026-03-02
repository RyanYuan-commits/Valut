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



---