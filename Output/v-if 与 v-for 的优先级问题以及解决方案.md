---
type: permanent
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
---
---

**关键词**: Vue, 条件渲染, 列表渲染 

---

当 `v-if` 指令与 `v-for` 指令作用于同一个元素上时, `v-if` 指令的优先级更高, 这就导致了在 `v-if` 表达式中, 无法访问 `v-for` 作用域的元素:

```vue
<!-- 异常 -->
<li v-for="todo in todos" v-if="!todo.isComplete">
	{{ todo.name }} 
</li>
```

可以使用 `<template>` 来解决:

```vue
<template v-for="todo in todos">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

---