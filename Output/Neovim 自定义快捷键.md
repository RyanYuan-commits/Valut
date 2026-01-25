---
type: permanent
---
# 🌐 关键词

Neovim

---

# 🔖 详细解释

## 1	使用的 API

在 Neovim 使用, `vim.keymap.set(mode, lhs, rhs, opts)` 来绑定一个快捷键:

- mode: 快捷键生效的模式, 可以是一个字符串(仅对单一模式生效), 也可以是一个 table(类似于 array 和 object 的混合, 即, 其中的元素可以没有键名也可以是键值对, 此时快捷键对多个模式生效); 这些模式都由一个字母构成, 例如 "n" (normal mode) / "i" (insert mode) / "c" (command-line mode)…
	
- lhs: 快捷键的按键, 是一个字符串; 以 Ctrl 开头的按键表示为 `<C->`, 例如 `<C-a>` 就表示 `Ctrl + a`; 以 Alt 开头的按键表示为 `<A-?>`, 例如 `<A-b>` 就表示 Alt + b
	
- rhs: 快捷键绑定的功能, 可以是另外一组按键, 也可以是一个 lua 函数
	
- opts: 又一个 table, 包含了对这个快捷键的一些额外设置, 这个我们在本讲后面单开一节进行说明

```lua
vim.keymap.set("n", "<C-a>b", ":lua print('hello world')<CR>", { silent = true })
```






---

# 📚 参考内容

