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
# 路径: lua/module.lua
require("module")

# 路径 lua/core/module.lua
require("core.module")
```

## 2	一些有用的配置

### 2.1	显示行号

```lua
vim.opt.number = true
```

### 2.2	高亮行 / 列

分别用于高亮当前行和高亮mou

```lua
vim.opt.cursorline = true
vim.opt.colorcolumn = "120"
```




---

# 📚 参考内容

