---
type: permanent
banner: Assets/Banner/pexels-ken-cheung-3355734-5574638.jpg
---
# 🌐 关键词

Neovim, Buffer, Window, Tab

---

# 🔖 详细解释

## 1	Buffer, Window, Tab

### 1.1	概念介绍

Neovim 的官方文档中是这样描述这三个概念的:

```
A buffer is the in-memory text of a file.

A window is a viewport on a buffer.

A tab page is a collection of windows.
```

当打开一个文件时, 就会创建一个对应的 Buffer, 对文件的编辑实际上是对 Buffer 的编辑 (源文件没有改变), 除非进行保存. 当使用 `edit` 命令打开一个新文件时, 原本的 Buffer 实际上并没有消失, 而是转为 `hidden` 状态;

Neovim 的 Window 是用来呈现一个 Buffer 的;

多个 Window 可以组成一个 Tab, 就是看到的整个页面.

### 1.2	原生操作方式

使用 `:buffers` 查看当前所有的 Buffer, 类似:

```
1 %a + "content/posts/neovim-beginner-guide/07/index.md" line 27
9  h   "~/Documents/orgfiles/neovim.org" line 95
```

然后可以通过 `:buffer` 加上 id 来切换到指定的 Buffer, 其中 `a` 表示 active, `h` 表示 hidden, `+` 表示被修改但是没有写入文件.

可以将同一个 Buffer 在不同的 Window 显示, 使用 `vsplit` 和 `split` 分别进行左右和上下分屏, 窗口管理的快捷键有:

- `<C-w>h`: 切换到左侧窗口
	
- `<C-w>l`: 切换到右侧窗口
	
- `<C-w>j`: 切换到下方窗口
	
- `<C-w>k`: 切换到上方窗口
	
- `<C-w>c`: 关闭窗口
	
- `<C-w>o`: 关闭其他所有窗口

## 2	使用 Bufferline 插件

使用 `plugins/bufferline.lua` 新建:

```lua
return {
    "akinsho/bufferline.nvim",
}
```



---

# 📚 参考内容

