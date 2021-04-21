### [<主页](/index.html)

## Aseprite 文档

#### 快捷键

```markdown
铅笔
Left Click 前景色绘画
Right Click 背景色绘画

123456
鼠标滚轮
变焦

Z
缩放工具

X
交换前后颜色

Alt
+Left Click 前景
+Right Click 后景

Space
+Left Click
+Drag  移动画布

Tab 显示隐藏Timeline
Ctrl 选择移动工具
Shift 查看边距

Ctrl + N 新建文件
Shift + N 新建图层
Alt + N 新建帧
Ctrl + Alt + N 选中新建

图层 帧 Cel
单击拖动 移动
同时按住ctrl或alt 复制

F3
洋葱皮

F4
修改调色板
```

```markdown
绘画工具列表

B
Pencil 铅笔

L
Line 直线
Shift + L
Curve 曲线

U
Rectangle 矩形
Shift + U
Ellipse 椭圆

D
Countour 
Shift + D
Polygon 多边形

其他工具

E
Eraser 橡皮擦

Alt
I
Eyedropper 吸管

M
Rectangluar Marguee 矩形选框

Ctrl
V
移动工具

Shift + C
切片

Alt + B
新建空白frame

Alt + D
复制Cels

Alt + Shift + D
或
Alt + M
创建到当前单元格（或当前时间线选择）的链接到下一个位置/帧。

标签操作
F2
两次
第一次创建一个标签
第二次显示标签属性

```

#### 精灵结构

```markdown
像素单位大小

颜色模式， RGB图像 索引图像。无法在同一子画面中混合使用两种模式

颜色配置文件，指示RGB在什么颜色空间

图层， 时间轴， 两种类型的层：不透明子画面的背景层 透明层。
子画面只能包含一个背景层，但可以包含多个透明层

动画帧，每个帧都有一个持续时间。

每个layer/帧的交点都成为cel，其中包含最终可绘制的图像
```

#### 颜色

```markdown
color mode
红绿蓝
RGBA 背景层没有alpha成分，因此它始终是不透明的
索引
每个像素都是引用调色板颜色的数字，调色板最多支持256色
如果修改调色板颜色，所有引用该色的像素都会改变外观
透明层：需要特殊的索引来充当透明颜色，通常索引是0，可以在Sprite>Properties改
灰阶
更像是RGBA，但是只有两个通道Value和Alpha
0表示黑色，255表示白色
alpha通道和RGBA模式下完全相同

color profile
颜色空间保留图像的RGB值，例如sRGB色彩空间

transparent color (only in indexed images)

```

#### 时间轴

```markdown
在文件之间复制
Edit>Copy
Edit>Paste

配置时间轴
Onion Skinning洋葱皮
一次看到几个帧
F3 或者 两个正方形框的按钮 激活/关闭
可以指定要查看的前后几个帧
指定色调
```

#### 精灵大小

```markdown
画布大小
Sprite>Canvas Size
宽高左右上下

Sprite>Crop
使用当前选择范围

Sprite>Trim
自动删掉透明边框

精灵大小
Sprite>Sprite Size
可以缩放精灵
```

#### 调色板

```markdown
F4 或 Edit Palette按钮
修改调色板

索引图像上，颜色栏显示了所有可用颜色

RGB图像，可以使用调色板不存在的颜色

不在编辑模式
没有的颜色，点击红色感叹号，可以加入新颜色

用于绘制的背景颜色，也用于清除图层
Edit>Clear
Frame>New Empty Frame
Layer>Background from Layer
```

#### 颜色配置文件

```markdown
Internet图像通常使用sRGB颜色空间
PNG和JPEG可以使用RGB色域和伽马校正来嵌入特定的ICC颜色配置文件

在 Sprite Properties 中指定或转换当前sprite的颜色配置文件

Edit>Preferences>Color管理颜色配置文件

Ctrl + P
Sprite>Properties
更改Sprite属性
索引颜色模式更改透明颜色
更改像素宽高比
分配或转换颜色配置文件
```

#### 图层

```markdown
Shift + P
修改图层名称

背景层
背景层是无法移动的不透明层 无alpha/透明组件
在文件，新建中选择不透明颜色，打开png不包含alpha成分的文件
会默认创建背景层
选择背景层的一部分并 清除 时，所选内容用背景色清除

透明层
所有具有alpha通道的图层都称为透明层
可以在同一个子画面中包含多个此类图层
可以使用时间轴堆叠它们，可以使用移动工具来移动图层
选择透明层的一部分并 清除 时，所选内容用透明色清除

Background from Layer
如果没有背景层，可以使用 Layers>Background from Layer
将任何透明图层转换为背景
所有透明像素将使用活动背景色填充

Layer from Background
如果要将背景转换为透明图层，例如，使用移动工具进行移动
可以使用 Layers>Background from Layer
```

#### 连续层

```markdown
普通层，具有不连续的cel，新的cel将被创建为未连接
连续层，以链接方式创建新的单元格

通常 对于背景图层 (具有静态内容) ，首选连续cel
对于每帧具有不同的cel的图层，选用不连续模式

链接Cels
当两个cel共享图像和xy坐标时，将它们链接在一起
修改一个，全都修改
创建链接Cels，必须在连续的图层中复制cel
单击右键可以取消链接
```

#### Cel

```markdown
Cel 特定xy坐标处的特定帧和图层中的一个图像

帧和cel的区别 帧是特定时间内所有图层的cel集合

更改不透明度
右键 Cel Properties
在RGB图像上，每个Cel都有自己的不透明度级别
可以使用状态栏滑块修改不透明度
```

#### 移动工具

```markdown
V
Ctrl

不能移动背景层
使用 Shift 锁定X或Y轴

使用 Ctrl + Left click时
快速选择和移动图层

同时选中多个Cels和多个Frame
框住多个Cels，在编辑区域移动
```

#### 吸管工具

```markdown
I
Alt click

上下文栏 Pick
HSV HSL RGB 等等选项

所有层 当前层 第一参考层

可以配置成右键单击选择颜色
Edit>Preferences>Editor
```

#### 墨水

```markdown
墨水会更改 活动工具 的绘制方式
默认是 简单墨水

Simple Ink 简单墨水
1 如果前景色是不透明的 将使用给定的不透明颜色进行绘制
2 如果颜色具有alpha通道，将颜色与图层表面进行合成
3 如果颜色是透明的 该工具类似橡皮擦

Alpha Compositing Alpha合成
将 前景色 与图层表面合并，取决于前景色的alpha值
1 如果 alpha == 255 ，前景色完全不透明
2 如果 alpha == 128 ，前景色与图层表面颜色合并50%
3 如果 alpha == 0 ，绘画无效

Copy Alph+Color 复制Alpha + 颜色
将 具有alpha值的活动前景色 替换 层表面像素
不进行任何alpha合成 仅仅采用活动颜色并将其精确放置在目标像素中
例如：alpha == 128， 最终颜色与alpha==128的前景色相同，忽略表面图层

Lock Alpha 锁定Alpha
保留来自图层表面的原始alpha值，仅将RGB颜色分量替换。

Shading 底纹
像素艺术专用Ink，可用于在精灵中创建阴影
可以使用 左键 右键 在渐变之间移动颜色
1 绘制一些东西 以添加基本颜色的光或阴影
2 选择 墨水模式 从调色板选择一组颜色，包括先前选择的基础颜色
此渐变将充当阴影和光
3 可以使用左键单击将颜色移动到渐变的左侧
4 可以使用右键单击将颜色向右移动
```

---

#### 动画

```
timeline 控制 frames layers cels

工作流
1 画第一帧
2 添加新帧 Alt+N
3 左右箭头或 , . 切换帧
4 预览动画 Enter 或播放按钮
5 标记一系列帧 添加标签 看哪几个帧是什么动画
```

### 选中

```
用选框工具选中
移动 或者 缩放 旋转 选框

进行选择时，仅仅针对当前cel进行


加 减 合并
上下文中，可以更改 对选定区域 的处理方式
默认操作：替换  鼠标左键
并： Shift 鼠标左键
减法： Alt Shift 鼠标左键
相交： Ctrl Shift 鼠标左键

Ctrl + A 全选
Ctrl + D取消选中
Ctrl + Shift + D 重新选择
Ctrl + Shift + I 倒置
```

### 变形

```
翻转选择
Shift + H
Shift + V

菜单更改画布大小
Sprite>Canvas Size

选中更改画布大小
选中 Sprite>Crop

修剪，自动
Sprite>Trim

移动选择
选中拖拽

移动
鼠标拖拽 V 或 Ctrl 选择移动工具
对cel的xy位置很有用
Shift 锁定 x 或 y 轴

选择并移动图层
Ctrl 和 左键 快速移动图层

移动多个cels
选中多个cels，移动图层

调整大小
Edit>Sprite Size

旋转
选中，拖动角
按住Shift 0 45 90度
```

### 导出

```
File>Export
可以导出gif或者png

序列
0 1 2 结尾的
可以打开为动画
可以导出为序列png
```

---

## [<主页](/index.html)