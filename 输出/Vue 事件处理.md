---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
aliases:
  - v-on
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

让开发者**在模板中**直接处理常见的 DOM 事件细节, 如 `event.preventDefault()`, 修饰符有 `.stop`, `.prevent`, `.self`, `.capture`, `.once`, `.passive`, 可以使用链式书写拼接多个修饰符.

```vue
<a @click.stop.prevent="doThat"></a>
```

## 3	按键修饰符

Vue 允许在监听按键时间时添加按键修饰符, 并为一些常用的按键提供了别名: `.enter`, `.tab`, `.delete`, `.esc`, `.space`, `.up`, `.down`, `.left`, `.right`,

可以通过 `.exact` 修饰符实现精确控制.

```vue
<!-- 当按下 Ctrl 时，即使同时按下 Alt 或 Shift 也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 仅当按下 Ctrl 且未按任何其他键时才会触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>
```

## 4	鼠标按键修饰符

允许将处理程序限定为由特定鼠标按键触发的事件, 修饰符有 `.left`, `.right`, `.middle`,

---