# Unity3D人工智能编程精粹

---

# 第一章 Unity3D人工智能架构模型

```
游戏AI的架构模型

三种基本能力：
运动：移动角色的能力
决策：做出决策的能力
战略：战略战术思考的能力

通用的AI架构模型
小队AI
	战略层
角色AI
	决策层
	运动层

棋类游戏只包含战略层，因为这种游戏中的角色不需要自己做出决定，也不用考虑如何移动
其他许多非棋类游戏不包含战略层，如平台游戏中的角色，可能是纯反应型的


运动层
导航和寻路是运动层AI的主要任务，它们决定了角色的移动路径
具体的移动行为还需要动画层的配合

决策层
决策层的任务是决定角色下一步该做什么
决策层的功能可以用 有限状态机 行为树实现
也可以用更复杂的AI技术 模糊状态机 神经网络等实现

战略层
如果需要团队协作，还需要某些战略AI
战略指的是一组角色的整体行为，小组中的每个角色可以有它们自己的决策层和运动算法
但总体上，它们的决策层会受到团队战略的影响
战略层的实现技术和决策层相同

其他支持：
动画系统 物理仿真系统
```

---

# 第二章 实现AI角色的自主移动-操控行为

```
操控行为包括一组基本行为：
使角色靠近或者离开目标的 Seek Flee 行为
当角色接近目标时使它减速的 Arrival 行为
使捕猎者追逐猎物的 Pursuit 行为
使猎物逃离捕猎者的 Evade 行为
使角色在游戏世界中随机徘徊的 Wander 行为
使角色沿着某条预定路线移动的 PathFollowing 行为
使角色避开障碍物的 ObstacleAvoidance 行为

对于组成小队或群体的多个AI角色，包括基本的组行为：
与其他相邻角色保持一定距离的 Separation 行为
与其他相邻角色保持一致朝向的 Alignment 行为
靠近其他相邻角色的 Cohesion 行为

seek 靠近 flee 离开 arrival 抵达 pursuit 追逐 evade 逃避 wander 随机徘徊 path following 路径跟随
obstacle avoidance 避开障碍 group behavior 组行为 radar 雷达 separation 分离 alignment 队列 cohesion 聚集
```

```
运动层基类

Vehicle
直译为 交通工具
将AI角色抽象成一个质点
包含 position mass velocity max_force max_speed orientation
计算方法：
1 确定每一帧的当前操控力 (最大不超过max_force)
2 除以交通工具的质量mass 可以确定一个加速度
3 将这个加速度与原来的速度相加，得到新的速度 (最大不超过max_force)
4 根据速度和这一帧流逝的时间，计算出位置的变化
5 与原来的位置相加，得到 交通工具 的新位置

AILocootion
是Vehicle的派生类，它能真正控制AI角色的移动
包括每次移动的距离，播放动画

Steering
各种操控行为的基类
包括操控行为共有的变量和方法，操控AI角色的寻找，逃跑，追逐，
躲避，徘徊，分离，队列，聚集
```

```
个体AI角色的操控行为

靠近 SteeringForSeek
离开 SteeringForFlee
抵达 SteeringForArrive
追逐 SteeringForPursuit
逃避 SteeringForEvade
等
```

```
群体AI角色的操控行为

组行为：
分离 避免个体在局部过于拥挤的操控力
队列 朝向附近同伴的平均朝向的操控力
聚集 向附近同伴的平均位置移动的操控力


检测附近的AI角色
Radar 脚本略


与群中邻居保持适当距离-分离
分离行为的作用是使角色与周围的其他角色保持一定的距离，这样可以避免多个角色相互挤到一起
实现：为了计算分离行为所需的操控力，首先要搜索指定邻域内的其他邻居，然后对每个邻居
计算AI角色到该邻居的向量r，将向量r归一化r/|r|，得到斥力的方向，偶遇斥力的大小和距离成反比
还需除以|r|得到排斥力 r / (|r| * |r|)
float comfortDistance = 1;//可接受的距离
float multiplierInsideComfortDistance = 2;//当角色与邻居之间距离过近时的惩罚因子
Vector3 steeringForce = Vector3.zero;
////遍历所有邻居，计算steeringForce
//计算当前AI角色与邻居s之间的距离
Vector3 toNeighbor = transform.position - s.transform.position;
float length = toNeighbor.magnitude;
//计算这个邻居引起的操控力 可以认为是排斥力，大小与距离成反比
steeringForce += toNeighbor.normalized / length;
//如果二者之间的距离大于可接受距离，排斥力再乘以一个额外因子
if (length < comfortDistance)
	steeringForce *= multiplierInsideComfortDistance;


与群中邻居朝向一致-队列
通过迭代所有邻居，可以求出AI角色朝向向量的平均值以及速度向量的平均值，
得到想要的朝向，然后减去AI角色的当前朝向，就可以得到队列操控力
//当前AI角色的邻居的平均朝向
Vector3 averageDirection = Vector3.zero;
//邻居的数量
int neighborCount = 0;
////遍历所有邻居
//将s的朝向加到averageDirection中
averageDirection += s.transform.forward;
//邻居数量加1
neighborCount++;
////遍历完毕
//如果邻居数量大于0
if (neighborCount > 0)
{
	//将累加得到的朝向除以邻居的个数，求出平均朝向向量
	averageDirection /= (float)neighborCount;
	//平均朝向向量减去当前朝向向量，得到操控向量
	averageDrection -= transform.forward;
}
结果可能不太稳定


成群聚在一起-聚集
迭代所有邻居求出AI角色位置的平均值，然后利用靠近行为，将
这个平均值作为目标位置
//操控向量
Vector3 steeringForce = Vector3.zero;
//AI角色的所有邻居的质心，平均位置
Vector3 centerOfMass = Vector3.zero;
//AI角色的邻居的数量
int neighborCount = 0;
////遍历邻居
//累加s的位置
centerOfMass += s.transform.position;
//邻居数量加1
neighborCount++;
////遍历完成
//如果邻居数量大于1
if (neighborCount > 0)
{
	//将位置的累加值除以邻居数量，得到平均值
	centerOfMass /= (float)neighborCount;
	//预期速度为邻居位置平均值与当前位置之差
	desiredVelocity = (centerOfMass - transform.position).normalized * maxSpeed;
	//预期速度减去当前速度，求出操控向量
	steeringForce = desiredVelocity - m_vehicle.velocity;
}
```

```
个体与群体操控行为的组合

增加多个行为，通过决策层开启和关闭某种行为
如：与玩家距离小于60米，AI生命值大于80，追逐玩家
与玩家距离小于60米，AI生命值小于80，逃避
否则，徘徊
Vehicle.cs
foreach (Steering s in steerings
{
	if (s.enabled)
		steeringForce += s.Force() * s.weight;
}
steeringForce = Vector3.ClampMagnitude(steeringForce, maxForce);
要增加权重，带优先级
不过很困难，容易出现奇怪的行为，具体还要找文献
```

```
Boids实现

创建一个prefab 添加碰撞体和刚体，带运动学
AILocomotion 控制运动
Radar检测邻居
分离 Separation
队列 Cohesion
聚集 Alignment
闲逛 Wander


人群避开障碍实现
同理
AILocomotion
Radar
Separation
CollisionAvoidance


跟随领队行为
跟随领队行为属于一种组合行为，只用靠近或者追逐行为效果不好。
在靠近中，AI角色会被推向目标，最终占据与目标相同的位置，
追逐行为把AI推向另一个角色，目的在于抓住它而不是跟随它
在跟随领队的过程中，目标是接近领队，但稍微落后。当角色距离领队较远时，可能会
较快的移动，但是当距离领队较近时，会减速，这可以通过下列行为组合

抵达：向领队移动，在即将到达时减速
逃避：如果AI跟随着挡住了领队的路线，它需要迅速移开
离开：避免多个跟随者过于拥挤

要实现这种行为，首先要找到一个正确的跟随点。
AI跟随者要与领队保持一定距离，就像士兵跟随在队长后面一样，跟随点可以
利用目标，即领队的速度来确定，因为它也表示了领队的行进方向
tv = leader.veocity * -1
tv = normalize(tv) * Leader_Behind_Dist
behind = leader.position + tv
计算出领队的后方
Leader_Behind_Dist越大，跟随点离领队越远

接下来，只要以behind点为目标， 应用到达行为就可以了
然后，为了避免跟随着之间过于拥挤，需要加上分离行为
如果领队突然转向，跟随者可能挡住领头者，必须马上移动。
这时需要加入逃避行为
为了检测AI跟随着是否在头领的视线内，我们采用和碰撞避免行为中相似的检测方法，基于领队当前的速度和方向，
找当它前方的一个点ahead，如果领队的ahead点与AI跟随者的距离小于某个值，那么认为跟随者在领队的视线内
并且需要移开

领队
CharacterController
AILocomotion
SteeringForWander

跟随者
CharacterController
AILocomotion
SteeringForArrive 接近领队减慢速度
SteeringForLeaderFollowing 跟随领队
SteeringForSeparation 不拥挤
Radar
SteeringForEvade 挡住领队，进行躲避
EvadeController 检测是否挡住领队


排队通过狭窄通道
实现排队行为，需要两种操控行为：
靠近
避开障碍

首先，角色需要确定前方是否有其他的AI角色，然后根据这个信息来确定自己是继续前行还是停止等待
这里我们利用与碰撞避免行为中相同的方法来检测前方是否有其他AI角色。
首先计算出角色前方的ahead点，如果这个点与其他某个AI角色的距离小于一个值MAX_QUEUE_RADIUS
那么表示这个AI角色的前方有其他AI角色，必须减速或停止等待
ahead = normalize(velocity) * MAX_QUEUE_RADIUS
ahead = qa + position
那么，如何让AI角色停止等待呢？需要知道的是，操控行为是基于变化的力的，因此系统是动态的
即使某个行为返回的力为0，甚至总的力为0，还是不会让AI的速度变为0
那么如果让AI角色停止等待呢？一种方法是计算其他所有力的合力，然后像抵达行为中一样，消除这些力的
影响，让它停下来。
另一种方法是直接控制AI的速度向量，而忽略其他的力。这种方式便是此处采用的方式：直接将速度乘以一个比例因子，
例如0.2，在此因子的作用下，AI角色的移动会迅速减慢，但当前方没有遮挡时，会慢慢恢复到原来的速度
然后，再加上靠近和避开障碍行为，就大功告成了
根据具体情况，还可以加入分离行为

CharacterController
AILocomotion
SteeringForCllisionAvoidanceQueue 用来避免碰撞
SteeringForArrive
Radar
SteeringForSeparation
SteeringForQueue

SteeringForQueue
OverlapSphere检测，如果球体内角色的速度比当前角色慢，当前角色放慢速度，避免碰撞

SteeringForCllisionAvoidanceQueue
射线检测碰撞，如果有墙壁，给一个反方向的力
```

---

# 第三章 寻找最短路径并避开障碍物-A*寻路

```
A* 寻路通常有三种
基于单元的导航图，基于可视点导航图 导航网格

如果同时为一群单元寻路，A*显得吃力
对于这种情况，一种方法是将A*寻路与 群体操控行为 结合，选择一个AI作为领队
其他单元跟随头领，这样降低了计算量，群体也更加真实，不会彼此碰撞

如果小队规模比较大，或者地形复杂，上述方案不一定能正常工作，这时，可以给每一个
单元的目标位置都加上一些偏移量，这样单元就不会再接近目标时挤在一起互相碰撞。
只要对设定的目标点稍加修改，可以实现包围敌人的场景

集群移动：由于小队中的单元需要协同移动
所以需要一些代码，让这些单元以这种方式运动
雷达：用于寻找邻居
寻路与障碍避免：让单元可以找到通往目标的路径，并且避开途中的障碍物
目标点：虽然只用鼠标单击了一个点，但是如果为所有小队单元都设置相同的目标点，它们就会
彼此碰撞甚至堆叠， 所有，需要为小队中的单元生成不同的目标点，这样它们就不会相互堆叠起来
点击一个目标点，为每个AI生成不同的目标点
```

---