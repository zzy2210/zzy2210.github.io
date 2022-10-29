---
title: "打造可移植 linux-go 开发环境"
date: 2022-08-15T16:57:03+08:00
lastmod: 2022-10-29T16:57:03+08:00
draft: false
keywords: ["development","vim"]
description: "基于docker镜像构建vim开发环境"
tags: ["development","vim"]
categories: ["development","vim"]
author: "y1nhui"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---

<!--more-->

# 打造可移植 linux-go 开发环境

## 前言

大家伙都不发帖，我来水一水

因为公司是内外网隔离环境，且内网机器屏幕素质太差，个人打算通过外网堡垒机进入内网linux机器上使用进行开发。

出于避免因网络问题导致的各种插件安装失败与方便迁移的目的，本人打算使用docker构建开发环境的镜像。这样只需要在外网导出镜像文件传入内网机器，便可使用，做到开箱即用。

本文主要内容是操作记录日志，并尽量附带指令解析。内容分为一下部分：

- 系统环境构建
- vim编辑器打造

## 系统环境构建

系统描述：

- ubuntu
- go 1.19
- vim
- gcc & gcc++
- make
- vim
- and so on

### ubuntu 构建

使用docker 下载ubuntu镜像
`docker pull ubuntu`

之后运行容器
 `docker exec -it go-dev bash`

进入容器后，出于习惯换源代，更新

`sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list`

 `apt update && apt upgrade`

ps:目前默认源也是可以使用的，速度大概几百k，说快不快说慢不慢的，只是个人喜欢使用中科大的源。

之后就是我们的环境安装
`apt install build-essential vim net-tools git sudo -y`

build-essential 可以安装包函 gcc g++ make 的 GNU 编辑器集合，GNU 调试器，和其他编译软件所必需的开发库和工具。

### 镜像制作

到了这一步发现因为启动容器时忘了-v，导致目前无法进行本机与容器的文件交互，所以打算先制作一次镜像，然后再启动容器

获取容器信息：

![容器列表](https://i.bmp.ovh/imgs/2022/08/08/0e2b84582e41d389.png)

接着使用docker commit 创建容镜像

`docker commit -a "y1nhui" -m "ubuntu-dev" 7658d10be2dd go-dev:v1`

命令描述： `-a` 作者 `-m` 描述 `7658...`容器id `go-dev:v1`新建的镜像名与版本

使用`docker images` 可见镜像已创建：
![镜像列表](https://i.bmp.ovh/imgs/2022/08/08/38314694da553520.png)

### go环境搭建

接着便是启动新容器，挂载/root到~/work

`docker run -itd -v /Users/y1nhui/WorkSpace/:/root/ --name dev go-dev:v1`

下载了go1.19后，执行命令

`tar -C /usr/local -xzf go1.19.linux-amd64.tar.gz`

编写/etc/profile 增加 `export PATH=$PATH:/usr/local/go/bin`

`source /etc/profile`

运行 `go version` 可见![go版本](https://i.bmp.ovh/imgs/2022/08/08/4c6ff045f00739ff.png)

### omz

需求总是测试中不断出现的，想用omz了
`apt install zsh wget -y`

`sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"`

如此的话，还需要变更go的环境变量
`vim ~/.oh-my-zsh/oh-my-zsh.sh`，
添加`export PATH=$PATH:/usr/local/go/bin:/root/go/bin`

添加指令`go env -w GOBIN="/root/go/bin"`。将gobin路径加入了path中，之后我们一些通过go install 安装的二进制文件便可以正常使用

安装到这里，再更新一次 docker 镜像，同时我在 docker hub 创建了一个仓库，我们可以push到远端

`docker commit -m "完成go环境安装与omz" 97ebe4cda8d0 y1nhui/go-dev`

`docker login`

`docker push y1nhui/go-dev:latest`

### ssh-server

想测试的时候发现忘记安装ssh-server了 安装配置一下
`apt install openssh-server`

`vim /etc/ssh/sshd_config`

`PermitRootLogin yes`
允许一下root远程登录

## vim

来到重头戏了，参考一下他人的博客

[将 VIM 打造成 go 语言的 ide](https://learnku.com/articles/24924)

本来是想直接使用这位的配置的，但是因为部分插件和我想用的不太一样，同时为了省去读者再打开一个网页的步骤。我个人还是会在文章里给出vimrc的内容。

首先通过修改~/.vimrc 来更改vim配置

一般情况下没有~/.vimrc。这是因为～/.vimrc是用户级的配置文件，一般需要用户手动创建.

同时可能会发现vim中文乱码问题

修改方法：
vim内使用指令：`:set encoding=utf8`

```vim
"==============================================================================
  3 " vim 内置配置
  4 "==============================================================================
  5
  6 " 设置 vimrc 修改保存后立刻生效，不用在重新打开
  7 " 建议配置完成后将这个关闭
  8 autocmd BufWritePost $MYVIMRC source $MYVIMRC
  9
 10 " 关闭兼容模式
 11 set nocompatible
 12
 13 " 设置编码 utf-8
 14 set encoding=utf8
 15
 16 set nu " 设置行号
 17 set cursorline "突出显示当前行
 18 " set cursorcolumn " 突出显示当前列
 19 set showmatch " 显示括号匹配
 20
 21 " tab 缩进
 22 set tabstop=4 " 设置Tab长度为4空格
 23 set shiftwidth=4 " 设置自动缩进长度为4空格
 24 set autoindent " 继承前一行的缩进方式，适用于多行注释
 25
 26 " 定义快捷键的前缀，即<Leader>
 27 let mapleader=";"
 28
 29 " ==== 系统剪切板复制粘贴 ====
 30 " v 模式下复制内容到系统剪切板
 31 vmap <Leader>c "+yy
 32 " n 模式下复制一行到系统剪切板
 33 nmap <Leader>c "+yy
 34 " n 模式下粘贴系统剪切板的内容
 35 nmap <Leader>v "+p
 36
 37 " 开启实时搜索
 38 set incsearch
 39 " 搜索时大小写不敏感
 40 set ignorecase
 41 syntax enable
 42 syntax on                    " 开启文件类型侦测
 43 filetype plugin indent on    " 启用自动补全
 44
 45 " 退出插入模式指定类型的文件自动保存
 46 au InsertLeave *.go,*.sh,*.proto write
```

之后下载插件，vim插件管理器 vim-plug

`curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim`

为了安装ncoc与tagbar等，安装前置工具

安装node js
`curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs`

安装 ctags
`sudo apt install universal-ctags`

下载之后继续配置vimrc，使用`call plug#begin()`与`call plug#end()`做插件列表开始结束提示。

```vim
call plug#begin()

 " 代码自动完成插件，抛弃vim-go插件
 Plug 'neoclide/coc.nvim', {'branch': 'release'}

" 查看代码结构
Plug 'majutsushi/tagbar'

"go-imports
Plug 'mattn/vim-goimports'

" markdown 预览
Plug 'iamcco/markdown-preview.nvim', { 'do': { -> mkdp#util#install() }, 'for': ['markdown', 'vim-plug']}

" 文件侧栏
Plug 'preservim/nerdtree' 

" nerdtree -git
Plug 'Xuyuanp/nerdtree-git-plugin'

" 底部状态栏
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'

" 文档显示git信息
Plug 'airblade/vim-gitgutter'

" vim-one 主题
Plug 'rakr/vim-one'
" gruvbox主题 
Plug 'morhetz/gruvbox'

call plug#end()
```

保存后再次打开vmrc，使用`:PlugInstall`命令下载安装插件

此处可能会因网络错误导致下载失败，如果像笔者一样使用proxychains代理工具 则使用这种方法
`proxychains vim ~/.vimrc`
这样便可以为vim进程附加代理

当然，如果可以直接使用`export https_proxy=`的方法再好不过

### docker 容器使用宿主机代理 / 设置相通网卡

这里就多了一个问题，如何使用docker访问本机代理。我的代理在127.0.0.1:7777上，如果容器内想使用的话就需要访问到该端口，使用-p映射是不现实的。那么就需要使用 docker network

linux机器可下docker会自己创建一个docker0网卡，但是mac似乎不会，所以我需要手动创建

`docker network create n1`

创建后，我们可以使用
`docker network inspect n1`
来查看

```bash
[
    {
        "Name": "n1",
        "Id": "sha1?",
        "Created": "time",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

再次创建启动docker容器，指定network，同时使用-h来设定主机名
`docker run -itd -p 6022:22   -h go-dev --network n1 --name dev y1nhui/go-dev`

进入容器，使用`ping host.docker.internal` host.docker.internal是docker容器对宿主机的代指，如此ping便可获得宿主机ip

接着便可以设定代理

### 插件配置

做完了安装工作后，需要继续插件配置，首先是对nerdtree调整

```vim
 74 " Nerdtree 配置
 75 " " 显示行号
 76 let NERDTreeShowLineNumbers=1
 77 " 打开文件时是否显示目录
 78 let NERDTreeAutoCenter=1
 79 " 是否显示隐藏文件
 80 let NERDTreeShowHidden=0
 81 " 设置宽度
 82 " let NERDTreeWinSize=31
 83 " 忽略一下文件的显示
 84 let NERDTreeIgnore=['\.pyc','\~$','\.swp']
 85 " 打开 vim 文件及显示书签列表
 86 let NERDTreeShowBookmarks=2
 87
 88 " 在终端启动vim时，共享NERDTree
 89 let g:nerdtree_tabs_open_on_console_startup=1
 ```

 之后配置调色

 ```vim
 set termguicolors
 " 主题配色
 colorscheme gruvbox
 " 主题背景 dark-深色; light-浅色
 set background=dark
 ```

之后对ncoc配置

安装gopls
`go get golang.org/x/tools/gopls@latest`

再次进入vimrc，安装ncoc的插件
`:CocInstall coc-go` `:CocInstall coc-sh`

最后贴一下全部配置
ps：配置文件是根据
[BroQiang](https://github.com/BroQiang/vim-go-ide)仓库为主体更改的。
因为ncoc更新缘故，部分ncoc配置需要更改，具体为tab与cr部分，本人文件已经完成更改，可直接使用.

```vim
"==============================================================================
" 处理 Gnome 终端不能使用 alt 快捷键， 不做这个处理无法在 vim 中映射 alt的快捷键
"==============================================================================
let c='a'
while c <= 'z'
	exec "set <A-".c.">=\e".c
	exec "imap \e".c." <A-".c.">"
	let c = nr2char(1+char2nr(c))
endw

set timeout ttimeoutlen=50

" ======================== 一些初始配置 ===========================
" 关闭兼容模式, 如果需要使用原始的 vi 模式， 配置： set compatible
" 默认就是关闭的， 如果不关，就无法使用 vim 的高级功能，包括下面的配置
set nocompatible

" 打开文件类型检测
filetype plugin indent on

" 定义快捷键的前缀，即<Leader> ， 默认是 \ ， 按的时候不太方便
let mapleader=";"

" 记住上次文件打开位置
if has("autocmd")
  au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif

" 行列相关配置
set number " 显示绝对行号
set relativenumber " 显示相对行号， 会覆盖上面选项
" 设置切换绝对行号和相对行号的快捷键， 这个需要的时候切换提高效率
" 这里默认是相对行号， 需要绝对行号时候可以通过快捷键修改
nmap <Leader>nn :set relativenumber<CR>
nmap <Leader>nu :set norelativenumber<CR>
set cursorline " 突出显示当前所在行
" set cursorcolumn " 突出显示当前列

" ==== tab 缩进
" 设置Tab长度为4空格， 只是显示， 真实的还是一个 tab
set tabstop=4

" 设置输入的 tab 转换成 4 个空格，这里只允许指定文件启用这个设置
au BufRead,BufNewFile *.md call SetTabToSpace()
function SetTabToSpace()
    set shiftwidth=4  " 转换 4 个空格
    set expandtab     " 实时生效，配合 shiftwidth
	set softtabstop=4 " 删除时候的行为， 也会同时删除 4 个空格
endfunction

" 继承前一行的缩进方式
set autoindent

" 开启实时搜索，随着你输入查询字串，显示不同的搜索结果
set incsearch
" 搜索时大小写不敏感
set ignorecase

" 退出插入模式指定类型的文件自动保存
au InsertLeave *.go,*.md,*.proto write

" 保存文件时候自动去除行尾空格
autocmd BufWritePre *.md,*.go :%s/\s\+$//e

" 全选
nmap <leader>a ggvG$

" 系统剪切板复制粘贴， vim 使用系统剪切板需要 vim 支持
" 查询可以通过 vim --version | grep clipboard 查看
" 如果显示 +clipboard 就是支持， 如果是 -clipboard 就是不支持
" ubuntu 可以直接安装 gui 包提供支持： sudo apt install vim-gtk
" normal 模式下复制到系统剪切板， 这里没有 "+yy 这样只能复制一行
" 使用的时候可以 alt + c ， 然后再输入 yy ， 就是一行
" 也可以在文件顶部， alt + c ， 然后再输入 yG ， 就是全部内容
" 粘贴可以 alt + c ，然后输入 p， 就是为了省略不太好按的 "+ 组合
nmap <M-c> "+
" 复制 v 模式的选中区域
vmap <M-c> "+y

" =================================================================


"==============================================================================
" 插件开始的位置
"==============================================================================
call plug#begin('~/.vim/plug')

set encoding=UTF-8

" 主题配色
Plug 'morhetz/gruvbox'

" 状态栏插件，包括显示行号，列号，文件类型，文件名，以及Git状态
Plug 'vim-airline/vim-airline'
" 状态栏目插件的主题，使用后可以使颜色看起来更加突出一点
Plug 'vim-airline/vim-airline-themes'

" 用来提供一个导航目录的侧边栏
Plug 'preservim/nerdtree'
" nerdtree 中显示 git 状态
Plug 'Xuyuanp/nerdtree-git-plugin'

" 可以在导航中显示图标， 不过需要有字体支持，否则就是乱码
" https://github.com/ryanoasis/nerd-fonts
" 终端也需要字体配合，如我使用的是 firacode nerd font Regular
" 喜欢其他字体也可以，不过要使用带 nerd font 的字体
Plug 'ryanoasis/vim-devicons'

" 可以在文档中显示 git 信息
Plug 'airblade/vim-gitgutter'

" markdown 预览
Plug 'iamcco/markdown-preview.nvim', { 'do': { -> mkdp#util#install() }, 'for': ['markdown', 'vim-plug']}
" markdown 中 latex 数学公式支持
" Plug 'iamcco/mathjax-support-for-mkdp'

" 下面两个插件要配合使用，可以自动生成代码块
" B
" Plug 'SirVer/ultisnips'   " 插件本身， 使用 coc-snippets 替换了
Plug 'honza/vim-snippets' " 代码片段仓库

" 代码自动完成插件
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" go-imports
Plug 'mattn/vim-goimports'

" go tag
Plug 'majutsushi/tagbar'
" 插件结束的位置，插件全部放在此行上面
call plug#end()
"==============================================================================


" ======================== 主题配色设置 ===========================
" 开启24bit的颜色，开启这个颜色会更漂亮一些
set termguicolors
" 主题配色
colorscheme gruvbox
" 主题背景 dark-深色; light-浅色
set background=dark
" =================================================================


" ======================== nerdtree 插件配置 ======================
" 设置一个打开的快捷键， 如我的就是 " + b 打开， 再按一次关闭
nnoremap <leader>b :NERDTreeToggle<CR>
" 导航目录展开的符号
let g:NERDTreeDirArrowExpandable = '▸'
" 导航目录关闭的符号
let g:NERDTreeDirArrowCollapsible = '▾'
" 默认显示行号
let NERDTreeShowLineNumbers=1
" 打开文件时是否显示目录， 1- 显示 0- 不显示
let NERDTreeAutoCenter=1
" 是否显示隐藏文件
let NERDTreeShowHidden=1
" 设置宽度
" let NERDTreeWinSize=31
" 忽略一下文件的显示， 可以定义只隐藏指定文件或目录
let NERDTreeIgnore=['\.pyc','\~$','\.swp','\.git']
" =================================================================


" ======================== nerdtree-git-plugin 插件 ===============
" 是否显示忽略文件 1- 显示 0- 不显示 默认 0
" let g:NERDTreeGitStatusShowIgnored = 1
" =================================================================


" ======================== vim-airline 插件配置 ===================
" 状态栏目使用主题， 还可以使用 dark， simple， solarized light 等
" 更多主题 https://github.com/vim-airline/vim-airline/wiki/Screenshots
let g:airline_theme='light'
" 安装字体后，启用这个选项后在状态栏可以看到图标
" 需要先安装 powerline 字体才可以 sudo apt-get install fonts-powerline
" 如果没有安装 powerline 字体不要启用这个选项，否则会显示乱码
" 如果 vim-devicons 插件配置好了就可以忽略 powerline 字体，这个插件包含
" 了字体图标，并且更加漂亮
let g:airline_powerline_fonts = 1
" 启动顶部的 tabline ， 可以显示打开的 buffers， 显示多 tab 标签
let g:airline#extensions#tabline#enabled = 1
" 设置切换 tab 标签（buffer）的快捷键
let g:airline#extensions#tabline#buffer_idx_mode = 1
 nmap <leader>1 <Plug>AirlineSelectTab1
 nmap <leader>2 <Plug>AirlineSelectTab2
 nmap <leader>3 <Plug>AirlineSelectTab3
 nmap <leader>4 <Plug>AirlineSelectTab4
 nmap <leader>5 <Plug>AirlineSelectTab5
 nmap <leader>6 <Plug>AirlineSelectTab6
 nmap <leader>7 <Plug>AirlineSelectTab7
 nmap <leader>8 <Plug>AirlineSelectTab8
 nmap <leader>9 <Plug>AirlineSelectTab9
 nmap <leader>0 <Plug>AirlineSelectTab0
 nmap <leader>h <Plug>AirlineSelectPrevTab
 nmap <leader>l <Plug>AirlineSelectNextTab
" =================================================================


" ======================== markdwon 插件配置 ======================
" 更多的配置见 https://github.com/iamcco/markdown-preview.nvim
" vim 打开markdown 文档是否自动预览， 0 - 否， 1 - 是， 默认： 0
" let g:mkdp_auto_start = 1

" 关闭 markdown 是否自动关闭预览文件， 0 - 否， 1 - 是， 默认： 1
" let g:mkdp_auto_close = 1

" 在启动 markdown 预览时是否在终端回显 url
" 如显示： Preview page: http://127.0.0.1:8472/page/1
let g:mkdp_echo_preview_url = 1

" markdwon 的快捷键
map <silent> <F5> <Plug>MarkdownPreview
map <silent> <F6> <Plug>StopMarkdownPreview
" =================================================================


" ======================== coc.nvim 插件配置 ======================
" 详细见 https://github.com/neoclide/coc.nvim#example-vim-configuration
" 不清楚作用，推荐配置里面有，详细见 https://github.com/neoclide/coc.nvim/issues/649
set nobackup
set nowritebackup

" 更改更新时间， 默认是 4000 ms
set updatetime=300

" 为消息留出更多的空间
set cmdheight=2


" 追踪到定义的位置
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
" 查看被谁引用
nmap <silent> gr <Plug>(coc-references)

" 预览窗口显示文档
nnoremap <silent> K :call <SID>show_documentation()<CR>
function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  elseif (coc#rpc#ready())
    call CocActionAsync('doHover')
  else
    execute '!' . &keywordprg . " " . expand('<cword>')
  endif
endfunction

autocmd CursorHold * silent call CocActionAsync('highlight')

" 重命名的快捷操作
nmap <leader>rn <Plug>(coc-rename)

" 关闭提示后手动唤醒提示
inoremap <silent><expr> <c-l> coc#refresh()
" 通过回车展开代码片段
inoremap <silent><expr> <CR> coc#pum#visible() ? coc#pum#confirm()
                              \: "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"

" 可以通过 tab 键来切换提示列表选择
inoremap <silent><expr> <TAB>
      \ coc#pum#visible() ? coc#pum#next(1):
      \ CheckBackspace() ? "\<Tab>" :
      \ coc#refresh()
inoremap <expr><S-TAB> coc#pum#visible() ? coc#pum#prev(1) : "\<C-h>"

function! CheckBackspace() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction
" 代码提示列表选择， 将原本的 c-p 和 c-n 添加更习惯的方式
inoremap <silent><expr> <M-k> "\<C-p>"
inoremap <silent><expr> <M-j> "\<C-n>"

" 格式化代码， 需要 lsp 支持
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

" 修复当前选中的代码
nmap <leader>qf  <Plug>(coc-fix-current)

" 显示错误信息
nmap <silent> <M-k> <Plug>(coc-diagnostic-prev)
nmap <silent> <M-j> <Plug>(coc-diagnostic-next)

" coc-go 保存的时候自动导包
autocmd BufWritePre *.go :silent call CocAction('runCommand', 'editor.action.organizeImport')
" =================================================================

" tag-bar
let g:godef_split=2
" 使用f8 执行
nmap <F8> :TagbarToggle<CR>
```

## 补充

最近调试发现 有个小问题 就是如果使用mac 会发现退格键无法使用

具体表现为：无法删除之前写的字符（除非使用d d命令），但是可以删除本次插入模式内写入的字符，具体原理不说了，解决方法.vimrc 加一句话

`set backspace=2`