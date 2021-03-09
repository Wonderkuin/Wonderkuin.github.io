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

## [<主页](https://www.wangdekui.com/)