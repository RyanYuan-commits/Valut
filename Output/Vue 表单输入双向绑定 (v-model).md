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





---