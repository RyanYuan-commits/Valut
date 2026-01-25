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
vim.g.mapleader = " "
vim.keymap.set({ "n", "i" }, "<leader>b", ":lua print('hello world')<CR>", { silent = true })
```

## 2	rhs

当在 insert 模式下执行时, 直接将内容输入在光标后, 没有执行命令, 可以通过将 `:` 替换为 `<Cmd>` 来确保后面的内容是在命令模式下输入的.

还有一种方式, 是使用 lua 的函数:

```lua
vim.keymap.set({ "n", "i" }, "<leader>b", function ()
    print("hello world")
end, { silent = true })
```

## 3	lhs

建议使用统一的前缀, 如 `SPEC`, `Alt` 等, 这个 "前缀" 可以用 `<leader>` 表示:

```java
vim.g.mapleader = " "
vim.keymap.set("n", "<leader>aa", ":lua print(123)<CR>", {})
```

## 4	opts

### 4.1	remap

布尔值, 如果为 false 则禁用递归映射. 默认为 false. 设想一下, 如果你突发奇想想要交换 j 和 k 该怎么办呢?像这样?

```lua
vim. keymap. set("n", "j", "k", { remap = true })
vim. keymap. set("n", "k", "j", { remap = true })
```

加入我们启用了 remap, 那么就会导致 j 被映射到 k 上, k 再被映射到 j 上, 无穷递归. 所以, 我们需要禁用 remap. 实际上, 我们绝大多数情况下都没必要启用这个属性. 之所以在这里讲一下只是因为一些历史遗留原因, 你可能经常会看到这个属性. 但实际上, 我们根本用不到它.

### 4.2	silent

有些时候, 我们的快捷键对应的功能本身并不会有什么输出. 这个时候, 你会发现, 如果我们的 rhs 是一条命令, 那么这条命令会在 command line 被显示出来. 经过这段时间的观察, 你应该也发现了, command-line mode 下输入的命令在执行后不会被清空. 例如:

```lua
vim. keymap. set("n", ", a", ":lua a=1<CR>", {})
```

此时你会看到, 按下快捷键后, :lua a=1 被显示出来了. 这挺烦人的, 所以我们可以添加 silent = true 来解决问题:

```lua
vim. keymap. set("n", ", a", ":lua a=1<CR>", { silent = true })
```

### 4.3	nowait

这个属性一般是在你设计的快捷键是另一个快捷键的开头的时候使用. 例如:

```lua
vim. keymap. set("n", ", a", ":lua print(456)<CR>", {})
```

我们前面也讲到了这个, 当按下 , a 的时候, neovim 会等待一段时间, 因为它要判断你是不是想要按 , ab. 为了不等待, 我们可以加上 nowait = true.

```lua
vim. keymap. set("n", ", ab", ":lua print(123)<CR>", {})
vim. keymap. set("n", ", a", ":lua print(456)<CR>", { nowait = true })
```

不过, 这样做存在一个问题, 那就是更长的那个快捷键会失效. 所以, 我们一般只在想要更短的快捷键暂时生效, 且确定当前情境下不需要更长的那个快捷键的情况下使用这个属性.

### 4.4	desc

这个属性类似于一个注释, 是对快捷键功能的描述, 例如:

```lua
vim. keymap. set("n", ", ab", ":lua print(123)<CR>", { desc = "Print 123 in the cmdline" })
```

那么它有什么用呢?几乎没有, 因为你也可以直接写注释. 据我所知, 这个属性唯一的作用, 是有一些插件会基于你绑定的快捷键动态给出提示(比如 which-key), 此时你设置的 desc 属性就会被作为描述显示在插件的页面中.

---

# 📚 参考内容

