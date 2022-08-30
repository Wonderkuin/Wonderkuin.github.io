# Practical Game AI Programming
# 游戏AI开发实用指南

---

# 可能性图 概率图

```
游戏状态
就是游戏中的预定动作
角色要么从一个状态切换到另一个状态，要么由于正处在另一个状态中而无法进行切换
```

```
可能性图

如果AI先看到玩家
	走 掩护 射击
如果玩家先看到AI
	跑 掩护 射击
如果AI血量低
	防守 跑 掩护
如果玩家血量低
	射击 跑 射击

被动状态
	保持位置
防守状态
	寻找掩护 进入建筑
进攻状态
	看到玩家 射击 搜寻玩家
AI 可以从被动状态 切换到 进攻状态 或 防守状态
进攻状态 防守状态 可以相互切换

玩家从右侧接近 AI可以掩护
玩家从左侧接近 AI没法掩护
玩家从中间接近 AI退回建筑

按照区域触发

这种原理可以广泛应用于各种游戏类型
```

```
概率图

概率图是一种更复杂 更细化版本的可能性图
基于概率改变角色行为，而不是基于简单的 开/关触发器

和可能性图相似的地方是，它也被用来提前规划角色可能的行为
而这次添加了一个百分比，依靠它AI会计算哪一种行为会被采用。

敌人AI白天会比晚上更具有进攻性。

时间  守卫  吃喝 走动
早上  0.87  0.1  0.03
下午  0.48  0.32  0.2
晚上  0.35  0.4  0.25

上面是一张概率图，定义了角色切换到每一种状态的概率。
这对创建角色的不可预测性很有帮助，让角色行为不那么明显。
好比每次玩这关，它都站在同样的位置上。
我们可以每5分钟左右更新一次概率，以针对玩家一直站在那里等待敌人变换位置的情况。

这种技术最多用于潜入式游戏，这种游戏里观察是关键。
因为玩家有机会待在一个安全位置观察敌人行动。

概率图可以用来让AI自己学习。
让玩家每次玩游戏之后，AI都会变得更聪明，玩家和AI敌人都会学习，
使得挑战随着游戏的进行时间而升级。

如果玩家习惯于使用某种武器从相同的方向过来，计算机都可以更新这些信息并且在将来使用。
如果玩家面对计算机敌人100次，并且60%的情况下使用了手榴弹，AI应当记住这条信息，
然后根据这个概率制定对策。
这会促使玩家思考其他的战术，探索其他打败敌人的方式。
```

---

# 产生式系统

```
如何结合其他技术和策略来使用可能性图和概率图

我们决定使用某种技术 或者某种技术是否比其他技术好，
这完全取决于我们将要创建的是怎样的游戏。
在创建AI角色时也一样，我们应该知道AI在游戏的每一秒钟，该做什么
什么时候做。
```

```
自动有限状态机

在规划AFSM阶段，将AI角色要执行的动作拆分为两到三列信息是一个很好的开始。
第一列，我们放上目标的主要信息，如方位 速度 目的地
其他列，我们放上用来达成目标所需执行的动作，如射击 移动 装弹 寻找掩体 隐匿 使用物品 等
这么做我们能确保我们的角色能根据第一列的内容做出反应，并且整个过程独立于它当前的位置。 

为了开发一个可以在任何地图上以相同方式进行反应的AI。
我们需要给AI角色分配一个主要的目的，以及让它知道实现目的的所有可能性
同时还需要确保它在游戏进行时每秒都有事情做。

主要目的
	击败玩家 生存下来
	如果AI血量大于20%，主要目的是击败玩家
	否则是存活下来
次要目的
	寻找玩家 寻找掩体 寻找道具点
	通过使用它们，AI将有能力实现主要目的，且总会有事情需要做
	不会去等待玩家的行为
所有可能动作
	移动 射击 使用物品 蹲下

这个方法与前面章节采用的方法完全不同，因为当时我们使用图来指示
AI角色应该去做什么事，只根据位置来下达指令。
而这个例子中，我们想要角色无论在哪个位置，哪张地图，都可以做出最佳决策。
这种方法会让我们的AI开发水平迈上一个全新的台阶
因为人类的真实行为很少会像之前那样，根据一个 统一 固定的标准做出决策


准备好这三列内容后，将第三列的每个动作，连接至第二列的每个行为，
然后将第二列的每个行为，连接到第一列的主要目的。
需要考虑：在AI想要寻找玩家，掩体或者道具时，需要做哪些具体的事情
同时，还需要定义它应该在什么时候寻找玩家，掩体或者道具点。


寻找玩家
	移动 射击
寻找掩体
	移动 蹲下
寻找道具点
	移动 使用物品

击败玩家
	寻找玩家 寻找掩体
生存下来
	寻找掩体 寻找道具点


添加细节
如果敌人AI面对玩家，有足够的HP，但是没有足够的弹药？
如果有足够的弹药，但是上一次攻击失败了怎么办？

从命中率开始：假设AI已经发出10颗子弹，命中4颗。
可以认为 它下一次射击中 有40%的可能性击中玩家
假设枪里只有两发子弹，此时它会怎么做？
是在命中率不高且弹药不多的情况下朝敌对玩家开火，之后没有子弹？
还是跑向道具点补充弹药？
为了做出这个决定，AI角色需要计算被玩家射到的可能性：
如果玩家射到AI角色的可能性较小，AI将会冒险尝试攻击玩家，
否则去补充弹药。

如果想要精确的百分比，可以考虑最近两分钟的命中率。
```

```
基于效用的函数

模拟人生

角色
	索菲
家里物品
	沙发 浴室 电视 床 烤箱 等
需求
	饥饿 活力 舒适 卫生 愉悦

第一列
	Hunger Energy Comfort Hygiene Fun
第二列 动作
	去干什么。。。

这个和 有限状态机中说明的FPS例子一样
下面说点不一样的

首先考虑饥饿感
吃了早晨 饥饿感100%
经过一段时间 98% 还不算饥饿状态
不能立刻补充那2%

游戏内能做的事情需要平衡，不能睡五分钟，工作5分钟，
往往设定为 睡足够长的时间来补偿清醒时的时间，
吃合理数量的食物，维持一段时间的饱腹状态。

我们需要判断出 “有一点饿，但首先需要完成工作” 的情形
除此之外还需要对一系列的决策进行比较，如：虽然饿，但是更累

start:
hunger = 100
energy = 100
comfort = 100
hygiene = 100
fun = 100

update:
overall = (hunger + energy + comfort + hygiene + fun)/5
hunger -= deltaTime / 9
energy -= deltaTime / 20
comfort -= deltaTime / 15
hygiene -= deltaTime / 11
fun -= deltaTime / 12

overall用来计算角色的整体情况，并完美呈现虚拟角色的心情
现在我们分别处理每个需求，为每一个需求都创建一个决策树。

是不是饿了
	有没有事物？
	检测冰箱
	做饭
		吃

是不是困了
	是不是有工作
		是不是需要上厕所
			睡觉

是否不舒适
	是不是正在做事
		是不是可以坐下
			坐下

是否需要洗澡
	是不是正在做事
		洗澡

是否无聊
	是不是忙
	能不能同时看电视
		看电视

缺陷：如果饿或者累，不能选择继续看电视。
为了改善这一点，可以添加一个概率图，用Overall变量来
定义她是否开心。如果开心，即使累了，也可以继续看电视。
```

```
游戏AI的动态平衡

通过调节属性改变难度 速度 生命值 魔法值 战斗力
另一个方法是 调节敌人数量
不要制造橡皮筋那样的敌人，比如，AI车在玩家后面，会更快，玩家车在对手前面，会降低速度。
	这样会变得乏味。
在格斗游戏中，AI这样战斗：玩家靠近，AI会拳打脚踢，否则向玩家靠近。
	然后用使用AI攻击的百分比和时间间隔了调整难度。
在FPS游戏中，游戏AI是由于在开发阶段考虑到玩家表现而调整的，程序员会调整AI状态值
	和战术 直到其与玩家的整体能力相匹配。
	如果玩家的命中率是70%左右，AI角色将使用这个值以保持相对接近人类的表现。

古惑狼 Crash Bandicoo
	使用的动态游戏难度平衡不会直接改变AI角色行为
	而是表现在动画速度上，如果玩家有困难，就放慢动画速度。
	这种难度调整的方式是根据玩家死亡次数进行的。

生化危机4 Resident Evil 4
	难度调整的原理相同，不过采取了更复杂的系统
	系统会根据玩家表现在后台对玩家评分，范围在1到10之间
	1意味着玩家在游戏中玩的不顺畅，10代表玩家非常熟练
	根据这些评分，敌人的行为会不同，攻击性和造成的伤害值都有所增减
	评分会持续更新

求生之路 Left 4 Dead
	也考虑了玩家评分，但不只是增加了敌人AI难度，还改变敌人出生地，
	让玩家在游玩同一个关卡，有不同的挑战。
	如果玩家刚上手游戏，敌人出现在比较容易被消灭的地方，
	如果玩家之前玩过这关，敌人出现在相对难消灭的位置

总结
	难度并不总要根据玩家的行为上调下降
	一个很好的例子是模拟游戏，其中关键是需要使得难度变化贴近真实生活
	而不是使玩家的游戏过程变得更困难或者更容易，否则它不再是模拟游戏
	其他例子：魔界村 Ghosts'n Goblins 黑暗之魂 Dark Souls
	开发者明显在不改变AI行为的情况下，让游戏从头到尾都非常难
```

---

# 环境与人工智能

```
当我们创建AI时，最重要的方面之一是它所处的位置
AI角色所处的位置会彻底影响它的行为以及将来的决策。

作为玩家，我们更喜欢探索生动的世界，里面有很多事情可做
对游戏开发者来说，那意味着更多的工作量
同样的问题对于AI角色本身来说也是如此。
如果我们允许角色与场景交互，那需要增加很多工作
玩家或者AI可以选择的事情数量往往等同于可能带来的问题的数量
所以我们创造游戏时需要特别关注环境

并不是所有游戏都必须要有地图或者地形，但是动作发生的位置
始终在玩法中占据重要地位，AI本身也要清楚这一点
当然有时候玩家没有在意环境和位置对角色产生的微妙影响
但是大多数情况下，这些微妙的变化会为游戏带来良好而愉悦的体验
与环境交互是创作电子游戏时很重要的方面
只有它才能赋予角色生命，没有它角色只是单纯的模型

另外，不能忘记角色对环境的影响
我们可以选择与环境的交互是玩法的一部分
或者仅仅为了改善视觉体验
无论哪种设计的终极目的都是创建一个人人喜爱的丰富环境
```

```
视觉交互

不会对游戏玩法产生直接的影响，但是对打磨游戏与角色的品质有帮助，
所以它也是环境的一部分，对提升用户玩游戏的沉浸感有显著贡献。

背景里的可破坏物体
元祖版恶魔城 Castlevania 1986 NES
玩家可以用鞭子破坏蜡烛和火炬，它们本来是用作背景的一部分

另外的例子
游戏中的物体在角色经过它们时会播放动画或者移动。
这和可破坏物体的原理一样，但这次是角色移动到物体位置附近时的一次微妙的交互。
从草地尘土到水洼，会飞走的鸟，做鬼脸的敌人。
这些不需要AI
```

```
基本环境交互

利用环境来达成游戏目标开始成为游戏体验的一部分。

古墓丽影
主角需要推动方块到标记位置的顶部，这会改变环境并打开一条新的路径。
从而继续推进关卡。


移动环境中的物体

玩家在正确的位置上吗？
	等待
我能看见方块吗？
	寻找方块
先推哪一个方向？
	X AXIS Y AXIS
方块的X坐标等于标记的X坐标吗？ 方块的Y坐标等于标记的Y坐标吗？
目标达成


环境中的障碍物

可以在游戏中使用或移动物体来达成目标
但是有物体挡住路线时会怎样？
障碍物可能是被玩家放在那的或者是本来就在那里，无论哪种
AI角色都应能知道如何应对

简化这个问题，分析当某个物体干扰了AI角色的行为时，AI应当如何应对
例如：AI角色需要进入房屋，但是当它到达附近后发现房屋被栅栏包围无法进入
这时，我们希望角色选择一块栅栏攻击，直到这部分栅栏被破坏，这样AI角色就可以进入了

对于这个例子，需要在考虑栅栏的距离与栅栏当前血量的前提下，计算角色应该攻击哪块栅栏
血量较低的栅栏比满血的具有较高的优先级

每个栅栏都有脚本，便于拓展栅栏，所有栅栏中最低的血量优先级最高，角色先攻击它
```

```
用区域阻断环境

创建地图时，有多个不同的区域用来改变玩法
区域可以包含 水 流沙 飞行区 山洞等
如果我们希望一个AI角色可以在任何地方使用，
我们需要考虑到这点，并且让AI感知地图上的所有不同区域

这意味着我们需要输入更多信息到角色行为中，
包括 怎样根据当前位置做出反应
某些情况下能够旋转自己去哪儿

应该避开某些区域吗，应该偏好其他区域吗
这些类型的信息与我们设计有关，这让角色能够感知周围的环境
选择或者适应的同时考虑他自己的位置
如果不正确设计这些，会导致不自然的决策

例如：Bethesda Softworks开发的上古卷轴5天际
我们能看到一些AI在遇到不知道该如何应对的区域时，只是简单转身往回走
特别是遇到山或者河流

视角色遇到的区域不同，它可能会有不同的反应，或者更新它自己的行为树以适应环境
角色周围的环境会重新定义行为的优先级，或者彻底改变角色的行为
```

```
高级环境交互

百战天虫 地图可以完全被摧毁，游戏AI可是适应这种地形并持续做出聪明的决策
到了物理元素时，环境变化的结果可以是完全随机的


适应不稳定的地形

分解这个例子
在底部，水面会自动杀死虫子
然后，有地形可供虫子走动，也可以被破坏
最后，地形的空隙，空白区域不能够走上去。
游戏开始，虫子们被随机放置在地图各个位置上，它们可以走动，跳跃，射击

																		N					呆在墙边或者攻击最近敌人
								N						附近有补血点吗  	    		  |
																						N
																		Y 我能拿到它吗？
																						Y 取得补血
														^
				N 附近有敌人吗							 |
														N
								Y 目标敌人血量较低吗？
														Y 向它射击
我的血量高于80%吗
								N goto附近有补血点吗
				Y 附近有敌人吗
													N 向它射击
								Y 敌人正被地形阻挡吗
													Y 射击地形

游戏中的角色可以持续适应会变化的地形，所以我们需要考虑到这一点并将其作为行为树的一部分
角色需要知道它现在的位置 对手的位置 血量 道具
由于地形会阻挡住角色，AI角色有几率被困在一个位置 无法攻击 也无法取得道具 这时需要给它一些选项
例如，虫子没有空间可以移动，没有拿得到的道具，也没有能打到的敌人，应该怎么做？ 
首先收集周围信息，做出当前情形的判断。
这里，我们定义角色对最近的敌人射击，然后靠近墙面，由于它可能会离攻击最近敌人时产生的爆炸过于接近。因此它应该
选择等待在一个安全的角落，直到下一个回合到来。


使用射线检测评估决策
理论上，角色有两条射线，一左一右，检测两边是否有墙阻挡。
帮忙确定角色应该朝那边移动，才不会被攻击到
当角色想要攻击的时候，我们会再用一条射线指向瞄准的方向，看
是否有什么东西阻挡攻击的通路，如果有物体，可以计算距离来判断能否安全射击
因此，每个角色能访问所有虫子所在位置的列表，这样就能比较和每一个虫子的距离
选择最近的射击。另外，添加了两个射线来检测两边是否有东西阻挡，而且还有基本的信息来让
角色适应持续变化的地形。
```

---


# 动画行为

```
动画状态机

静止  静止到移动 移动
移动到静止左脚 移动到静止右脚
跳跃 静止到跳跃 移动到跳跃 落地到移动
爬梯子 静止到爬梯子 移动到爬梯子
爬梯子到静止 爬梯子到移动

动画状态比行为状态要多，所有要把Gamplay逻辑和动画分离开来
开发游戏时，使用的是表达式和值，所以走路和跑步的唯一区别是角色速度变量的值
而使用动画状态的原因，是要把走路和跑步这个信息转化为视觉效果，动画是根据行为状态
来工作的，使用这种方法并不意味着动画不能干预游戏逻辑，因为根据需要，仍然可以
把相关动画信息反馈给游戏逻辑代码，来改变游戏逻辑

导入所有动画到动画状态机窗口，接下来根据代码来配置每个动画何时播放。
这里的表达式可以用整数，浮点数，布尔值，触发器。
有了这些，可以定义每个动画播放的时机
给动画连线的时候，就使用了表达式的值来决定什么时候切换动画的状态。
：如果角色的移动速度达到某个值，就会开始播放跑步动画，
而当减小到这个值以下时，就会改为播放走路动画

状态和Gameplay无关，可以随意修改，删除，新增动画
IDLE
	idle00 idle01
ATTACK
	punch20 kick20
JUMP
	jump10
LOCOMOTION 动画过渡
	STRAIGHT 站着走
		walk00 run00 runned00
	CROUCH 蹲着走
		crouch20 crounddown10 crouchwalk10 crouchup10
		crouchidle10 croudture10
DAMAGE
	damage20 down20 group20

利用变量控制动画
Animator characterAnimator;
int Health;
int Stamina;
float movementSpeed;
float rotationSpeed;
float maxSpeed;
float jumpHeight;
float jumpSpeed;

float currentSpeed;
bool Dead;


平滑过渡

2D和3D动画考虑的方式不同。
如果使用2D精灵，就需要为每一种动作转换画出必要的帧，每次角色切换动画时，切换的相应
动画就会被播放。
对于3D角色，我们可以用骨骼动画自动创建过渡，某些时候也必须手动创建一些过渡动画，
而且有时为了效果更好也需要这么做。比如：角色在使用武器或物品时，在进行下一个动作前要维持当前的动画状态。

要创建平滑的动画，我们需要让后一个动画的第一帧和前一个动画的最后一帧相同。需要从相同的位置开始播放下一个动画。
这在避免出现跳跃性的动画过渡方面是关键性的方法。
我们可以用游戏引擎中的动画转换系统来帮助我们创建平滑的动画过渡，我们可以通过调整过渡时间，可以设置为快速过渡或者是
慢一些，通常需要根据视觉效果来进行实验。

有时我们可能需要牺牲平滑的动画切换来换取更好的游戏体验。例如：在格斗游戏中，快读切换就比流畅切换要重要的多，因此
需要根据实际情况考虑动画切换所要花费的时间。
```

---

# 导航行为和寻路

```
导航行为 实际指的是一个角色面对特定情况时的行为
这种情况下，需要计算去哪儿或者去做什么
一张地图可以有许多特殊的地点，这些地点可能需要楼梯或者跳跃才能最终到达目的地。
角色应该知道如何运用这些动作来保持正确移动，否则它可能会因为没有跳过地洞而跌到洞里，
或者走到墙边时因为没有爬楼梯而一直卡在墙下。
为了避免这种情况，我们需要计划好所有可能性，确保能通过跳跃或者其他动作来保持正确移动。
```

```
选择新的方向
当AI遇到障碍时，可以选择新方向以绕过障碍
这对于AI来说是非常重要的一个方面
角色应该意识到前方有物体挡住了去路，当它不能继续朝那个方向前进时，它需要选择一个新的方向以避免撞到障碍物

避免撞墙
如果我们的角色面对一堵墙，它需要知道自己不能穿过那堵墙，并且可以选择其他可能的路线。
除非允许角色爬墙或者摧毁墙，否则角色应当面朝一个没有被阻挡的新方向移动。

选择其他的路线
我们的角色在每次接近墙壁时都选择了一个新的方向，现在我们希望它能够在地图上更自由地移动
为了做到这一点我们将添加一些逻辑，让角色有一个左右转向的机会时，可以自由旋转，即使前方的道路是通畅的
我们可以用概率来决定角色是否会改变方向，在这个例子中我们指定了90%的概率会选择一个新的方向。


点到点的移动

塔防类型
敌人在一个起始点生出来，并沿着一条路径到达终点。

赛车类型
AI驾驶员使用点到点移动的方式来对抗玩家。
实现一个点到另一个点的平滑过渡很重要。
```

```c#
// 赛车类型 代码
public static bool raceStarted = false;//AI是否开始

public float aiSpeed = 10.0f;//车辆的速度
public float aiTurnSpeed = 2.0f;//车辆转向速度
public float resetAISpeed = 0.0f;
public float resetAITurnSpeed = 0.0f;

public GameObject waypointController;//关联场景中的waypoints组
public List<Transform> waypoints;
public int currentWaypoint = 0;//当前寻路点
public float currentSpeed;//车辆当前速度
public Vector3 currentWaypointPosition;//当前车辆正在跟随路点的位置

void Start()
{
	GetWaypoints();//获取寻路点组里所有存在的寻路点
	//初始速度 影响车辆上挂载的刚体组件
	resetAISpeed = aiSpeed;
	resetAITurnSpeed = aiTurnSpeed;
}

void Update()
{
	if (raceStarted)
	{
		MoveTowardWaypoints();
	}
}

void GetWaypoints()
{
	Transform[] potentialWaypoints = waypointController.GetComponentsInChildren<Transform>();

	waypoints = new List<Transform>();

	foreach (Transform potentialWaypoint in potentialWaypoints)
	{
		if (potentialWaypoint != waypointController.transform)
		{
			waypoints.Add(potentialWaypoint);
		}
	}
}

void MoveTowardWaypoints()
{
	// 获取寻路点位置
	float currentWaypointX = waypoints[currentWaypoint].position.x;
	float currentWaypointY = transform.position.y;
	float currentWaypointZ = waypoints[currentWaypoint].position.z;

	// 用来计算下一个寻路点离当前位置的距离，而且要将这个方向从世界空间转换到本地空间
	// InverseTransformPoint 将世界坐标转换成局部坐标
	Vector3 relativeWaypointPosition =
		transform.InverseTransformPoint (new Vector3(currentWaypointX,currentWaypointY, currentWaypointZ));
	currentWaypointPosition = new Vector3(currentWaypointX,currentWaypointY,currentWaypointZ);

	// 插值平滑旋转
	Quaternion toRotation = Quaternion.LookRotation
		(currentWaypointPosition - transform.position);
	transform.rotation = Quaternion.RotateTowards
		(transform.rotation, toRotation, aiTurnSpeed);

	GetComponent<Rigidbody>().AddRelativeForce(0, 0, aiSpeed);

	if (relativeWaypointPosition.sqrMagnitude < 15.0f)
	{
		currentWaypoint++;

		if (currentWaypoint >= waypoints.Count)
		{
			currentWaypoint = 0;
		}
	}

	currentSpeed = Mathf.Abs(transform.InverseTransformDirection
		(GetComponent<Rigidbody>().velocity).z);

	float maxAngularDrag = 2.5f;
	float currentAngularDrag = 1.0f;
	float aDragLerpTime = currentSpeed * 0.1f;
	float maxDrag = 1.0f;
	float currentDrag = 3.5f;
	float dragLerpTime = currentSpeed * 0.1f;

	float myAngularDrag = Mathf.Lerp(currentAngularDrag,
		maxAngularDrag, aDragLerpTime);
	float myDrag = Mathf.Lerp(currentDrag, maxDrag, dragLerpTime);

	GetComponent<Rigidbody>().angularDrag = myAngularDrag;
	GetComponent<Rigidbody>().drag = myDrag;
}
```

```
MOBA类型 多人在线战术竞技游戏
小兵 多组寻路点
```

```c#
// 很多寻路点 找到离目的地最近的寻路点
public float speed;
private List<GameObject> wayPointsList;
private Transform target;
private GameObject[] wayPoints;

void Start()
{
	target = GameObject.FindGameObjectWithTag("target").transform;
	wayPointsList = new List<GameObject>();

	wayPoints = GameObject.FindGameObjectsWithTag("wayPoint");

	foreach (gameObject newWayPoint in wayPoints)
	{
		wayPointsList.Add(newWayPoint);
	}
}

void Update()
{
	Follow();
}

void Follow()
{
	GameObject wayPoint = null;

	if (Physics.Linecast(transform.position, target.position))
	{
		wayPoint = findBestPath();
	}
	else
	{
		wayPoint = GameObject.FindGameObjectWithTag("target");
	}

	Vector3 Dir = (wayPoint.transform.position - transform.position).normalized;
	transform.position += Dir * Time.deltaTime * speed;
	transform.rotation = Quaternion.LookRotation(Dir);
}

GameObject findBestPath()
{
	GameObject bestPath = null;
	float distanceToBestPath = Mathf.Infinity;

	foreach (GameObject go in wayPointsList)
	{
		float distToWayPoint = Vector3.
			Distance(transform.position, go.transform.position);
		float distWayPointToTarget = Vector3.
			Distance(go.transform.position, target.transform.position);
		float distToTarget = Vector3.
			Distance(transform.position, target.position);
		bool wallBetween = Physics.Linecast(transform.position, go.transform.position);

		if ((distToWayPoint < distanceToBestPath) && (distToTarget > distWayPointToTarget) && (!wallBetween))
		{
			distanceToBestPath = distToWayPoint;
			bestPath = go;
		}
		else
		{
			bool wayPointToTargetCollision = Physics.Linecast(go.transform.position, target.position);
			if (!wayPointToTargetCollision)
			{
				bestPath = go;
			}
		}
	}
	return bestPath;
}
```

```
点到点移动 和 躲避动态障碍

这里使用一种结合寻路点移动与迷宫移动的方法，而同一时刻AI只能选择两种方法的一个
AI可以根据它面临的现状选择最合适的方法

如果道路畅通
	向寻路点移动
如果有阻挡
	中间有阻挡
		向右
	左边有阻挡
		向右
	右边有阻挡
		向左

射线检测会一直判定，直到不再被阻挡


MOBA中的小兵，有一个触发范围，如果什么东西触发了区域，角色停止朝寻路点移动
```

---

# 高级寻路

```
A*搜索算法
开销一致性 启发式特性 不需要设置寻路点
游戏的地图和场景需要提前分析和准备
环境与其包含的所有资源会被当做一张图来处理 这意味着地图将会分为不同的点和位置
我们称为节点，这些节点用来记录整个搜索的过程。在记录地图位置时，每个节点都有其自身的
属性，包括 fitness goal  heuristic启发式
通常写作 f g h 综合考虑三个属性可以看出当前节点的优劣

A*算法使用两个列表 开启列表 和 关闭列表
开启列表里包含需要检查的所有节点 还可以用标记数组来检查某个节点是否在开启列表或关闭列表中
这意味着AI角色将会不断搜索最佳节点，以得到最快或者最近的结果

缺点：
CPU负担大
让AI自行工作不受人工干预时，容易产生bug


算法描述：
G 表示从起点A移动到当前方格的距离有多远
H 表示从当前方格移动到终点B的预计距离有多远
F=G+H F用来得出最佳路径
有多个最佳选项 即多个F相等时 需要选择其一
由于多个F相同，因此只需要根据节点的H值来做出决定 选择H最小的节点
继续寻路，这时F值大于之前的F值，这意味着需要返回之前的节点才能找到最佳路线
重新选择之前多个相同的F，继续寻路，发现F更大了，因此需要回去计算另一个值，
看看后面是否有一个较小的值
```

```
// 已经搜索过的节点
OPEN // the set of nodes to be evaluated
// 尚未探索的节点
CLOSED // the set of nodes already evaluated
// 将起始节点分配给OPEN
Add the start node to OPEN

loop
	// 当前OPEN里F最低的节点
	current = node in OPEN with the lowest f_cost
	// 从OPEN里移除
	remove current from OPEN
	// 添加到CLOSED
	add current to CLOSED

	// 如果当前节点是目标节点，已经找到，退出循环
	if current is the target node // path has been found
		return

	// 检查当前节点的每个相邻节点
	foreach neighbor of the current node
		//如果其中某个节点不能通过，或者在CLOSED列表中
		if neighbor is not traversable or neighbor is in CLOSED
			// 跳到下一个相邻节点
			skip to the next neighbor
		// 这部分设置了所有可能移动的位置，同时也告诉了AI不要考虑先前已经探索过的位置

		// 如果没有跳过这个节点
		// 如果到相邻方块的新路径比旧路径短，或者相邻方块并不在OPEN列表中
		if new path to neighbor is shorter OR neighbor is not in OPEN
			// 通过g_cost h_cost来计算f_cost
			set f_cost of neighbor
			// 设置相邻节点的父节点为当前节点
			// 我们可以看到新的可能节点有来自当前节点的子节点 因此我们可以跟踪正在执行的步骤
			set parent of neighbor to current
			// 最后如果相邻节点不在OPEN列表中，我们将其添加进去
			if neighbor is not in OPEN
				add neighbor to OPEN

	// 循环这个过程
```

---

# 群体交互

```
小组战斗

通过检测AI角色和玩家之间的距离来决定首先由哪个角色发起攻击
我们希望近处的角色优先攻击，其他人在等这个角色血量较低时再攻击
一旦角色血量偏低，第二近的角色就会加入战斗并攻击玩家

现在我们已经设定了第一个判断标准以决定谁先攻击，接下来要确定当其他角色
等待时应该做些什么，要考虑到，玩家可以在任何时间攻击任何一个敌人，而且
我们不希望AI角色仅仅由于没有等到攻击的时机而一直处于静止状态。
因此解决方法是思考可能发生的不同状况并计划AI应当计划AI应当如何反应，
特别是角色之间应当如何交互
```

```c#
public static int attackOrder;
public bool nearPlayer;
public float distancePlayer;
public static int charactersAttacking;
private bool Attack;
private bool Defend;
private bool runAway;
private bool superiseAttack;

void Update()
{
	if (distancePlayer < 30f)
		nearPlayer = true;
	if (distancePlayer > 30f)
		nearPlayer = false;

	if (nearPlayer && attackOrder == 1)
		Attack = true;
	else
		Defend = true;
}
```

```
通信 警告区域

AI角色进入玩家触发区域，会发出喊叫让附近AI角色知道玩家的存在
AI角色也有自己的触发区域，用来警告其他角色，代表喊叫范围
```

```c#
public bool nearEnemyAttacked;
public Update()
{
	if (nearEnemyAttacked)
		runPlayerDirection();
}
```

```
通信 与其他AI角色交谈

添加一个整形变量probabilityFriendly代表发现熟人的概率
当新的角色进入了触发区域，就会产生一个随机数，如果这个数字满足设定的条件
两个角色都会停止走动并开始交谈。

背后的思想是让角色随机地与其他人交互，从玩家的视角看来，这就好像这些角色本来就是朋友
他们停下来交谈只是因为他们认视对方
```

```
团队竞技

体育游戏往往具有优秀的AI系统，分析一下现实中的足球

球员向球跑动
离球最近的角色通知其他角色，其他角色不再向球的位置跑动
踢球，球的落点位置，选择一个队友并把球提给他
```

```
群体碰撞避免

多个角色想要同时到达一个目标点，就会造成互相碰撞
群体运动的解决方案通常涉及：寻路 局部碰撞避免

一个流行的方法是：结合A*算法和velocity obstacle方法

在高密度群体的场景中，仅仅依靠局部碰撞避免和理想化的寻路算法会引起角色
在热点，路径交汇点聚积。
碰撞避免算法只能帮助我们避免在进行寻路时的局部的碰撞。
游戏经常依赖于这些算法在高密度群体的场景里将角色转移到不太拥挤的，不太直接的路线
上去。在特定的情况下，碰撞避免可以达到这个目的。

将群体移动与群体密度纳入到寻路计算中的研究已经完成了。
根据群体密度优化寻路的方法并没有将群体的移动或移动方向考虑进去。
Congestion maps在很多方面与现有的合作寻路算法类似，例如Direction Maps DMs
但是在几个关键方面略有不同，DMs使用平均群体移动来鼓励角色跟随群体移动。
因此 很多Congestion maps方法中会局部振动的地方被平滑地解决了。
另一方面，这种随时间平滑的方法无法让DMs在环境突然变化时实现快速精准的响应
Congestion maps与DMs都使用了总体人群移动趋势的信息，以强化寻路规划过程，
这点是类似的 但是 Congestion maps会考虑到每个个体的大小与形状，
而DMs通常假设所有个体是一致的。
DMs和Congestion maps的最后一个重要区别在于，Congestion maps将移动惩罚的比重与
人群密度关联了起来。由于没有考虑密度，DMs显示出过于悲观的寻路行为，它会鼓励
个体移动时围绕阻挡了路径的群体，就算阻挡的群体比较稀疏。
```

---

# AI规划与避免碰撞

```
搜索行为
搜索可能是角色做出的第一个决策
因为大部分情况下，我们总希望角色搜索一些东西
可能是玩家，也可能是能让角色胜利的其他事物

让角色能够顺利地找到某样东西非常有用，也极其重要。
这个特性在大部分电子游戏中都可以找到，我们会想要用到它的。

敌人搜索玩家而不是守株待兔
为打猎游戏创建拟真的动物，例如让动物的主要目标是进食和喝水
饥饿时，一直搜索食物和水
一旦吃饱喝足，搜索温暖舒适的地方并待在那里
一旦发现猎人，搜索安全的位置


积极搜索
让搜索成为AI角色的主要目标，
游戏中的角色处于某种原因必须找到玩家，类似于捉迷藏


预测对手的行动
从一个简单的问题开始：玩家的脸朝向角色吗？
让角色检查这一点对于做出判断非常有帮助。
为了达到这个目标，我们添加一个触发器放在角色背部，再添加一个放在角色前面，包括玩家角色也这样做。
在角色前后放上触发器是为了帮助另一个角色识别看到的是前面还是后面。所以，我们为游戏中的每个角色
添加触发器，并将触发器命名为back和front
现在，让角色能区分出后置和前置触发器，这可以通过两种方式做到
第一种是在角色前面添加一个拉长的触发器代表观察范围
另一种方法，可以创建角色位置到视野终点的射线。


碰撞避免
预测碰撞对实现AI角色来说是一个很有用的方法，它也可以用在集群系统中来让群落更有组织地移动。
现在，让我们试试用一个简单的方式来实现这个功能
要预测碰撞至少需要两个物体
float MAX_SEE_AHEAD;
Vector3 velocity;
Vector3 ahead = transform.position + Vector3.Normalize(velocity) * MAX_SEE_AHEAD;
Vector3 ahead2 = transform.position + Vector3.Normalize(velocity) * MAX_SEE_AHEAD * 0.5f;
```

```c#
// 碰撞避免
public Vector3 velocity;
public Vector3 ahead;
public float MAX_SEE_AHEAD;
public float MAX_AVOID;
public Transform a;
public Transform b;
public Vector3 avoidance;

void Start() {
	ahead = transform.position + Vector3.Normalize(velocity) * MAX_SEE_AHEAD;
}

void Update() {
	float distA = Vector3.Distance(a.position, transform.position);
	float distB = Vector3.Distance(b.position, transform.position);

	if (distA > distB)
		avoidB();

	if (distB > distA)
		avoidA();
}

void avoidB()
{
	avoidance = ahead - b.position;
	avoidance = Vector3.Normalize(avoidance) * MAX_AVOID;
}

void avoidA()
{
	avoidance = ahead - a.position;
	avoidance = Vector3.Normalize(avoidance) * MAX_AVOID;
}
```

# 感知

```
开发使用战术与感知来达成目标的AI
这里会用到所有之前学到的东西
了解如何结合所有技术来创建人工智能角色
用于潜入或其他依赖战术与感知的游戏

感知：例如 声音 视野 知觉
```

```c#
//模拟角色视野

// 方向
public float viewRadius;
[Range(0, 360)]
public float viewAngle;

public Vector3 DirFromAngle(float angleInDegrees, bool angleIsGlobal)
{
	if (!angleIsGlobal)
		angleInDegrees += transform.eulerAngles.y;

	float rad = angleInDegrees * Mathf.Deg2Rad;
	// Normalized Forward, Unity坐标 x正向右，z正向前
	return new Vector3(Mathf.Sin(rad), 0, Mathf.Cos(rad));
}

// 物理检测视野范围
public LayerMask targetMask;
public LayerMask obstacleMask;
public List<Transform> visibleTargets = new List<Transform>();

void FindVisibleTargets()
{
	visibleTargets.Clear();
	Collider[] targetsInViewRadius = Physics.OverlapSphere(transform.position, viewRadius, targetmask);
	for (int i = 0; i < targetsInViewRadius.Length; i++)
	{
		Transform target = targetsInViewRadius[i].transform;
		Vector3 dirToTarget = (target.position - transform.position).normalized;
		if (Vector3.Angle(transform.forward, dirToTarget) < viewAngle / 2)
		{
			float dstToTarget = Vector3.Distance(transform.position, target.position);
			// 障碍物判断
			if (!Physics.Raycast(transform.position, dirToTarget, dstTarget, obstacleMask)
			{
				visibleTargets.Add(target);
			}
		}
	}
}

// 协程延迟反应时间
IEnumerator FindTargetsWithDelay(float delay)
{
	yield return new WaitForSeconds(delay);
	FindVisibleTargets();
}

// 拟真视野效果
// 射线碰撞检测 mesh显示
public float meshResolution;

void DrawFieldOfView()
{
	int stepCount = Mathf.RoundToInt(viewAngle * meshResolution);
	float stepAngleSize = viewAngle / stepCount;
	for (int i = 0; i <= stepCount; i++)
	{
		float angle = transform.eulerAngles.y - viewAngle / 2 + stepAngleSize * i;
		Debug.DrawLine(transform.position, transform.position + DirFormAngle(angle, true) * viewRadius, Color.red);
	}
}


// 结构体表示一个点
public struct ViewCastInfo
{
	public bool hit;
	public Vector3 point;
	public float dst;
	public float angle;

	public ViewCastInfo(bool _hit, Vector3 _point, float _dst, float _angle)
	{
		hit = _hit;
		point = _point;
		dst = _dst;
		angle = _angle;
	}
}


void DrawFieldOfView()
{
	int stepCount = Mathf.RoundToInt(viewAngle * meshResolution);
	float stepAngleSize = viewAngle / stepCount;
	List<Vector3> viewPoints = new List<Vector3>();
	ViewCastInfo oldViewCast = new ViewCastInfo();
	for (int i = 0; i <= stepCount; i++)
	{
		float angle = transform.eulerAngles.y - viewAngle / 2 + stepAngleSize * i;
		ViewCastInfo newViewCast = ViewCast(angle);
		Debug.DrawLine(transform.position, transform.position + DirFormAngle(angle, true) * viewRadius, Color.red);
		viewPoints.Add(newViewCast.point);
	}

	// 0 1 2 3 4
	// 012 023 034
	// 顶点总数是5
	// 三角形总数是3
	// t = v - 2
	// 数组长度为
	// (v-2) * 3
	int vertexCount = viewPoints.Count + 1;
	Vector3[] vertices = new Vector3[vertexCount];
	int[] triangles = new int[(vertexCount - 2) * 3];

	vertices[0] = Vector3.zero;
	for (int i = 0; i < vertexCount - 1; i++)
	{
		vertices[i+1] = viewPoints[i];

		if (i < vertexCount - 2)
		{
			triangles[i * 3] = 0;
			triangles[i * 3 + 1] = i + 1;
			triangles[i * 3 + 2] = i + 2;
		}
	}

	viewMesh.Clear();
	viewMesh.vertices = vertices;
	viewMesh.triangles = triangles;
	viewMesh.RecalculateNormals();
	// TODO 测试
	// 记得删除Debug DrawLine 否则网格不会出现在编辑器
}


// 显示mesh 创建游戏物体，添加MeshRenderer组件，拖动到 viewMeshFilter
public MeshFilter viewMeshFilter;
Mesh viewMesh;

void Start()
{
	viewMesh = new Mesh();
	viewMesh.name = "View Mesh";
	viewMeshFilter.mesh = viewMesh;
	StartCoroutine("FindTargetsWithDelay", .2f);
}


// 为了优化视觉效果，我们需要修改 viewPoints从世界坐标到本地坐标
void DrawFieldOfView()
{
	for (int i = 0; i < vertexCount - 1; i++
	{
		vertices[i + 1] = transform.InverseTransformPoint(viewPoints[i]) + Vector3.forward * maskCutawayDst;
	}
}

void LateUpdate()
{
	DrawFieldOfView();
}
```

---