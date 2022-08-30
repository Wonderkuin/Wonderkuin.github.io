# Learning the vi and Vim Editors
## 学习Vi和Vim编辑器

---

```
What are the dark corners of Vim your mom never told you about?

something iw
something aw
diw
viw
yiw

viwu 转换小写
viwU 转换大写

de
bde
df something 删除直到本行某个字符

Ctrl+R and " 从内部寄存器" 中粘贴

在变更列表中移动
g; 向前
g, 向后
```

```
"gvim设置

set encoding=utf-8
set fileencodings=utf-8,chinese,latin-1
if has("win32")
 set fileencoding=chinese
else
 set fileencoding=utf-8
endif
"解决菜单乱码
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
"解决consle输出乱码
language messages zh_CN.utf-8
```

```
"显示编码格式
set fileencoding
"转换格式
set fileencoding=utf-8
"显示但不保存到文件
set encoding
```

---

### 文本编辑

```markdown
i 插入模式
Esc 命令模式
cw 修改一个词
: 进入ex模式
ZZ 保存并退出
:e! 重置
:q! 不保存退出
:w! 覆盖已有 文件名
:w 写入新文件中
:!rm Shell删除文件
:!dir Cmd查看文件
:!ls /tmp Shell查看临时文件夹
:sh 开一个新shell
CTRL_D 结束shell，回到vim
:!cmd 打开新Cmd
:pre :preserve 强制写入缓冲区

0 $ b w h j k l
4l
:set wm=10 wrapmargin 自动换行
:set nu 设置行号
G 42G gg 42gg
dd 删除一行
p 粘贴到后面
P 粘贴到前面
a 追加
r 替换
x 删除
I 插入到开头
A 插入到结尾
c2b 修改前面两个单词
c0 修改到开头
C c$ 修改到结尾
R 替换到结尾
s 删除并插入
S 整行删除并插入 和C不同，不从光标开始
~ 更改大小写
D 删除一行
2dd 删除两个一行
u 撤销
xp xP 修改位置
dd p 修改行位置
yy 2j p 修改行位置
yw y$ 4yy 拷贝
. 重复上一个命令
u 撤销上一个命令，光标不需要在同一行
U 撤销对同一行的编辑动作，只要光标还在这一行
秘诀：u可以撤销自己，回到上一次编辑的地方，只需要撤销，再撤销
Ctrl + R redo

A I o O s S R 都会进入插入模式
S == cc
50i* Esc 输入50个星号
25a*- Esc 输入25个*-
2r& || 替换为 &&

J 合并行 . 继续
Ctrl + j 合并行
ea 追加到单词后面

更改 替换 复制
cw dw yw
2cw 2dw 2yw 同 c2w d2w y2w
3cb 3db 3yb 同 c3b d3b y3b
cc dd yy或Y
c$或C d$或D y$
c0 d0 y0
r x或X yl或yh
5s 5x 5yl

+ 到下一行的第一个字符
[Enter] 同上
- 到上一行的第一个字符
e或E 到单词的结尾
w或W 往前一个单词
b或B 往后一个单词
$ 一行结尾
0 一行开头

p或P 往缓冲区放置文本
ZZ 保存并离开
:q! 不保存离开

i I a A o O S R J ~ . u U
```

---

### 快速移动

```markdown
^F 往前滚动一整屏
^B 往后滚动一整屏
^D 往下滚动半屏
^U 往上滚动半屏
^E 往上滚动一行
^Y 往下滚动一行

z Enter 将光标移到屏幕顶端并滚动屏幕
z . 将光标移到屏幕中心并滚动屏幕
z- 将光标移动到屏幕底端并滚动屏幕
200z Enter 把第200行移动到屏幕顶端

H 移到屏幕顶端的行
M 移到屏幕中央的行
L 移到屏幕底端的行
nH 移到屏幕顶端向下第n行
nL 移到屏幕顶端向上第n行


在当前行移动
^ 移到当前行第一个非空格处
n| 移到当前行的第n列
0 行首
$ 行尾


根据文本块来移动
e 单词结尾
E 单词结尾 忽略标点符号
( 当前句子开头
) 下一个句子开头
{ 当前段开头
} 下一段开头
[[ 当前节开头
]] 下一节开头


搜索
/pattern 搜索
?pattern 往回搜索
n 相同方向下一个
N 相反方向下一个
:set nowrapscan 取消循环搜索
d?set 从光标位置开始向前删除到出现set的地方
d/set 同上

在当前行搜索
fx 本行中下一个出现x的地方
Fx 本行中上一个出现x的地方
tx 本行中下一个出现x的前一个字符
Tx 本行中上一个出现x的前一个字符
; 重复上一个搜索命令 方向相同
, 重复上一个搜索命令 方向相反


根据行号移动
CTRL-G 显示信息 行号，文件总行数，当前百分比
G 跳转
`` 跳回
'' 跳回，行首
```

---

### 越过基础的藩篱

```markdown
更改	删除	复制	从光标位置到...
cH		dH		yH		屏幕顶端
cL		dL		yL		屏幕底端
c+		d+		y+		下一行
c5|		d5|		y5|		本行第五列
2c)		2d)		2y)		向下第二个句子
cG		dG		yG		文件结尾
c13G	d13G	y13G	第13行
c{		d{		y{		上一段
c/ptn	d/ptn	y/ptn	ptn=>pattern模式
cn		dn 		yn		下一个模式
```

```shell
vi file					打开文件
vi +n file				在第n行打开文件
vi + file				在最后一行打开文件
vi +/pattern file		在第一个出现pattern的地方打开文件

# 根据POSIX标准 vi应该用 -c command，而不是 +c command 但是为了向下兼容，两种写法都可以

vi +/screen practice
vi +/"you make" # 如果包含空格
vi +/you\ make  # 同上

# 如果决定要改变文件内容，可用w!覆盖掉只读模式
vi -R file #只读模式 这是shell的限制，不是vi的限制
view file  #通常view只是对vi的链接
```

```shell
#恢复缓冲区

ex -r
vi -r
vi -r practice
:pre 强制系统保存缓冲区 如果没有写入权限而不能保存时，这很有用

最后一个 d x y 的内容会保存到缓冲区中，可用访问这些缓冲区并使用 p
最后9次的删除操作，保存到默认缓冲区 1~9 ?? vscode不会默认保存
小规模的删除，如一行中的一部分，并不会保存编号的缓冲区中，只能通过p P恢复
自定义缓冲区 a~z

"2p  #恢复删除
1pu.u.u #缓冲区编号会自动增加 ?? vscode 不可用

"dyy 将当前行拖拽到缓冲区d中
"a7yy 将后续7行拖拽到缓冲区a中

"dP 将缓冲区d的内容放置在光标前
"ap 将缓冲区a的内容放置在光标后

"a5dd 将删除的5行保存到缓冲区a中
如果用大写字母指定缓冲区名称，则拖拽或删除的文本会被附加到当前缓冲区中
因此可用选择性地做移动与复制

"zd) 删除从光标处开始到所在句子结尾处的内容，并将内容保存在z缓冲区中
2) 往前移两个句子
"Zy) 将下一个句子添加到缓冲区z中。你可以继续在某个命名缓冲区中添加更多的文本。
但请注意，如果一时忘记，在拖拽或者删除文本到缓冲区时没用大写字母指定缓冲区名称，则
会把缓冲区的内容覆盖掉， 前面已有的文本都会消失
?? 不会用
```

```markdown
对一处做标记 标记只有在当前vi会话中有用，并不会储存在文件中
mx 将当前位置标记成x
'x 将光标移到标记x所在行的第一个字符
`x 将光标移到以x标记的字符
`` 在移动位置之后，回到上一个标记或上下文的确切位
'' 回到上一个标记或上下文所在行的开头
```

---

# 第五章 ex编辑器概述

```markdown
TODO
```