# 打造linux开发环境2.0

## 前言

笔者曾经写过一篇《打造可移植linux-go开发环境》主要是用docker 为内网机器提供 vim go 开发能力
现在笔者换了家公司做云原生，开发环境也可以直接联通公网。于是打算写一篇新的 linux 开发环境，目标是优化vim开发体验，
并且增加k8s支持等

## tmux

一个简单的分屏能力，通过该软件可以在终端分屏

`apt install tmux`

使用tmux只是为了分屏能力，更多的功能不是很需要，使用tmux 只需要在终端输入 `tmux`

所以只需要记录几个关键的快捷键就好

- 退出该session ctrl+d
- 左右分屏 ctrl+b+%
- 上下分屏 ctrl+b+"
- 分屏间移动 ctrl+b+方向键

### tmux 美化

因为原生的 tmux 不是很符合笔者审美，需要进行美化
这里选择的是 [tmux](https://github.com/gpakosz/.tmux)

安装方法很简单

```bash
 cd ~
 git clone https://github.com/gpakosz/.tmux.git
 ln -s -f .tmux/.tmux.conf
 cp .tmux/.tmux.conf.local .
```

## k9s

K9s是一个基于终端的界面化工具，功能丰富，可以方便地管理和监控Kubernetes集群。它提供了查看、编辑、删除和搜索资源的功能，还支持查看实时日志和执行命令。

[k9s官网](https://k9scli.io/)

k9s 的官网只给出了 通过 brew 与 源码编译两种方法，但是如果查看github文档会发现还有其他方法。

这里笔者先是尝试了源码编译与基于 go install 安装的方式

`go install github.com/derailed/k9s@v0.27.4`

但是发现，笔者环境的go版本是 `1.18.10` 而如果直接源码编译最新版（当前为0.27.4）或 go install。 会出现部分包需要更高版本的go支持的问题。所以最终采用了基于`webi`的方式

`curl -sS https://webinstall.dev/k9s | bash`

运行结束后有如下命令提醒:

`Installed 'k9s v0.27.4' as /root/.local/bin/k9s`

接下来我们只要在`/root/.local/bin 加入环境变量即可`

`export PATH=$PATH:/root/.local/bin`

## nvim

在发现nvim似乎可以原生支持lsp后，笔者决定将nvim设置成开发工具，而非vim

因为过去使用 centos 经常出现安装到极其落后的 vim 版本的情况。所以笔者此处并不打算通过 apt 包管理来安装neovim （同时安装测试了一下，neovim安装了0.4.3版本，但是当前neovim已经发行0.9.1版本）

1. 获取 nvim 发行版

`https://github.com/neovim/neovim/releases/download/stable/nvim-linux64.tar.gz`

2. 解压

`tar -C /usr/local -xzf nvim-linux64.tar.gz`

3. 修改PATH

`export PATH=$PATH:/usr/local/nvim-linux64/bin`

修改之后，nvim便安装完成了

### nvim 配置与插件管理

在 vim 配置中，笔者使用 vim-plug 作为插件管理工具。
而在 nvim 中则使用 lazy.nvim 作为插件管理工具

该插件管理工具的安装有三个前提：

- nvim >= 0.8.0
- git >= 2.19.0

安装只需在 `init.lua` 内引入对应配置即可，不过当前环境暂无 `init.lua` 相关，需要配置

创建 nvim 文件夹
`mkdir ~/.config/nvim`

nvim 文件夹 路径树

basic.lua 内容

``` lua
-- utf8
vim.g.encoding = "UTF-8"
vim.o.fileencoding = 'utf-8'
-- jk移动时光标下上方保留8行
vim.o.scrolloff = 8
vim.o.sidescrolloff = 8
-- 使用相对行号
vim.wo.number = true
-- vim.wo.relativenumber = true
-- 高亮所在行
vim.wo.cursorline = true
-- 显示左侧图标指示列
vim.wo.signcolumn = "yes"
-- 右侧参考线，超过表示代码太长了，考虑换行
-- vim.wo.colorcolumn = "80"
-- 缩进2个空格等于一个Tab
vim.o.tabstop = 2
vim.bo.tabstop = 2
vim.o.softtabstop = 2
vim.o.shiftround = true
-- >> << 时移动长度
vim.o.shiftwidth = 2
vim.bo.shiftwidth = 2
-- 新行对齐当前行，空格替代tab
vim.o.expandtab = true
vim.bo.expandtab = true
vim.o.autoindent = true
vim.bo.autoindent = true
vim.o.smartindent = true
-- 搜索大小写不敏感，除非包含大写
vim.o.ignorecase = true
vim.o.smartcase = true
-- 搜索不要高亮
vim.o.hlsearch = false
-- 边输入边搜索
vim.o.incsearch = true
-- 使用增强状态栏后不再需要 vim 的模式提示
vim.o.showmode = false
-- 命令行高为2，提供足够的显示空间
vim.o.cmdheight = 2
-- 当文件被外部程序修改时，自动加载
vim.o.autoread = true
vim.bo.autoread = true
-- 禁止折行
vim.o.wrap = false
vim.wo.wrap = false
-- 行结尾可以跳到下一行
vim.o.whichwrap = 'b,s,<,>,[,],h,l'
-- 允许隐藏被修改过的buffer
vim.o.hidden = true
-- 鼠标支持
vim.o.mouse = "a"
-- 禁止创建备份文件
vim.o.backup = false
vim.o.writebackup = false
vim.o.swapfile = false
-- smaller updatetime 
vim.o.updatetime = 300
-- 等待mappings
vim.o.timeoutlen = 100
-- split window 从下边和右边出现
vim.o.splitbelow = true
vim.o.splitright = true
-- 自动补全不自动选中
vim.g.completeopt = "menu,menuone,noselect,noinsert"
-- 样式
vim.o.background = "dark"
vim.o.termguicolors = true
vim.opt.termguicolors = true
-- 不可见字符的显示，这里只把空格显示为一个点
vim.o.list = true
vim.o.listchars = "space:·"
-- 补全增强
vim.o.wildmenu = true
-- Dont' pass messages to |ins-completin menu|
vim.o.shortmess = vim.o.shortmess .. 'c'
vim.o.pumheight = 10
-- always show tabline
vim.o.showtabline = 2
```

### 安装 lazy.lua

首先在 lua 文件夹下 创建 lazy.lua 文件
内容：

``` lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

之后在 init.lua 中加入

`require("lua.lazy")`

再次打开即第一次安装 lazy.lua

之后为 lazy.lua 添加 neo lsp支持，测试是否安装成功

``` lua
require("lazy").setup {
  -- Other plugins ...
  {
    "neovim/nvim-lspconfig",
    config = function()
      -- Configuration for nvim-lspconfig
    end,
  },
  -- Other plugins ...
}
```

追加写入后，再次打开 nvim 便会显示安装界面
