---
layout:     post
title:      例子VIM配置文件
subtitle:   日常维护
date:       2019-05-30
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Linux
    - Shell
    - VIM
---

>Linux 知识库

# 进入主目录执行下面脚本

```shell

cd ~
mkdir .vim
cd .vim
vi vimrc

```

# 在vimrc文件中粘贴下面内容

```vim

"vimim 相关
let g:vimim_enable_static_enu=1
"let g:vimim_enable_sexy_input_style=1
" 关闭中文标点
let g:vimim_disable_chinese_punctuation=1
"中英文之间不加空格
let g:vimim_disable_seamless_english_input=1
 
 
set dictionary-=/etc/funclist.txt dictionary+=/etc/funclist.txt
"Use the dictionary completion
set complete-=k complete+=k
 
"自动补全..
set completeopt=longest,menu
 
 
imap <C-L> <C-x><C-o>
set nu
autocmd FileType python set complete+=k~/.vim/tools/pydiction
 
filetype plugin indent on
set nocp
filetype plugin on
set helplang=cn
map <F6> :Project<CR>
"autocmd BufNewFile *.py 0r ~/.vim/template/simple.py
"
"
set ignorecase
set shiftwidth=4
set softtabstop=4
set tabstop=4
 
 
set nocompatible    " Use Vim defaults (much better!)
"set ai  " always set autoindenting on
"set backup  " keep a backup file
set viminfo='200,f1,<1500   " read/write a .viminfo file, don't store more
" than 50 lines of registers
set history=1500     " keep 50 lines of command line history
set ruler    " show the cursor position all the time
 
if has("autocmd")
    filetype plugin indent on
    autocmd FileType python set omnifunc=pythoncomplete#Complete
    autocmd FileType ruby set omnifunc=rubycomplete#Complete
    autocmd FileType c set omnifunc=ccomplete#Complete
    autocmd FileType ada set omnifunc=adacomplete#Complete
    autocmd FileType php set omnifunc=phpcomplete#CompletePHP
    autocmd FileType javascript set omnifunc=javascriptcomplete#CompleteJS
    autocmd FileType html set omnifunc=htmlcomplete#CompleteTags
    autocmd FileType css set omnifunc=csscomplete#CompleteCSS
    autocmd FileType xml set omnifunc=xmlcomplete#CompleteTags
    autocmd FileType sql set omnifunc=sqlcomplete#Complete
endif
 
 
set ru
"该命令打开 VIM 的状态栏标尺。 默认情况下， VIM 的状态栏标尺在屏幕底部， 它能即时显示当前光标所在位置在文件中的行号、列号， 以及对应的整个文件的百分比。 打开标尺可以给文件的编辑工作带来一定方便。
set nocp
"该命令指定让 VIM 工作在不兼容模式下。 在 VIM 之前， 出现过一个非常流行的编辑器叫 vi。 VIM 许多操作与 vi 很相似，但也有许多操作与 vi 是不一样的。 如果使用“:set cp”命令打开了兼容模式开关的话， VIM 将尽可能地模仿 vi 的操作模式。
"也许有许多人喜欢“最正统的 vi”的操作模式， 对于初学者来说， vi 里许多操作是比较不方便的。
"举一个例子， VIM 里允许在 Insert 模式下使用方向键移动光标， 而 vi 里在 Insert 模式下是不能移动光标的， 必须使用 ESC 退回到 Normal 模式下才行。
"再举一个例子， vi 里使用 u 命令可以撤消一次先前的操作， 再次按下 u 时， 将撤消“撤消”这个动作本身，也就是我们常说的“重复”(redo)。 而 VIM 里可以使用 u 命令撤消多步操作， “重复”使用的快捷键是 Ctrl + R。
"使用兼容模式后， VIM 将放弃这些新的功能， 尽量模仿 vi 的各种操作方式。 只有在不兼容模式下， 才能更好地发挥 VIM 自身的特点。 Bram 爷爷强烈推荐大家使用 VIM 的不兼容模式， 滇狐也是这样推荐的。 请务必在你的 .vimrc 中的第一行写上： “set nocp”。
set hls
"搜索时高亮显示被找到的文本。 该指令的功能在 vimtutor 中已经有过介绍， 这里就不多说了。 其实似乎许多人并不喜欢这个功能。
"set is
"搜索时在未完全输入完毕要检索的文本时就开始检索。 vimtutor 对该命令也有过介绍， 滇狐并不喜欢这个功能， 因此滇狐自己的配置文件里是没有这条命令的。 但是周围有朋友很喜欢这个， 因此滇狐还是将它列在这里。
syntax on
"打开关键字上色。 进行程序设计的朋友应该都知道关键字上色是什么东西， 因此这里就不多说了。 不进行程序设计的朋友不妨也打开这个功能， 虽然不一定能够用得着， 但这个功能其实也是很好玩的。
set backspace=indent,eol,start
"设想这样一个情况： 当前光标前面有若干字母， 我们按下 i 键进入了 Insert 模式， 然后输入了 3 个字母， 再按 5 下删除(Backspace)。 默认情况下， VIM 仅能删除我们新输入的 3 个字母， 然后喇叭“嘟嘟”响两声。 如果我们“set backspace=start”， 则可以在删除了新输入的 3 个字母之后， 继续向前删除原有的两个字符。
"再设想一个情况： 有若干行文字， 我们把光标移到中间某一行的行首， 按 i 键进入 Insert 模式， 然后按一下 Backspace。默认情况下， 喇叭会“嘟”一声， 然后没有任何动静。 如果我们“set backspace=eol”， 则可以删除前一行行末的回车，也就是说将两行拼接起来。
"当我们设置了自动缩进后， 如果前一行缩进了一定距离， 按下回车后， 下一行也会保持相同的缩进。 默认情况下， 我们不能在 Insert 模式下直接按 Backspace 删除行首的缩进。 如果我们“set backspace=indent”， 则可以开启这一项功能。
"上述三项功能， 你可以根据自己的需要， 选择其中一种或几种， 用逗号分隔各个选项。 建议把这三个选项都选上。
set whichwrap=b,s,<,>,[,]
"默认情况下， 在 VIM 中当光标移到一行最左边的时候， 我们继续按左键， 光标不能回到上一行的最右边。 同样地， 光标到了一行最右边的时候，我们不能通过继续按右跳到下一行的最左边。 但是， 通过设置 whichwrap 我们可以对一部分按键开启这项功能。如果想对某一个或几个按键开启到头后自动折向下一行的功能， 可以把需要开启的键的代号写到 whichwrap 的参数列表中，各个键之间使用逗号分隔。 以下是 whichwrap 支持的按键名称列表：
 
"    * b
"      在 Normal 或 Visual 模式下按删除(Backspace)键。
"    * s
"      在 Normal 或 Visual 模式下按空格键。
"    * h
"      在 Normal 或 Visual 模式下按 h 键。
"    * l
"      在 Normal 或 Visual 模式下按 l 键。
"    * <
"      在 Normal 或 Visual 模式下按左方向键。
"   * >
"      在 Normal 或 Visual 模式下按右方向键。
"    * ~
"      在 Normal 模式下按 ~ 键(翻转当前字母大小写)。
"    * [
"      在 Insert 或 Replace 模式下按左方向键。
"    * ]
"      在 Insert 或 Replace 模式下按右方向键。
set encoding=utf-8
"设置当前字符编码为 UTF-8。 UTF-8 是支持字符集最多的编码之一， 在 UTF-8 下进行工作， 会带来许多方便之处。 由于 VIM 在运行过程中切换 encoding 会造成许多问题， 如提示信息乱码、 register 丢失等， 因此强烈建议大家在启动 VIM 的时候把 encoding 设置为 UTF-8， 在编辑非 UTF-8 的文件时， 通过 fileencoding 来进行转码。
set langmenu=zh_CN.UTF-8
"使用中文菜单， 并使用 UTF-8 编码。 如果没有这句的话， 在非 UTF-8 的系统， 如 Windows 下， 用了 UTF-8 的 encoding 后菜单会乱码。
"language message en_US.UTF-8
"使用中文提示信息， 并使用 UTF-8 编码。 如果没有这句的话， 在非 UTF-8 的系统， 如 Windows 下， 用了 UTF-8 的 encoding 后系统提示会乱码。
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
"设置编码的自动识别。 关于这条设置的详细含义， 请参看这篇文章。
"set ambiwidth=double
"防止特殊符号无法正常显示。 在 Unicode 中， 许多来自不同语言的字符， 如果字型足够近似的话， 会把它们放在同一个编码中。但在不同编码中， 字符的宽度是不一样的。 例如中文汉语拼音中的 ā 就很宽， 而欧洲语言中同样的字符就很窄。 当 VIM 工作在 Unicode 状态时， 遇到这些宽度不明的字符时， 默认使用窄字符， 这会导致中文的破折号“——”非常短， 五角星“★”等符号只能显示一半。因此， 我们需要设置 ambiwidth=double 来解决这个问题。
filetype plugin indent on
"开启文件类型自动识别， 启用文件类型插件， 启用针对文件类型的自动缩进
set smarttab
"当使用 et 将 Tab 替换为空格之后， 按下一个 Tab 键就能插入 4 个空格，
"但要想删除这 4 个空格， 就得按 4 下 Backspace， 很不方便。 设置 smarttab
"之后， 就可以只按一下 Backspace 就删除 4 个空格了。
"set spell
"打开拼写检查。 拼写有错的单词下方会有红色波浪线， 将光标放在单词上， 按 z= 就会出现拼写建议， 按 ]s 可以直接跳到下一个拼写错误处。
 
 
"set tw=78
"设置光标超过 78 列的时候折行。
set lbr
"不在单词中间断行。 设置了这个选项后， 如果一行文字非常长， 无法在一行内显示完的话， 它会在单词与单词间的空白处断开， 尽量不会把一个单词分成两截放在两个不同的行里。
set fo+=mB
"打开断行模块对亚洲语言支持。 m 表示允许在两个汉字之间断行， 即使汉字之间没有出现空格。 B 表示将两行合并为一行的时候， 汉字与汉字之间不要补空格。 该命令支持的更多的选项请参看用户手册。
"
"
"
 
" c/c++
set sm
"显示括号配对情况。 打开这个选项后， 当输入后括号 (包括小括号、中括号、大括号)
"的时候， 光标会跳回前括号片刻， 然后跳回来， 以此显示括号的配对情况。
set ai
"打开普通文件类型的自动缩进。 该自动缩进不如 cindent 智能， 但它可以为你编辑非
"C/C++ 文件提供一定帮助。
set selectmode=
"不使用 selectmode。
 
set keymodel=
"不使用“Shift + 方向键”选择文本， “Shift + 方向键”代表向指定方向跳一个单词。 如果你喜欢这项功能的话， 可以使用“set keymodel=startsel,stopsel”打开它。
set keymodel=startsel,stopsel
 
set wildmenu
"在命令模式下使用 Tab 自动补全的时候，
"将补全内容使用一个漂亮的单行菜单形式显示出来。
set sw=4
"自动缩进的时候， 缩进尺寸为 4 个空格。
set ts=4
"Tab 宽度为 4 个字符。
set et
"编辑时将所有 Tab 替换为空格。
"该选项只在编辑时将 Tab 替换为空格， 如果打开一个已经存在的文件， 并不会将已有的 Tab 替换为空格。 如果希望进行这样的替换的话， 可以使用这条命令“:retab”。
retab
"set paste
"set nopaste
set pastetoggle=<F3>
"很多兄弟都碰到过这样一个问题，在vim中粘贴代码有时会自动增加缩进，造成代码排版
"的混乱。如何让它不缩进,保持原格式?其实vim有一个paste开关。
"输入 :set paste
"需要关闭时
"输入:set nopaste
"我是在vimrc中加入了：
"set pastetoggle=<F3>
"这样就可以用F3来切换了。
if (has("win32"))
 
     "-------------------------------------------------------------------------
     " Win32
     "-------------------------------------------------------------------------
    if (has("gui_running"))
        set guifont=Bitstream_Vera_Sans_Mono:h9:cANSI
        set guifontwide=NSimSun:h9:cGB2312
    endif
 
else
    if (has("gui_running"))
        set guifont=Bitstream\ Vera\ Sans\ Mono\ 9
    endif
endif
 
"如果去掉这一行,默认值set mouse=a就生效了,这时不能用鼠标选中复制.
set mouse=v
"打开文件时自动到达上次浏览时的位置
" Only do this part when compiled with support for autocommands
if has("autocmd")
" " In text files, always limit the width of text to 78 characters
autocmd BufRead *.txt set tw=78
" " When editing a file, always jump to the last cursor position
autocmd BufReadPost *
            \ if line("'\"") > 0 && line ("'\"") <= line("$") |
            \ exe "normal g'\"" |
            \  endif
 endif
 
au BufReadPost * if line("'\"") > 0|if line("'\"") <= line("$")|exe("norm '\"")|else|exe "norm $"|endif|endif
 
 
 
"
"其中字符串 “string” 规定了什么要储存。该字符串的语法为一个选项字符跟一个参数.
""选项和参数组成的对子之间由逗号分隔.
""来看一下你可以怎样构建你自己的 viminfo 字符串。首先，选项 ‘ 用于规定你为多
"少个文件保存标记 (a-z)。为此选项挑一个美妙的偶数 (比如 1000). 你的命令现在看
"起来像这样:
"
":set viminfo=’1000
"
"选项 f 控制是否要储存全局标记 (A-Z 和 0-9)。如果这个选项设为 0，那么什么也不
"存储。如果设为 1 ，或你对 f 选项不作规定, 那么标记就被存储. 你要这个功能, 现
"在你有了:
"
":set viminfo=’1000,f1
"
"选项 < 控制着每个寄存器内保存几行文本。默认情况下，所有的文本行都被保存. 如果
"设为 0，则什么也不保存。为了避免成千上万行文本被加入你的信息文件 (那些文本可能
"永远也没用，徒然使 Vim 起动得更慢), 你采用 500 行的上限:
"
":set viminfo='1000,f1,<500
"
"你也许用得着的其它选项:
": 保存命令行历史记录内的行数
"@ 保存输入行历史记录内的行数
"/ 保存搜索历史记录内的行数
"r 可移介质，其中没有任何标记存入 (可用多次)
"! 以大写字母开头的全局变数，并且不含有小写字母
"h 起动时解除选项 'hlsearch' 的高亮度显示
"% 缓冲列表 (只有当不带参数起动 Vim 时才还原)
"c 用编码 'encoding' 转换文本
"n 用于 viminfo 文件的名称 (必须为最后一项选项)
set viminfo='1000,f1,<5000

let g:SnipeMateAllowOmniTab = 1

"ino <c-j> <c-r>=TriggerSnippet()<cr>
"snor <c-j> <esc>i<right><c-r>=TriggerSnippet()<cr>
"
"filetype plugin on
ino <tab> <c-r>=TriggerSnippet()<cr>
snor <tab> <esc>i<right><c-r>=TriggerSnippet()<cr>
"设置缩进
set sw=4
set ts=4


" history文件中需要记录的行数
set history=1000
" 带有如下符号的单词不要被换行分割
set iskeyword+=_,$,@,%,#,-
" 高亮显示普通txt文件（需要txt.vim脚本）
au BufRead,BufNewFile *  setfiletype txt
"不需要保持和 vi 非常兼容
set nocompatible
"执行 Vim 缺省提供的 .vimrc 文件的示例，包含了打开语法加亮显示等最常用的功能
"source $VIMRUNTIME/vimrc_example.vim
" 使backspace正常处理indent, eol, start等
set backspace=2
" 允许backspace和光标键跨越行边界
set whichwrap+=<,>,h,l
"在输入命令时列出匹配项目，也就是截图底部的效果
set wildmenu
set nocindent
"打开自动缩,继承前一行的缩进方式，特别适用于多行注释 进
set autoindent
"默认情况下手动折叠
set foldmethod=manual
"显示行号（否：nonumber）
set nu
"不自动换行(否：wrap)
set nowrap
"缺省不产生备份文件
set nobackup
" 不要生成swap文件，当buffer被丢弃的时候隐藏它
setlocal noswapfile
"set bufhidden=hide 
"在输入括号时光标会短暂地跳到与之相匹配的括号处，不影响输入
set showmatch
" 匹配括号高亮的时间（单位是十分之一秒）
set matchtime=5
" 在搜索时，输入的词句的逐字符高亮（类似firefox的搜索）
set incsearch
" 不要闪烁
set novisualbell
"正确地处理中文字符的折行和拼接
set formatoptions+=mM
"文件 UTF-8 编码
set fileencodings=utf-8

let g:SuperTabRetainCompletionType=2
let g:SuperTabMappingBackward="<tab>"

nmap <C-M> <ESC>:NERDTreeToggle<RETURN>
"let g:SuperTabDefaultCompletionType="<C-X><C-O>"
"------------ renlu split ----------

"设置工作目录
function! CHANGE_CURR_DIR()
let _dir = expand("%:p:h")
exec "cd " . _dir
unlet _dir
endfunction
autocmd BufEnter * call CHANGE_CURR_DIR()




" 窗口区域切换,Ctrl+↑↓←→  来切换
imap <silent> <C-K> <esc><C-W><C-W><CR>
nmap <silent> <C-K> <esc><C-W><C-W><CR>
imap <silent> <F7> <esc>:NERDTreeToggle<CR>
nmap <silent> <F7> :NERDTreeToggle<CR>
imap <silent> <C-Q> <esc>:q!<CR><esc>:q!<CR>
colorscheme desert
set guifont=Monaco:h18



" 窗口区域切换,Ctrl+↑↓←→  来切换
"imap <silent> <C-left> <esc><C-W><left>
"vmap <silent> <C-left> <esc><C-W><left>
"nmap <silent> <C-left> <C-W><left>
"imap <silent> <C-right> <esc><C-W><right>
"vmap <silent> <C-right> <esc><C-W><right>
"nmap <silent> <C-right> <C-W><right>
imap <silent> <D-left> <esc><C-W><left>
vmap <silent> <D-left> <esc><C-W><left>
nmap <silent> <D-left> <C-W><left>
imap <silent> <D-h> <esc><C-W><left>
vmap <silent> <D-h> <esc><C-W><left>
nmap <silent> <D-h> <C-W><left>
imap <silent> <D-l> <esc><C-W><right>
vmap <silent> <D-l> <esc><C-W><right>
nmap <silent> <D-l> <C-W><right>
imap <silent> <C-up> <esc><C-W><up>
vmap <silent> <C-up> <esc><C-W><up>
nmap <silent> <C-up> <C-W><up>
imap <silent> <C-down> <esc><C-W><down>
vmap <silent> <C-down> <esc><C-W><down>
nmap <silent> <C-down> <C-W><down>

" delete hack
nmap <silent> <del> "_x
vmap <silent> <del> "_x
nmap <silent> dd V<del>
nmap <silent> dw viw<del>
vmap <silent> dw iw<del>

set wrap
set fdm=manual
let Tlist_Ctags_Cmd="/usr/bin/ctags"




map fg : Dox<cr>
let g:DoxygenToolkit_authorName="renlu.xu"
let g:DoxygenToolkit_licenseTag="My own license\<enter>"
let g:DoxygenToolkit_undocTag="DOXIGEN_SKIP_BLOCK"
let g:DoxygenToolkit_briefTag_pre = "@brief\t"
let g:DoxygenToolkit_paramTag_pre = "@param\t"
let g:DoxygenToolkit_returnTag = "@return\t"
let g:DoxygenToolkit_briefTag_funcName = "no"
let g:DoxygenToolkit_maxFunctionProtoLines = 30

"对 NERD_commenter的设置
let NERDShutUp=1
"支持单行和多行的选择，//格式
map <c-h> ,c<space>

if(executable('ctags'))
"silent! execute "!ctags -R --c-types=+p --fields=+S *"
"silent! execute "!ctags  --c++-kinds=+p --fields=+iaS --extra=+q ."
endif


:set pastetoggle=<F3> "粘贴模式开启和关闭

```