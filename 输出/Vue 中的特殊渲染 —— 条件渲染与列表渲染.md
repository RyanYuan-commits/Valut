---
type: permanent
aliases:
  - v-for
  - v-if
  - v-show
banner: Assets/Banner/pexels-bertellifotografia-1144690.jpg
---
---

**关键词**: Vue, 条件渲染

---

## 1	条件渲染

### 1.1	v-if 指令

条件渲染是编程中一种**根据特定条件**动态决定用户界面是否显示, 如何显示或渲染哪些内容的重要机制.

`v-if` 指令用于条件性的渲染一块内容, 这块内容只会在指令表达式返回真值时才会被**渲染**. Vue 也提供了 `v-else` 和 `v-else-if` 指令来提供完善的条件控制. 如果想一次控制多个元素的渲染, 可以将这些元素包裹在 `<template>` 标签中, 这是一个**不可见的包装器元素**, 渲染结果不会包含 `<template>` 元素.

```vue
<h1 v-if="awesome">Vue is awesome!</h1>
```

### 1.2	v-show 指令

另一个提供条件渲染能力的指令是 `v-show`, 用法与 `v-if` 基本一致: 不同之处在于 `v-show` 会在**渲染**中**保留**该元素, 其控制显隐的方式是修改元素的 `display` 属性.

```vue
<h1 v-show="ok">Hello!</h1>
```

### 1.3	两者之间的对比

v-if 是“真实的”按条件渲染, 因为它确保了在切换时, 条件区块内的事件监听器和子组件都会被销毁与重建. v-if 也是惰性的: 如果在初次渲染时条件值为 false, 则不会做任何事. 条件区块只有当条件首次变为 true 时才被渲染. 相比之下, v-show 简单许多, 元素无论初始条件如何, 始终会被渲染, 只有 CSS display 属性会被切换.

总的来说, v-if 有更高的切换开销, 而 v-show 有更高的初始渲染开销. 因此, **如果需要频繁切换**, 则使用 v-show 较好;如果**在运行时绑定条件很少改变**, 则 v-if 会更合适.

## 2	列表渲染指令 v-for

### 2.1	遍历数组

可以使用 `v-for` 指令基于一个数组来渲染一个列表, 指令格式为 `v-for="item in items"`, `v-for` 块内部可以完整的访问父作用域内的不属性和变量.

```js
const parentMessage = ref('Parent')
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
```

```vue
<li v-for="(item, index) in items">
  {{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

### 2.2	遍历对象属性

可以使用 `v-for` 来遍历一个对象的所有属性, 遍历的内容为 `Object.values()` 的返回值. 可以提供三个遍历参数, 分别表示属性值, 属性名和索引.

```vue
<ul>
  <li v-for="value in myObject">
    {{ value }}
  </li>
</ul>

<li v-for="(value, key) in myObject">
  {{ key }}: {{ value }}
</li>

<li v-for="(value, key, index) in myObject">
  {{ index }}. {{ key }}: {{ value }}
</li>
```

### 2.3	通过 key 管理状态

Vue 使用数据项在数组中的位置来标识每个元素, 当 id-1 与 id-2 互换了位置时, 对于位于位置 1 的 DOM, Vue 会将 id-2 的内容更新到这个 DOM 上, 也就是 "就地更新" 的方式, 这种方式更加高效, 但是当元素中带有输入框, 选择框这样的临时状态时, 采取这种方式会导致状态错位.

所以, 需要提供一个对于该位置的唯一标识给 Vue, 防止其错误的复用当前的 DOM 元素, 从而正确的重用和重新排序元素. 提供的方式是 `key` 属性.

```vue
<template v-for="todo in todos" :key="todo.name">
  <li>{{ todo.name }}</li>
</template>
```

---

[官方文档-条件渲染](https://cn.vuejs.org/guide/essentials/conditional.html)

[官方文档-列表渲染](https://cn.vuejs.org/guide/essentials/list.html)
