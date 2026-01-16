---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
# 🌐 核心观点

Vue 使用一种基于 HTML 的模板语法, 可以通过声明式的方式将数据绑定到特定的 DOM 元素上.

---

# 🔖 详细解释

## 1	解释为纯文本

最基本的数据绑定形式是 "文本插值", 使用方式为:

```vue
<span> Message: {{message}} </span>
```

这种方式会将数据解析为**纯文本**, 如果希望数据被当作 HTML 元素处理, 需要使用 `v-html` 指令.

```vue
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

## 2	将数据绑定到 HTML 属性

### 2.1	v-bind 指令与其简写形式

双大括号的方式无法作用于 HTML Attribute, 想要响应式的绑定一个 Attribute, 应该使用 `v-bind` 指令:

```vue
<div v-bind:id="dynamicId"></div>
```

因为 `v-bind` 经常使用, Vue 为其提供了以下简写形式:

```vue
<div :id="dynamicId"></div>

<!-- 与 :id="id" 相同 -->
<div :id></div>
<div v-bind:id></div>
```

### 2.2	布尔型 Attribute

在 HTML Attribute 中有一类特殊的 "布尔属性", 用该属性**是否存在**来表示 `true` 或者 `false`, 常见的有 `required`, `readonly`, `disabled`.

Vue 的 `v-bind` 在这种场景下的行为略有不同:

```vue
<button :disabled="isButtonDisabled">Button</button>
```

当 `isButtonDisabled` 为真值或一个空字符串 (即 `<button disabled="">`) 时, 元素会包含这个 `disabled` attribute. 而当其为其他假值时 attribute 将被忽略.

### 2.3	动态绑定多个值

可以通过 `v-bind` 指令将一个包含多个 Attribute 的 JavaScript 对象绑定到某个元素上:

```vue
<script>
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}
</script>

<template>
<div v-bind="objectOfAttrs"></div>
</template>
```

## 3	在模板中使用 JavaScript 表达式

Vue 所有的数据绑定都支持使用表达式的形式 (不支持语句), 表达式会作为 JavaScript 代码, 以当前实例为作用域执行.

```vue
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>

<time :title="toTitleDate(date)" :datetime="date"> {{ formatDate(date) }} </time>
```

这些表达式是在一个受隔离的环境中执行的, 模板中的变量查找, 会按照 当前实例 -> [白名单内容](https://github.com/vuejs/core/blob/main/packages/shared/src/globalsAllowList.ts#L3) 的顺序查找

## 4	指令 Directives

Vue 提供了很多类似 v-bind, v-html 这样的内置指令, 指令的期望值是一个 JavaScript 表达式, **指令会在表达式的值变化时响应式的更新 DOM 元素**;

有几个特殊指令的入参不是表达式: `v-for`, `v-on`, `v-slot`;

### 4.1	指令参数

某些指令可能需要带一个 "参数", 跟在指令的后面, 用 `:` 隔开, 如 `v-bind:href="url`, 用参数指定要**响应式**更新的 HTML 参数, 还有 `v-on:click="doSomething"`.

同样, 在指令参数上也可以使用 JavaScript 表达式, 需要包裹在 `[ ]` 中.

```vue
<a v-on:[eventName]="doSomething"> ... </a>

<!-- 简写 -->
<a @[eventName]="doSomething"> ... </a>
```

当 `eventName` 为 `"foucs"` 时, 上面的指令就等价于 `v-on:foucs`.

### 4.2	修饰符

修饰符是以 `.` 开头的特殊后缀, 表明指令需要以一些特殊方式被绑定, 例如:

```vue
<form @submit.prevent="onSubmit">...</form>
```

表明 `v-on` 指令对触发的事件调用 `event.preventDefault()`.

![[Vue 指令完整结构.png|800]]

---

# 📚 参考内容

