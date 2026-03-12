---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
aliases:
  - v-model
---
---

**关键词**: Vue, 绑定, v-model 

---

## 1	基本用法

`v-model` 可以将表单内容同步给 JavaScript 中相应的变量, 它根据所用的元素自动使用对应的 DOM 元素和事件组合.

```vue
<div>Selected: {{ selected }}</div>

<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

## 2	值绑定

可以通过 `v-bind` 将类似单选按钮, 复选框和选择器选项绑定的值绑定到该组件实例上的动态数据.

```vue
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no" />

<input type="radio" v-model="pick" :value="first" />
<input type="radio" v-model="pick" :value="second" />

<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option :value="{ number: 123 }">123</option>
</select>
```

## 3	修饰符

- `.lazy`: 实现在 `change` 事件后更新数据 (默认是 `input`);
	
- `.number`: 用户输入自动转化为数字, 会在输入框有 `type="nuber"` 时自动启动;
	
- `.trim`: 自动去除用户输入内容两端的空格.

---