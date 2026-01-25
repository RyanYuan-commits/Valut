---
type: permanent
banner: Assets/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
# 🌐 关键词

Neovim, Plugins

---

# 🔖 详细解释

## 1	插件管理器

插件管理器负责了安装, 更新, 加载插件的功能, 使用 lazy.

安装 lazy 前需要安装好 git 和一款 nerd font (一种将普通的字体和一些特殊符号 patch 在一起的新的字体), 可以在 https://www.nerdfonts.com/font-downloads 安装.

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"

if not vim.uv.fs_stat(lazypath) then
    vim.fn.system({
        "git",
        "clone",
        "--filter=blob:none",
        "https://github.com/folke/lazy.nvim.git",
        "--branch=stable",
        lazypath,
    })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({})
```


---

# 📚 参考内容

