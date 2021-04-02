### [<主页](/index.html)

# UE4 游戏编程入门

#### 熟悉编辑器
```
Ctrl+Alt+拖拽 多选

Alt+左键+拖拽 绕轴旋转相机
Alt+右键+拖拽 绕轴缩放相机
Alt+中键+拖拽 上下左右移动相机

Ctrl+双键 上下移动物体
Ctrl+右键 横向移动物体
Ctrl+左键 纵向移动物体
```
### 理解Gameplay框架
```
项目结构
Config 存储配置文件 .ini
Content 所有资源
Intermediate 项目ini 偏好 缓存bin

文件类型
uasset 导入外部资源
umap 新建地图

导入 记得 save
*代表没有保存

合并 选中资源-资源操作-合并
选择一个Content文件夹

建议：有些资源是内部创建的，蓝图 粒子系统 摄像机数据
有些资源是外部的，在项目的根目录添加一个文件夹来追踪原始文件
原始文件放进git或者svn
目录结构为：RawAssets -Models -Texutes -Audios

Saved包含
AutoSaves 临时工作文件
Backup 同上
Config 存储项目设置ini
Logs
```
#### Gameplay框架
```
是一个C++或蓝图类集，管理每个项目中的游戏规则，用户输入和Avatar
摄像机，玩家的HUD

GameMode类
一个GameMode及其依赖被创建后,可以将GameMode分配给这个项目
或项目中的每个独立关卡，许多项目包含两三个GameMode
默认Mode在项目中设置，在编辑器的世界设置中,关卡可以Override Mode
分配到GameMode的一些类：
DefaultPawn
HUD
PlayerController
Spectator
ReplaySpectator
PlayerState
GameState

Controller类
一个Controller控制一个Pawn
PlayerController类利用玩家的输入来命令玩家的Pawn
两种基本类型：PlayerController AIController
PlayerController可以管理玩家的输入
通过占有游戏中的Pawn来控制这个Pawn
PlayerController也是 开启光标，显示和设置游戏如何响应
鼠标点击事件的地方
多人游戏有多个PlayerController类
例如：每当一个玩家加入，一个PlayerController类的实例会被
创建在GameMode中，成为游戏会话的其他部分，被分配给这个玩家
PlayerController在游戏中没有可见的物体表示

Pawn类和Character类
Pawn类接收来自PlayerController类的输入，然后使用这些输入
来命令玩家在游戏世界中的物理代表
它有时像是在关卡中表现玩家的位置一样基础，有时又像使用一个
带有碰撞壳的动画骨架网格物体在游戏世界中移动一样复杂。
有几个类可以分配在GameMode的DefaultPawn属性上，
例如Pawn 通用类 Character Vehicle 常见类
Pawn采用来自Controller的指令

HUD类
HUD类用来绘制2D Head Up Display
一个游戏的整个HUD系统可以编写在HUD类中
Epic提供了Unreal Motion Graphics UMG编辑器
用来制作复杂界面和HUD的工具集和类集

Epic为常见游戏类型提供了一些已经有GameMode
Project setting可以修改 DefaultGameMode
场景 setting 可以 Override mode
```
### 坐标系 变换 单位和组织
```
控制个别Actor的类型变换
使用右手笛卡尔坐标系

平移 缩放 旋转
W  E  R 快捷键
Space 循环切换

旋转
Pitch X
Yaw  Y
Roll  Z

默认情况下，
单位：一虚幻单位 uu == 1 厘米

建造Actor前需要确保尺寸
移动旋转缩放 都有对齐到格子5倍单位 固定角度 倍率
```
#### World Outliner
```
搜索
搜索减法 ground-aria1 不包含aria1的ground名称

Group 组
Ctrl+G
对于组，变换应用给组的中心
一个Actor一次仅可以从属于一个组

图层
Window-Layers
选择Actor加入图层

组和图层的区别
同一个Actor可以放入多个图层，但只能同时存在于一个组

附加 树型结构

旧的虚幻度量系统：
编辑 编辑器偏好设置 Grid Snapping
Use Power of Two Snap Size

对齐Actor到网格
右键被移出网格的Actor，在右键菜单选择
变换 对齐/排列 对齐原点到网格

即使网格关闭，Actor仍然会沿着网格移动
```

---


## [<主页](/index.html)
