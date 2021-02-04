### [<主页](https://www.wangdekui.com/)

# Learning the vi and Vim Editors

# 旧的

```
i
a
o
I
A
O

:set nu
:vs
:sp
:q
:% s/java/python/g
:syntax on
:set hls         high light
:set incsearch    inc search

v
V
Ctrl+v

Switch
ngg
nG
:n Enter
vim +n filename

Buffer
d 
y
p

Shell
Ctrl+h delete last alphabet
Ctrl+w delete last word
Ctrl+u delete last row
Ctrl+e move end
Ctrl+a move ahead
Ctrl+b move back
Ctrl+l clear screen

gi fast edit in last position

h
j
k
l
w
W
e
E
b
B
f{char} find a char
        ,  prev
    ;  next
F{char} find a char reverse
        ,  prev
    ;  next

t{char} find a char before

0 ^  move to first char in a line
$ g_ move to last char in a line

0w the first word

() move in sentance
{} move in paragraph

gg to the start position
G  to the end position
Ctrl+o   fast backword

H   Head
M   Middle
L   Lower

Ctrl+u upword
Ctrl+f forword
zz  middle screen

ChangeWords
x
d
       daw
       diw
       dt{char}
       dd
       d0
       d$
       2dd
       4x
       xp             change position of two chars
r
s      sbstitude, delete and insert mode
       4s
R
S

c
       caw
       ct{char}
       ci[
       ci(
C      delete the row and insert mode

v
       vi"
p
       yiw p
       yy p
       dw p
       x p

/      find next ->
?      find next <-
n      find next
N      find forward
*      find current word next
#      find current word forward

Search and substitude
:[range]s[ubstitude]/{pattern}/{string}/[flags]
        range :10,20
          :%     all
    flags g      global
          c      confirm
          n      number
    :% s/self/this/g
    :1,6 s/self/this/g
    :1,6 s/self//n
    :% s/\<quack\>/jio/g        use regular expression

u      undo
Ctrl+r   undoundo

BufferChange
:ls
:b n     switch to n buffer index
:bpre[vious]
:bnext
:bfirst
:blast
:b buffer_name_or_file + tab

:e file_name       open another file
:bd[elete]

:sp == Ctrl+w s
:vs == Ctrl+w v

Ctrl+w w    switch tab
Ctrl+w h    left
Ctrl+w j    down
Ctrl+w k    up
Ctrl+w l    right

Ctrl+w H    all window the same height width
Ctrl+w _    max the alive window height
Ctrl+w |    max the alive window width
n Ctrl+w _  set the alive window height to n lines
n Ctrl+w |  set the alive window width to n lines

TabPage
ctrl+w T   change tabpage

:tabe[dit] {filename}
:tabc[lose]
:tabo[nly]

:tabn[ext] {N}    {N}gt
:tabn[ext]        gt
:tabn[ext]        gT

UserRegister
:a yiw
:b dd
:reg a
:reg b
""     nameless register, default
"+     copy to system clipboard
"0     copy to register0, which is copy using only
"%     current filename
".     last insert text

:echo has('clipboard')     look it has clipboard or not, 1 yes
:set clipboard=unnamed     set clipboard default register
Shift+Insert     Paste from Clipboard
:e!     clear change before save

Macro
normal mode      q    RECORD    q     END_RECORD
q{register}     RECODE in which register a to z
@{register}     PLAYBACK

qa I"  ESC  A"  ESC q
@a

Visual+Macro
        Visual select all row
    use : enter deline mode
    :'<,'>normal @a

Visual Only
        Visual select all row
    use : enter deline mode
    :'<,'>normal I"
    :'<,'>normal A"

:  <Ctrl+p>            use last order


AutoFill
Ctrl+n             default keyword
Ctrl+x Ctrl+n      current buffer keyword
Ctrl+x Ctrl+i      include file keyword
Ctrl+x Ctrl+j      tag file keyword
Ctrl+x Ctrl+k      dict find
Ctrl+x Ctrl+l      whole row fill

Ctrl+x Ctrl+f      filename
Ctrl+x Ctrl+p      word

Ctrl+n             next
Ctrl+p             previous

Ctrl+x Ctrl+o      Omni fill

:filetype on       open filetype
:set filetype      print filetype

:r[ead]
:r! echo %         current filename
:r! echo %:p       current filepath

ColorScheme
:colorscheme
:colorscheme Ctrl+d      show all colorscheme
:colorscheme darkblue

OpenFile
vim c.txt b.txt O        not zero, o


VIMRC
" 设置行号
set number
" 设置主题
colorscheme darkblue
" 语法高亮
syntax on
" F2进入粘贴模式
set pastetoggle=<F2>
" 高亮搜索
set hlsearch
" 折叠方式
set foldmethod=indent
" 一些方便的映射
set mapleader=","
set g:mapleader=","
" 使用jj进入normal模式
inoremap jj <Esc>`^
" 使用leader+w直接保存
inoremap <leader>w <Esc>:w<cr>
noremap <leader>w:w<cr>
" 切换buffer
nnoremap <slient> [b :bprevious<CR>
nnoremap <slient> [n :bnext<CR>
" use ctrl+j k l h switch window
noremap <C-h> <C-w>h
noremap <C-j> <C-w>j
noremap <C-k> <C-w>k
noremap <C-l> <C-w>l
" Sudo to write
cnoremap w! w !sudo tee & >/dev/null
" json 格式化
com! FormatJSON $!python3 -m json.tool
"  插件设置，这里使用了vim-plug
call plug#begin('~/.vim/plugged')
" 安装插件只需要把github地址放到这里重启后执行
Plug 'mhinz/vim-startify'
Plug 'scrooloose/nerdtree'

call Plug#end()

" 插件相关配置
" 禁止 startify自动切换目录
let g:startify_change_to_dir

"nerdtree
nmap ,v :NERDTreeFind<cr>
nmap ,g :NERDTreeToggle<cr>

" 定义函数SetTitle自动插入文件头
func SetTitle()
    fi &filetype == 'python'
        call setline(1, "\#!usr/bin/env python")
	call setline(2, "\# -*- coding:utf-8 -*-")
	normal G
	normal o
	normal o
	call setline(5, "if __name__ == '__main__':")
	call setline(6, "    pass")
    endif
endfunc
```

## [<主页](https://www.wangdekui.com/)