---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
---

**关键词**: Vue, 事件处理 

---

可以使用 `v-on` 指令 (简写为 `@`) 来监听 DOM 事件, 并在事件触发时执行对应的 JavaScript.

```vue
<button @click="greet">Greet</button>

<script>
const name = ref('Vue.js')

function greet(event) {
  alert(`Hello ${name.value}!`)
  if (event) {
    alert(event.target.tagName)
  }
}
</script>
```

---