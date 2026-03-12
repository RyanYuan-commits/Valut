---
type: permanent
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
banner-x: 51
banner-y: 70
---
# 🌐 核心观点

Vue 专门为 `class` 和 `style` 的 `v-bind` 用法做了特殊的功能增强, 除了字符串以外, 表达式的值也可以是对象或数组.

---

# 🔖 详细解释

## 1	绑定 HTML class

### 1.1	绑定对象

```vue
<template>
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
</template>

<script>
const isActive = ref(true)
const hasError = ref(false)
</script>
```

可以给 `:class` 传递一个对象来动态切换 `class`, 最终的渲染结果会是:

```html
<div class="static active"></div>
```

### 1.2	绑定数组

可以给 `:class` 绑定一个数组来渲染多个 CSS class:

```vue
<template>
<div :class="[activeClass, errorClass]"></div>
</template>

<script>
<div :class="[isActive ? activeClass : '', errorClass]"></div>
</script>
```

## 2	绑定内联样式

`:style` 支持绑定 JavaScript 对象值, 对应的是 HTML 元素的 `style` 属性.

```vue
<template>
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
</template>

<script>
const activeColor = ref('red') const fontSize = ref(30)
</script>
```


---

# 📚 参考内容

