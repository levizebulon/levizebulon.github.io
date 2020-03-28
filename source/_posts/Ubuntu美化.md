---
title: Ubuntu美化
date: 2019-01-02 07:32:54
tags:
---

# GNOME主题

## 1. 首先安装tweak tool
~~~bash
sudo apt install gnome-tweak-tool
~~~

## 2. 然后安装user themes插件
~~~bash
sudo apt install gnome-shell-extensions
~~~

## 3. 下载主题
#### - 安装Arc主题 [horst3180/arc-theme](https://github.com/horst3180/arc-theme)
~~~bash
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/Horst3180/xUbuntu_16.04/ /' > /etc/apt/sources.list.d/home:Horst3180.list"
sudo apt-get update
sudo apt-get install arc-theme
~~~

#### - 安装Flatabulous主题 [anmoljagetia/Flatabulous](https://github.com/anmoljagetia/Flatabulous)
~~~bash
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
~~~

然后在Tweak Tool-->Apprarance-->Theme中选择修改


点击下载主题:[https://www.gnome-look.org/](https://www.gnome-look.org/)

## 4. 解压

将下载好的文件夹解压后移动到~/.themes文件夹下面，如果没有就创建一个

重新启动tweak tool，在Appearance选项卡的shell theme中选择你想要的shell theme即可

按"Alt+F2"组合键，弹出命令窗口，输入"r"，重启桌面环境，然户就可以看到成功更换主题了

# 图标和指针主题

图标和指针的主题更换方式和上面一样，只不过下载并解压后的文件夹放置位置不同。

- 图标: 放置到/usr/share/icons/中
- 指针: 放置到~/.icons目录中

# vim插件

### 1. 首先我们先安装一个插件管理器
~~~bash
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
~~~
### 2. 然后配置插件
打开~/.vimrc,在call vundle#begin()和call vundle#end()之间添加
~~~bash
" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
~~~
### 3. 安装插件
在vim下运行:PluginInstall

以后所有插件都可以用这种方法安装
这里贴一下我的vimrc配置文件
~~~bash
set nocompatible              " 去除VI一致性,必须
filetype off                  " 必须

" 设置包括vundle和初始化相关的runtime path
set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
" 另一种选择, 指定一个vundle安装插件的路径
"call vundle#begin('~/some/path/here')

" 让vundle管理插件版本,必须
Plugin 'VundleVim/Vundle.vim'

" The NERDTree is a file system explorer for the Vim editor
" https://github.com/scrooloose/nerdtree
Plugin 'scrooloose/nerdtree'

" Tagbar: a class outline viewer for Vim
" https://github.com/majutsushi/tagbar
Plugin 'majutsushi/tagbar'

" altercation/vim-colors-solarized
" https://github.com/altercation/vim-colors-solarized
Plugin 'altercation/vim-colors-solarized'

" 以下范例用来支持不同格式的插件安装.
" 请将安装插件的命令放在vundle#begin和vundle#end之间.
" Github上的插件
" 格式为 Plugin '用户名/插件仓库名'
Plugin 'tpope/vim-fugitive'
" 来自 http://vim-scripts.org/vim/scripts.html 的插件
" Plugin '插件名称' 实际上是 Plugin 'vim-scripts/插件仓库名' 只是此处的用户名可以省略
"Plugin 'L9'
" 由Git支持但不再github上的插件仓库 Plugin 'git clone 后面的地址'
"Plugin 'git://git.wincent.com/command-t.git'
" 本地的Git仓库(例如自己的插件) Plugin 'file:///+本地插件仓库绝对路径'
"Plugin 'file:///home/gmarik/path/to/plugin'
" 插件在仓库的子目录中.
" 正确指定路径用以设置runtimepath. 以下范例插件在sparkup/vim目录下
"Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" 安装L9，如果已经安装过这个插件，可利用以下格式避免命名冲突
"Plugin 'ascenator/L9', {'name': 'newL9'}

Plugin 'suan/vim-instant-markdown'

" 你的所有插件需要在下面这行之前
call vundle#end()            " 必须
filetype plugin indent on    " 必须 加载vim自带和插件相应的语法和文件类型相关脚本

" 忽视插件改变缩进,可以使用以下替代:
"filetype plugin on
"
" 简要帮助文档
" :PluginList       - 列出所有已配置的插件
" :PluginList       - 列出所有已配置的插件
" :PluginInstall    - 安装插件,追加 `!` 用以更新或使用 :PluginUpdate
" :PluginSearch foo - 搜索 foo ; 追加 `!` 清除本地缓存
" :PluginClean      - 清除未使用插件,需要确认; 追加 `!` 自动批准移除未使用插>件
"
" 查阅 :h vundle 获取更多细节和wiki以及FAQ
" 将你自己对非插件片段放在这行之后

set nu          "在每一行最前面显示行号
" set autoindent        "自动缩排
" set cindent   "按照 C 语言的语法,自动地调整缩进的长度

syntax on       "可以进行语法检验,颜色显示
set background=dark
colorscheme solarized

set fileencoding=utf-8

" 贴时保持格式  
set paste
"为方便复制，用<F6>开启/关闭行号显示:  
nnoremap <F6> :set nonumber!<CR>:set foldcolumn=0<CR>

" Tagbar
let g:tagbar_width=15
let g:tagbar_autofocus=1
let g:tagbar_right = 1
nmap <F3> :TagbarToggle<CR>

" NERDTree config
" " open a NERDTree automatically when vim starts up
autocmd vimenter * NERDTree
" "open a NERDTree automatically when vim starts up if no files were specified
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
" "open NERDTree automatically when vim starts up on opening a directory
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
" "open NERDTree automatically when vim starts up on opening a directory
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | endif
" "map F2 to open NERDTree
map <F2> :NERDTreeToggle<CR>
" "close vim if the only window left open is a NERDTree
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
let g:NERDTreeWinSize=20
~~~

### 介绍一下据说是最难装的vim插件[YouCompleteMe](https://github.com/Valloric/YouCompleteMe)
#### 1. 首先clone到~/.vim/bundle/
~~~bash
cd ~/.vim/bundle/
git clone https://github.com/Valloric/YouCompleteMe.git
git submodule update --init --recursive		# 下载子模块
~~~
#### 2. 安装
~~~
./install.py --clang-completer		# 支持C家族语言
~~~
- C# support: install Mono with Homebrew or by downloading the Mono Mac package and add --cs-completer when calling install.py.
- Go support: install Go and add --go-completer when calling install.py.
- JavaScript and TypeScript support: install Node.js and npm and add --ts-completer when calling install.py.
- Rust support: install Rust and add --rust-completer when calling install.py.
- Java support: install JDK8 (version 8 required) and add --java-completer when calling install.py.
#### 3. 在~/.vimrc中添加
~~~bash
Plugin Valloric/YouCompleteMe)
~~~
#### 4. 保存生效



[vim进阶 | 使用插件打造实用vim工作环境](https://juejin.im/post/5a38c37f6fb9a0450909a151)

[altercation/vim-colors-solarized](https://github.com/altercation/vim-colors-solarized)

[Valloric/YouCompleteMe](https://github.com/Valloric/YouCompleteMe#full-installation-guide)

[SpaceVim/SpaceVim](https://github.com/SpaceVim/SpaceVim)

[VundleVim/Vundle.vim](https://github.com/VundleVim/Vundle.vim)

[scrooloose/nerdtree](https://github.com/scrooloose/nerdtree)

[majutsushi/tagbar](https://github.com/majutsushi/tagbar)

# 主题配色
在~/.vimrc中添加
~~~bash
Plugin 'altercation/vim-colors-solarized'
~~~
在vim中执行
~~~bash
:PluginInstall
~~~

[altercation/vim-colors-solarized](https://github.com/altercation/vim-colors-solarized)

# 终端美化Oh my zsh

### 1. 安装zsh
~~~bash
sudo apt install zsh
~~~
### 2. 安装Oh-my-zsh
~~~bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
~~~
### 3. 修改Shell
~~~bash
chsh -s $(which zsh)
~~~
#### 注意
> 如果之前用的是bash，现在改成zsh之后，可能会导致之前的环境变量不可用，可以尝试在~/.zshrc中添加source ~/.bashrc,这样每当zsh启动时就会加载原来bash中的环境变量，然后执行source ~/.zshrc.如果执行source ~/.zshrc之后出现错误，说明zsh不能运行bash中的命令，只能手动将bashrc中的命令替换成同等功能的zshrc中的命令。

[robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

[Oh-My-Zsh! A Work of CLI Magic — Tutorial for Ubuntu](https://medium.com/wearetheledger/oh-my-zsh-made-for-cli-lovers-installation-guide-3131ca5491fb)

[oh-my-zsh/plugins/](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins)

[zsh oh-my-zsh 插件推荐](https://juejin.im/entry/5ae00e54f265da0b8635ea5c)

## 更换Oh-my-zsh主题

在这里我使用的是ys主题
在～/.zshrc里面配置
~~~bash
ZSH_THEME="ys"
~~~
但是由于ys主题主机名太长，所以这里在~/.oh-my-zsh/themes/ys.zsh-theme
~~~bash
PROMPT="
%{$terminfo[bold]$fg[blue]%}#%{$reset_color%} \
%(#,%{$bg[yellow]%}%{$fg[black]%}%n%{$reset_color%},%{$fg[cyan]%}%n) \
%{$fg[white]%}@ \
%{$fg[green]%}%m \  
%{$fg[white]%}in \
%{$terminfo[bold]$fg[yellow]%}%~%{$reset_color%}\
${hg_info}\
${git_info}\
 \
%{$fg[white]%}[%*] $exit_code
%{$terminfo[bold]$fg[red]%}$ %{$reset_color%}"
:NERDTreeToggle
~~~
将第1，3,4行删除
保存并退出
执行
~~~bash
source ~/.zshrc
~~~
