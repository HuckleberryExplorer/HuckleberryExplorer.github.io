---
layout: default
title:  ideavim配置
date:   2023-05-02 13:07:00 +0800
---

[返回主页](../)

自己用的比较顺手的ideavim配置

# 依赖插件

- IdeaVim
- AceJump
- IdeaVim-EasyMotion
- IdeaVimExtension
- Which-Key

# .ideavimrc

``` vim
Plug 'tpope/vim-surround'
Plug 'preservim/nerdtree'
Plug 'easymotion/vim-easymotion'

let mapleader=' '

set hlsearch
set incsearch
set ignorecase
set smartcase
set showmode
set number
set relativenumber
set scrolloff=3
set history=100000
set clipboard=unnamed
set keep-english-in-normal

" 清除高亮
nnoremap <Leader>sc :nohlsearch<CR>

" 保存关闭
nnoremap <Leader>q :q<CR>
nnoremap <Leader>Q :qa!<CR>

" 上下翻页
nnoremap <Leader>d <C-d>
nnoremap <Leader>u <C-u>

" 退出可视模式
vnoremap v <Esc>

" Redo
nnoremap U <C-r>

" 复制单个单词到寄存器a并标记到o
nnoremap <Leader>y mo"+yiw"ayiw

" 剪贴单个单词到寄存器a并标记到o
nnoremap <Leader>x mo"+yiw"ayiwdiw

" 删除单个字符串并粘贴寄存器a的内容并来回跳标记o和p
nnoremap <Leader>v mpviw"ap'o'p

" 内置快捷键Alt+F1
nnoremap <Leader>m :action SelectIn<CR>

" 内置快捷键Ctrl+E
nnoremap <Leader>e :action RecentFiles<CR>

" 生成方法
nnoremap <Leader>cc :action Generate<CR>

" 向上翻页
nnoremap <Leader>h <S-h>zz

" 向下翻页
nnoremap <Leader>l <S-l>zz

" 窗口操作
nnoremap <Leader>ww <C-W>w
nnoremap <Leader>wd <C-W>c
nnoremap <Leader>wj <C-W>j
nnoremap <Leader>wk <C-W>k
nnoremap <Leader>wh <C-W>h
nnoremap <Leader>wl <C-W>l
nnoremap <Leader>ws <C-W>s
nnoremap <Leader>w- <C-W>s
nnoremap <Leader>wv <C-W> v
nnoremap <Leader>w\| <C-W>v

" 切换标签
nnoremap tn gt
nnoremap tp gT

" search everywhere
nnoremap <Leader>se :action SearchEverywhere<CR>

" 查找用法
nnoremap <Leader>fu :action FindUsages<CR>

" 打断点
nnoremap <Leader>bb :action ToggleLineBreakpoint<CR>

" 查看所有断点
nnoremap <Leader>br :action ViewBreakpoints<CR>

" DEBUG启动
nnoremap <Leader>cd :action ChooseDebugConfiguration<CR>

" 跳转到Action
nnoremap <Leader>ga :action GotoAction<CR>

" 跳转到类
nnoremap <Leader>gc :action GotoClass<CR>

" 跳转到声明
nnoremap <Leader>gd :action GotoDeclaration<CR>

" 跳转到文件
nnoremap <Leader>gf :action GotoFile<CR>

" 跳转到实现类
nnoremap <Leader>gi :action GotoImplementation<CR>

" 显示当前文件路径
nnoremap <Leader>fp :action ShowFilePath<CR>

" 修改所有的关联名
nnoremap <Leader>re :action RenameElement<CR>

" 修改当前文件的文件名
nnoremap <Leader>rf :action RenameFile<CR>

" 显示用法
nnoremap <Leader>su :action ShowUsages<CR>

" 关闭活动显示面板
nnoremap <Leader>tc :action CloseActiveTab<CR>

" 代码格式化
nnoremap <Leader>fm :action ReformatCode<CR>

" 打开命令管理器
nnoremap <Leader>tl Vy<CR>:action ActivateTerminalToolWindow<CR>
vnoremap <Leader>tl y<CR>:action ActivateTerminalToolWindow<CR>

" 查找字符串
nnoremap / :action Find<CR>

" vim自带的搜索
nnoremap <Leader>/ /

" 添加注释
nnoremap <Leader>;; :action CommentByLineComment<CR>

" 改变视图
nnoremap <Leader>cv :action ChangeView<CR>

" 跳转到标识
nnoremap <Leader>gs :action GotoSymbol<CR>

" 检查代码
nnoremap <Leader>ic :action InspectCode<CR>

" 显示右键菜单
nnoremap <Leader>pm :action ShowPopupMenu<CR>

" 正常启动工程
nnoremap <Leader>rc :action ChooseRunConfiguration<CR>

```

