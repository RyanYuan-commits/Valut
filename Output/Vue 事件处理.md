---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
---

**关键词**: Vue, 事件处理 

---

## 1	基础使用

可以使用 `v-on` 指令 (简写为 `@`) 来**监听** DOM 事件, 并在**事件触发**时执行对应的 JavaScript.

```vue
<button @click="warn('Form cannot be submitted yet.', $event)"> 
	Submit 
</button>

<script>
function warn(message, event) {
  if (event) {
    event.preventDefault()
  }
  alert(message)
}
</script>
```

## 2	事件修饰符

```vue
<a @click.stop.prevent="doThat">...</a>
```

让开发者在模板中直接处理常见的 DOM 事件细节, 如 `event.preventDefault()`.


---