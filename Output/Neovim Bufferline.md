---
type: permanent
banner: Assets/Banner/pexels-ken-cheung-3355734-5574638.jpg
---
# 🌐 关键词

Neovim, Buffer, Window, Tab

---

# 🔖 详细解释

## 1	Buffer, Window, Tab

Neovim 的官方文档中是这样描述这三个概念的:

```
A buffer is the in-memory text of a file.

A window is a viewport on a buffer.

A tab page is a collection of windows.
```

当打开一个文件时, 就会创建一个对应的 Buffer, 对文件的编辑实际上是对 Buffer 的编辑 (源文件没有改变), 除非进行保存. 当使用 `edit` 命令打开一个新文件时, 原本的 Buffer 实际上并没有消失, 而是转为 `hidden` 状态;

Neovim 的 Window 是用来呈现一个 Buffer 的;

多个 Window 可以组成一个 Tab, 就是看到的整个页面.





---

# 📚 参考内容

