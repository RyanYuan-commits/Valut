---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
# 🌐 关键词

Neovim, File Explorer

---

# 🔖 详细解释

## 1	安装

通过 Lazy 安装, 文件路径为 `lua/plugins/nvim-tree.lua`

```lua
return {
    "nvim-tree/nvim-tree.lua",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    opts = {
        view = {
            width = 40,
        },
        renderer = {
            group_empty = true,
        },
    },
    keys = {
        { "<leader>uf", ":NvimTreeToggle<CR>", { slient = true } },
    },
    lazy = false,
}
```

## 2	操作

### 2.1	命令

`:NvimTreeToggle`: 打开 nvim-tree 界面;

`:NvimTreeFocus`: 聚焦与 nvim-tree 界面, 但 nvim-tree 界面是一个 Buffer, 可以通过 `<C-w>h` 与 `<C-w>l` 切换;

`:NvimTreeFindFile`: 将当前 Buffer 打开的文件在文件树中显示;

`NvimTreeCollapse`: 递归的折叠文件树.

### 2.2	配置

配置了文件树的宽度, 以及折叠空目录功能.

```lua
opts = {
	view = {
		width = 40,
	},
	renderer = {
		group_empty = true,
	},
}
```



---

# 📚 参考内容

