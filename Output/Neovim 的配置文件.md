---
type: permanent
banner: Assets/Banner/pexels-8kspain-21564213.jpg
---
# 🌐 关键词

Neovim

---

# 🔖 详细解释

## 1	Neovim 的配置文件

在 neovim 中, 配置文件往往位于以下位置, 可以通过命令 `:= vim.fnstdpath("config")` 输出.

```
Unix: ~/.config/nvim/init.lua
Windows: ~/AppData/Local/nvim/init.lua
```

Neovim 的配置文件可以拆分, 拆分的文件放在配置文件目录下的 lua 目录中, 并通过如下方式引入:

```lua
-- 路径: lua/module.lua
require("module")

-- 路径 lua/core/module.lua
require("core.module")
```

## 2	一些有用的配置

### 2.1	显示行号

```lua
vim.opt.number = true
```

### 2.2	高亮行 / 列

分别用于高亮当前行和高亮某一列 (用于一行代码推荐最大长度配置)

```lua
vim.opt.cursorline = true
vim.opt.colorcolumn = "120"
```

### 2.3	Tab 键与缩进

使用 `vim.opt.expandtab` 属性, 将 Tab 转化为空格, 但是在开头敲下 Tab, 会打出 8 个空格, 这是因为 Neovim 开启了 smarttab 选项, 在开头敲下 Tab 时, 会填充 `shiftwidth` 个空格, 所以需要配置将其设置为 0.

```lua
vim.opt.expandtab = true
vim.opt.tabstop = 4
vim.opt.shiftwidth = 0
```

### 2.4	autoread

配置当文件被外部修改后, Neovim 会重新打开它.

```lua
vim.opt.autoread = true
```

## 3	最终配置

```lua
 -- 显示行号
vim.opt.number = true

-- 高亮当前行
vim.opt.cursorline = true

-- 高亮第 120 列
vim.opt.colorcolumn = "120"

-- Tab 配置
vim.opt.expandtab = true
vim.opt.tabstop = 4
vim.opt.shiftwidth = 0

-- 文件被外部修改后, 重新打开
vim.opt.autoread = true
```

---

# 📚 参考内容

