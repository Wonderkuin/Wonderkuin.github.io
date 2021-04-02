### [<主页](/index.html)

# Learning the vi and Vim Editors

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
200z Enter 把第200行移动到屏幕顶端 code中不管用


```

---