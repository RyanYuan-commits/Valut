---
type: permanent
aliases:
  - v-for
  - v-if
  - v-show
banner: Assets/Banner/pexels-bertellifotografia-1144690.jpg
content-start: 461
banner-x: 51
banner-y: 89
---
# 🌐 核心观点



---

# 🔖 详细解释

## 1	条件渲染

条件渲染是编程中一种根据特定条件动态决定用户界面是否显示, 如何显示或渲染哪些内容的重要机制.

### 1.1	v-if 指令

`v-if` 指令用于条件性的渲染一块内容, 这块内容只会在指令表达式返回真值时才会被渲染.

```vue
<h1 v-if="awesome">Vue is awesome!</h1>
```

Vue 也提供了 `v-else` 和 `v-else-if` 指令来提供完善的条件控制.

```vue
<button @click="awesome = !awesome">Toggle</button>

<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>

<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

如果想一次控制多个元素的渲染, 可以将这些元素包裹在 `<template>` 标签中, 这是一个**不可见的包装器元素**, 渲染结果不会包含 `<template>` 元素.

### 1.2	v-show 指令

另一个提供条件渲染能力的指令是 `v-show`, 用法与 `v-if` 基本一致:

```vue
<h1 v-show="ok">Hello!</h1>
```

不同之处在于 `v-show` 会在渲染中保留该元素, 其控制显隐的方式是修改元素的 `display` 属性.

### 1.3	两者之间的对比

v-if 是“真实的”按条件渲染, 因为它确保了在切换时, 条件区块内的事件监听器和子组件都会被销毁与重建. v-if 也是惰性的: 如果在初次渲染时条件值为 false, 则不会做任何事. 条件区块只有当条件首次变为 true 时才被渲染.

相比之下, v-show 简单许多, 元素无论初始条件如何, 始终会被渲染, 只有 CSS display 属性会被切换.

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

可以使用 `v-for` 来遍历一个对象的所有属性, 遍历的内容为 `Object.values()` 的返回值.

```js
const myObject = reactive({ 
	title: 'How to do lists in Vue', 
	author: 'Jane Doe', 
	publishedAt: '2016-04-10' 
})
```

可以提供三个遍历参数, 分别表示属性值, 属性名和索引.

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

### 2.3	使用范围值

```vue
<span v-for="n in 10">{{ n }}</span>
```

可以接收一个整数值, 输出 `[1, n]` 的序列, 注意是从 1 开始的.

### 2.4	v-if 与 v-for 的优先级问题

当 `v-if` 指令与 `v-for` 指令作用于同一个元素上时, `v-if` 指令的优先级更高, 这就导致了在 `v-if` 表达式中, 无法访问 `v-for` 作用域的元素:

```vue
<!-- 异常 -->
<li v-for="todo in todos" v-if="!todo.isComplete">
	{{ todo.name }} 
</li>
```

此时可以使用 `<template>` 来解决这个问题:

```vue
<template v-for="todo in todos">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

### 2.5	通过 key 管理状态

Vue 使用数据项在数组中的位置来标识每个元素, 当 id-1 与 id-2 互换了位置时, 对于位于位置 1 的 DOM, Vue 会将 id-2 的内容更新到这个 DOM 上, 也就是 "就地更新" 的方式, 这种方式更加高效, 但是当元素中带有输入框, 选择框这样的临时状态时, 采取这种方式会导致状态错位.

所以, 需要提供一个对于该位置的唯一标识给 Vue, 防止其错误的复用当前的 DOM 元素, 从而正确的重用和重新排序元素. 提供的方式是 `key` 属性.

```vue
<template v-for="todo in todos" :key="todo.name">
  <li>{{ todo.name }}</li>
</template>
```

### 2.6	数组变化侦测

Vue 能够侦听响应式数组的变更方法, 在其发生变化时触发相关更新, 这些方法包括:

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

当触发不可变方法时, 如 `filter()`, `concat()`, `slice()` 时, 需要将旧数组替换为新的, Vue 会最大化对已有 DOM 元素的重用, 以优化性能:

```js
// `items` 是一个数组的 ref
items.value = items.value.filter((item) => item.message.match(/Foo/))
```

### 2.7	展示过滤或排序后的结果

可以使用计算属性或函数的方式来展示过滤或排序后的结果:

```js
// 使用计算属性
const numbers = ref([1, 2, 3, 4, 5])

const evenNumbers = computed(() => {
  return numbers.value.filter((n) => n % 2 === 0)
})

`<li v-for="n in evenNumbers">{{ n }}</li>`

// 使用方法
const sets = ref([
  [1, 2, 3, 4, 5],
  [6, 7, 8, 9, 10]
])

function even(numbers) {
  return numbers.filter((number) => number % 2 === 0)
}

`<ul v-for="numbers in sets"> <li v-for="n in even(numbers)"> {{ n }} </li> </ul>`
```

---

# 📚 参考内容

[官方文档-条件渲染](https://cn.vuejs.org/guide/essentials/conditional.html)

[官方文档-列表渲染](https://cn.vuejs.org/guide/essentials/list.html)
