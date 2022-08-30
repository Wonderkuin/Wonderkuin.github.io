# 游戏人工智能编程案例精粹
#### Programming Game AI by Example

---

# 第一章 数学和物理学初探

```
你可以用 “剪切和粘贴” 的方式使用许多人工智能技术，但是那对你自己毫无益处；
一旦你不得不解决的问题与借来的代码有些微小差别时，你将陷入困境。
```

```
1.1.1 笛卡尔坐标系
1.1.2 函数和方程
1.1.3 三角学

勾股定理应用
A 8,4
T 2,1
max = 10
弓箭手在A，目标在P，射程max，能否射中？
设点P
AP = 4 - 1 = 3
TP = 8 - 2 = 6
TA = sqrt( AP^2 + TP^2 ) = sqrt(9 + 36) = 6.71
max > TA 能射中

三角函数
  '
  |\ h
o | \
  |__\ theta
   a

o = a tan(theta)
h = o / sin(theta)

cos(theta) = a / h
已知 a h 怎么求theta
theta = cos^-1(a/h)
反余弦函数

1.1.4 矢量

矢量分解
Oa = |v| cos(theta) = y分量
Ob = |v| sin(theta) = x分量

点乘
点乘得到两个矢量之间的夹角
u*v = ux vx + uy vy
u*v = |u| |v| cos(theta)
归一化后
u*v = cos(theta) = ux vx + uy vy

点乘的一个重要用途是它能很快地告诉你一个实体在另一个实体的
朝向平面的后面还是前面
      ^朝向
+     |
_______________朝向平面

-
如果对象在智能体朝向平面的前面，则智能体方向矢量
和从智能体到对象的矢量的点乘为正

矢量数学的实例
|      P
|    /
|   /
|  /
| /         H
|/_________________
T
巨人在T，朝向H，目标在P点，求转过多少弧度朝向P
h = H - T 归一化
p = P - T 归一化
h*p = cos(theta)
theta = cos^-1(h*p)


1.1.5 局部空间和世界空间

我们可以使用局部坐标系来变换世界坐标系，这样在世界坐标系中的所有对象都可以用
相对于它的位置和方向来描述
超出了本书范围，看图形学矩阵变换的书


1.2 物理学

时间
s

距离
m

质量
kg

位置

速度
m/s
v = delta(x) / delta(t)
delta(x) = v delta(t)

加速度
a = delta(v) / delta(t)
v = at + u 常量u代表车在t=0时的速度
一定时间内行驶的距离
u在时间t内行驶的距离
A = delta(t) u
加速度在时间t内行驶的距离
B = 0.5 (v - u) delta(t)
两个速度带来的距离和
delta(x) = u delta(t) + 0.5 (v - u) delta(t)
我们知道，v-u等于速度delta(v)的改变量
v-u = delta(v) = a delta(t)
带入公式
delta(x) = u delta(t) + 0.5 a delta(t)^2
根据 v = at + u得知
delta(t) = (v - u) / a
带入delta(x)公式得
delta(x) = u[(v - u)/a] + 0.5 a [(v-u)/a]^2
化简得
v^2 = u^2 + 2a delta(x)
这个公式特别有用
例如，一个球从大厦掉落，在坠地时速度有多快
v^2 = 0^2 + 2 * 9.81 * 381
v = sqrt(7467.6)
v = 86.41.m/s

力
N 牛 在1s内使1kg质量从静止到1m/s的速度所需要的力
F 力
a = F / m
F = ma
	acceleration = force / mass;
	velocity += acceleration * deltaTime;
	position += velocity * deltaTime;
```

---

# 第二章 状态取动智能体设计

```
有限状态机 FSM

FSM架构还将在今后很长一段时间无处不在，优点：
编程快速简单 易于调试 计算开销很少 直觉性 灵活性

定义：
一个有限状态机是一个设备，或者是一个设备模型，具有有限数量的状态，
它可以在任何给定的时间根据输入进行操作，使得从一个状态变换到另一个状态,
或者是促使一个输出或者一种行为的发生。一个有限状态机在任何瞬间只能处在一种状态

实现：
switch加if else结构，这非常糟糕
enum StateType{RunAway, Patrol, Attack};
void Agent::UpdateState(StateType CurrentState)
{
	switch(CurrentState)
	{
		case state_RunAway:
			EvadeEnemy();
			if (Safe())
			{
				ChangeState(state_Patrol);
			}
			break;
		case state_Patrol:
			FollowPatrolPath();
			if (Threatened())
			{
				if (StrongerThanEnemy())
				{
					ChangeState(state_Attack);
				}
				else
				{
					ChangeState(state_RunAway);
				}
			}
			break;
		case state_Attack:
			if (WeakerThanEnemy())
			{
				ChangeState(state_RunAway);
			}
			else
			{
				BashEnemyOverHead();
			}
			break;
	}
}
乍一看，是合理的，但是稍微复杂就非常糟糕，难以调试，不灵活。
进入状态或退出状态时，尝尝需要完成指定行动，这些不应该在Update
中调用，只能在进入、退出状态时才发生

2.2.1 状态变换表
例:
当前状态			条件				状态转换
逃跑				安全				巡逻
攻击				比敌人弱			逃跑
巡逻				受到威胁并比敌人强	攻击
巡逻				受到威胁并比敌人弱	逃跑
表中的规则在每个时间步都进行测试

2.2.2 内置的规则
将转换放入模块中，每个模块可以不依赖外部逻辑来转换到其他状态
class State
{
public:
	virtual void Execute (Troll* troll) = 0;
}
Trool类，具有成员变量表示特征，例如健康，发怒，毅力等
还有一个接口允许客户询问并且调整那些值。
一个Trool类可以被赋予有限状态机的功能性，只要增加一个指向State类继承
对象的实例的指针和允许用户改变指针指向的实例的方法。
class Troll
{
	State* m_pCurrentState;
public:
	void Update()
	{
		m_pCurrentState->Execute(this);
	}
	void ChangeState(const State* pNewState)
	{
		delete m_pCurretnState;
		m_pCurrentState = pNewState;
	}
}
//--------------------逃跑状态
class State_RunAway : public State
{
public:
	void Execute(Troll* troll)
	{
		if (troll->isSafe())
		{
			troll->ChangeState(new State_Sleep());
		}
		else
		{
			troll->MoveAwayFromEnemy();
		}
	}
};
//--------------------睡觉状态
class State_Sleep : public State
{
public:
	void Execute(Troll* troll)
	{
		if (troll->isSafe())
		{
			troll->ChangeState(new State_RunAway());
		}
		else
		{
			troll->Snore();
		}
	}
}
这称为状态设计模式，尽管偏离了数学形式化的FSM
但它是直觉的，编码简单，容易拓展。
容易增加 Enter Exit方法，调整ChangeState方法
```

```c++
//2.3 West World项目

//BaseGameEntity.cpp
class BaseGameEntity
{
private:
	int m_ID;
	static int m_iNextValidID;
	void SetID(int val);
public:
	BaseGameEntity(int id)
	{
		SetID(id);
	}
	virtual ~BaseGameEntity(){}
	virtual void Update() = 0;
	int ID() const {return m_ID;}
};

//Miner.cpp
class Miner : public BaseGameEntity
{
private:
	State* m_pCurrentState;
	location_type m_Location;
	int m_iGoldCarried;
	int m_iMoneyInBank;
	int m_iThirst;
	int m_iFatigue;
public:
	Miner(int ID);
	void Update();
	void ChangeState(State* pNewState);
};

void Miner:Update()
{
	m_iThirst += 1;
	if (m_pCurrentState)
	{
		m_pCurrentState->Execute(this);
	}
}

//四种状态
//EnterMineAndDigForNugget:
//如果矿工没有在金矿里，他改变位置。
//如果已经在金矿里，挖掘金块。
//口袋满了，改变状态到VisitBankAndDepositGold
//如果挖掘时口渴， 停下来并且改变状态到QuenchThirst

//VisitBankAndDepositGold:
//走到银行并且存储所携带的金块
//之后如果认为自己足够有钱了，会改变状态到GoHomeAndSleepTilRested
//否则会改变状态到EnterMineAndDigForNugget

//GoHomeAndSleepTilRested:
//回到小木屋睡觉直到他的疲劳等级下降到一个可接受的等级下
//然后他会改变状态到EnterMineAndDigForNugget

//QuenchThirst
//如果在任何时候感到口渴，改变状态并且到酒吧买一杯威士忌
//当解除了口渴问题， 改变状态到EnterMineAndDigForNugget

//画一张状态转换图更好，可以非常容易发现逻辑流错误

//加强的State基类
class State
{
public:
	virtual ~State(){}
	virtual void Enter(Miner*) = 0;
	virtual void Execute(Miner*) = 0;
	virtual void Exit(Miner*) = 0;
};

void Miner::ChangeState(State* pNewState)
{
	assert(m_pCurrentState && pNewState);
	m_pCurrentState->Exit(this);
	m_pCurrentState = pNewState;
	m_pCurrentState->Enter(this);
}

// 单例模式 略

//EnterMineAndDigForNugget
class EnterMineAndDigForNugget : public State
{
private:
	EnterMineAndDigForNugget(){}
public:
	static EnterMineAndDigForNugget* Instane();
	virtual void Enter(Miner* pMiner);
	virtual void Execute(Miner* pMiner);
	virtual void Exit(Miner* pMinder);
};

void EnterMineAndDigForNugget::Enter(Miner* pMiner)
{
	// 如果矿工还没有处于金矿中，他必须改变位置到金矿中
	if (pMiner->Location() != goldmine)
	{
		cout << "\n" << GetNameOfEntity(pMiner->ID()) << ": "
			 << "Walkin' to the gold mine";
		pMiner->ChangeLocation(goldmine);
	}
}

void EnterMineAndDigForNugget::Execute(Miner* pMinerJ)
{
	// 矿工挖掘寻找金子直到金子达到最大天然金块数
	// 如果在挖掘时感到口渴， 他会停止工作并且改变状态去酒吧喝一杯威士忌
	pMiner->AddToGoldCarried(1);
	//挖掘是一项很难的工作
	pMiner->IncreaseFatigue();
	cout << "\n" << GetNameOfEntity(pMiner->ID()) << ": "
		<< "Pickin' up a nugger";
	// 如果开采了足够的金子，去把它放进银行里
	if (pMiner->PocketsFull())
	{
		pMiner->ChangeState(VisitBankAndDepositGold::Instance());
	}
	//如果口渴去买了一杯威士忌
	if (pMiner->Thirty())
	{
		pMiner->ChangeState(QuenchThirst::Instance());
	}
}

void EnterMineAndDigForNugget::Exit()
{
	cout << "\n" << GetNameOfEntity(pMiner->ID()) << ": "
		<< "Ah' m leavin' the gold mine with mah pockets full o' sweet gold";
}

//2.4 使State基类可重用

template <class entity_type>
class State
{
public:
	virtual void Enter(entity_type*) = 0;
	virtual void Execute(entity_type*) = 0;
	virtual void Exit(entity_type*) = 0;
	virtual ~State(){}
};

class EnterMineAndDigForNugget : public State<Miner>
{
public:
	// 略
};
```

```
2.5 全局状态和状态翻转 State Blip

当设计一个有限状态机时，你往往会因为在每个状态中复制代码而死掉
最好设计一个全局状态，每次FSM更新时就会被调用。

全局状态，需要一个额外的成员变量
State<Miner>* m_pGlobalState;


除了全局行为之外， 偶尔让一个智能体带着一个条件进入一个状态也会带来方便，
条件就是当状态退出时，智能体返回到前一个状态。
我们称这种行为为 状态翻转 State Blip
为了赋予FSM这种类型的功能，必须保持前一个状态的记录，从而回到前一个状态
这非常容易做到， 需要在ChangeState方法中增加另一个成员变量和一些附加逻辑

class Miner : public BaseGameEntity
{
private:
	State<Miner*> m_pCurrentState;
	State<Miner*> m_pPreviousState;
	State<Miner*> m_pGolbalState;
public:
	void ChangeState<State<Miner>* pNewState);
	void RevertToPreviousState();
};
```

```C++
//2.6 创建一个StateMachine类

//通过把所有与状态相关的数据和方法封装到一个StateMachine类中，可以使得设计更为简洁。
//这种方式下一个智能体可以拥有一个StateMachine类的实例，并且委托它管理当前状态，全局状态，前面的状态

template <class entity_type>
class StateMachine
{
private:
	//指向拥有这个实例的智能体的指针
	entity_type* m_pOwner;
	State<entity_type>* m_pCurrentState;
	//智能体处于的上一个状态的记录
	State<entity_type>* m_pPreviousState;
	//每次FSM被更新时，这个逻辑状态被调用
	State<entity_type>* m_pGlobalState;
public:
	StateMachine(entity_type* owner) : m_pOwner(owner),
										m_pCurrentState(NULL),
										m_pPreviousState(NULL),
										m_pGlobalState(NULL)
	{}

	//使用这些方法来初始化FSM
	void SetCurrentState(State<entity_type>* s){m_pCurrentState = s;}
	void SetGlobalState(State<entity_type>* s){m_pGlobalState = s;}
	void SetPreviousState(State<entity_type>* s){m_pPreviousState = s;}
	//调用这个来更新FSM
	void Update() const
	{
		//如果一个全局状态存在，调用它的执行方法
		if (m_pGlobalState) m_pGlobalState->Execute(m_pOwner);
		//对当前的状态调用
		if (m_pCurrentState) m_pCurrentState->Execute(m_pOwner);
	}
	//改变到一个新状态
	void ChangeState(State<entity_type>* pNewState)
	{
		assert(pNewState && "<StateMachine::ChangeState>: trying to change to a null state");
		//保留前一个状态的记录
		m_pPreviousState = m_pCurrentState;
		//调用现有的状态的退出方法
		m_pCurrentState->Exit(m_pOwner);
		//改变状态到一个新状态
		m_pCurrentState = pNewState;
		//调用新状态的进入方法
		m_pCurrentState->Enter(m_pOwner);
	}
	//改变状态回到前一个状态
	void RevertToPreviousState()
	{
		ChangeState(m_pPreviousState);
	}
	//访问
	State<entity_type>* CurrentState() const { return m_pCurrentState; }
	State<entity_type>* GlobalState() const { return m_pGlobalState; }
	State<entity_type>* PreviousState() const { return m_pPreviousState; }
	//如果当前的状态类型等于作为指针传递的类的类型，返回true
	bool isInState(const State<entity_type)& st) const;
};

class Miner : public BaseGameEntity
{
private:
	StateMachine<Miner>* m_pStateMachine;
public:
	Miner(int id) : m_Location(shack),
					m_iGoldCarried(0),
					m_iMoneyInBank(0),
					m_iThirst(0),
					m_iFatigue(0),
					BaseGameEntity(id)
	{
		m_pStateMachine = new StateMachine<Miner>(this);
		m_pStateMachine->SetCurrentState(GoHomeAndSleepTilRested::Instance();
		m_pStateMachine->SetGlobalState(MinerGlobalState::Instance());
	}
	~Miner() {delete m_pStateMachine;}
	void Update()
	{
		++m_iThirst;
		m_pStateMachine->Update();
	}
	StateMachine<Miner>* GetFSM() const {return m_pStateMachine;}
};
```

```
2.7 引入Elsa

VisitBathroom 状态总是回到前面的状态，
全局状态包含了对VisitBathroom的检测
```

```
2.8 为FSM增加消息功能

当一个事件发生了，事件被广播给游戏中相关对象
事件驱动结构被普遍选取的原因是因为它们是高效率的
没有事件处理，对象不得不持续地检测游戏世界来看是否有一个特定的行为已经发生了
使用事件处理，对象可以简单的继续它们自己的工作，直到有一个事件消息广播给他
```

---

# 第三章 如何创建自治的可移动游戏智能体

```
3.4.4 Pursuit 追逐

当智能体要拦截一个可移动目标时，pursuit行为就变得很有用
当然它能向目标的当前位置靠近，但是这不能制造出智能的假象。
如果你抓人，不会直接移动到它的当前位置，而是预测它的未来位置，然后
移向那个偏移位置，期间通过不断调整来缩短差距。

pursuit函数的成功与否取决于追逐者预测逃避着的运动轨迹有多准
这可以做的很复杂，所以做个折中以得到足够的效率，但又不消耗太多时钟周期
追逐者可能会碰到一种提前结束enables an early out 的情况
如果逃避者在前面，几乎面对智能体，那么智能体应该直接向逃避者当前位置移动
这可以通过点积快速算出（参考第一章）。
在示例代码中，逃避者朝向的反方向和智能体的朝向必须在近似20度内，才被认为是面对着的
一个好的预测的难点之一就是决定预测多远。很明显，这个多远应该正比于追逐者与逃避者的距离
反比于追逐者和逃避者的速度，一旦多远确定了，我们就可以估算出追逐者seek的位置
```

```c++
Vector2D SteeringBehaviors::Pursuit(cosnt Vehicle* evader)
{
	//如果逃避者在前面，而且面对着智能体
	//那么我们可以正好靠近逃避者的当前位置
	Vector2D ToEvader = evader->Pos() - m_pVehicle->Pos();

	double RelativeHeading = m_pVehicle->Heading().Dot(evader->Heading());

	if ((ToEvader.Dot(m_pVehicle->Heading() > 0) &&
		(RelativeHeading < -0.95)) //acos(0.95)=18 degs
	{
		return Seek(evader->Pos());
	}

	//预测逃避者的位置
	//预测的时间正比于逃避者和追逐者的距离，反比于智能体的速度
	double LookAheadTime = ToEvader.Length() / (m_pVehicle->MaxSpeed() + evader->Speed());

	//现在靠近逃避者的被预测位置
	return Seek(evader->pos() + evader->Velocity() * LookAheadTime);
}

// 一些运动模型可能需要考虑智能体转向偏移位置所需的时间
// 通过给LookAheadTime加上一个正比于两个朝向向量点击和最大转弯率的值
LookAheadTime += TurnAroundTime(m_pVehicle, evader->Pos());

double TurnaroundTime(const Vehicle* pAgent, Vector2D TargetPos)
{
	//确定到目标的标准化向量
	Vector2D toTarget = Vec2DNormalize(TargetPos - pAgent->Pos());
	double dot = pAgent->Heading().Dot(toTarget);
	//改变这个值到预期行为
	//交通工具的最大转弯率越高，这个值越大
	//如果交通工具正在朝向到目标位置的反方向
	//那么0.5意味着这个函数返回1秒的时间以便让交通工具转弯
	const double coefficient = 0.5;
	//如果目标直接在前面，那么点击为1
	//如果目标直接在后面，那么点击为-1
	//减去1，除以负的coefficient，得到一个正的值
	//且正比于交通工具和目标的转动角位移
	return (dot - 1.0) * -coefficient;
}
```

```
3.4.12 Offset Pursuit 保持一定距离的追逐

计算出一个操控力，使交通工具与目标交通工具之间保持指定偏移。
这对于创建编队特别有用。

偏移总定义在 领头飞机 空间，所以当计算这个操控力时，我们要做的
第一件事是得到在世界空间的偏移位置。然后如同pursuit那样处理：
预测下一个偏移位置，交通工具需要到达那个位置
```

```c++
SVector2D SteeringBehaviors::OffsetPursuit(const Vehicle* learder, const SVector2D offset)
{
	//在世界空间中计算偏移的位置
	SVector2D WorldOffsetPos = PointToWorldSpace(offset, leader->Heading(), 
												leader->Side(), leader->Pos());
	SVector2D ToOffset = WorldOffsetPos - m_pVehicle->Pos();

	//预期的时间正比于领队与追逐者的距离
	//反比于两个智能体的速度之和
	double LookAheadTime = ToOffset.Length() / (m_pVehicle->MaxSpeed() + leader->Speed());

	//现在到达偏移的预测位置
	return Arrive(WorldOffsetPos + leader->Velocity() * LookAheadTime, fast);
}
//Arrive被用来替代seek，因为它能提供更流畅的动作，且与最大速度和最大力的设置无关。
//seek有时能给出一些非常奇异的结果（使有序的编队变得看上去像一群蜜蜂攻击一个编队领头一样）。
//Offset pursuit适用于各种各样的情况，下面列出一些
```

```
3.5 组行为 Group Behaviors

组行为是考虑游戏世界中的一些或者所有的其他交通工具的操控行为
群集行为，是三个组行为：Cohesion Separation Alignment
为了得到组行为的操控力，交通工具将在一个以自我为中心且以一个预定义尺寸半径的圆形
区域内（邻近半径）考虑其他所有交通工具。
在计算操控力之前，我们必须先得到交通工具的邻居，或者存储在一个容器中，或者做个标记准备处理
这里用BaseGameEntity::Tag标记交通工具的邻居
```

```c++
template<class T, class conT>
void TagNeighbors(const T* entity, conT& ContainerOfEntities, double raidus)
{
	//迭代所有实体，检查范围
	for (typename conT::iterator curEntity = ContainerOfEntities.begin();
		curEntity != ContainerOfEntities.end();
		++curentity)
	{
		//首先清除当前的标记
		(*curEntity)->UnTag();

		Vector2D to = (*curEntity)->Pos() - entity->Pos();

		//考虑其他的包围半径 把它添加到范围
		double range = radius + (*curEntity)->BRadius();

		//如果实体在范围内，标记它，作进一步考虑
		//（为了避免开方，用平法计算）
		if (((*curEntity) != entity) && (to.LengthSq() < range*range))
		{
			(*curEntity)->Tag();
		}
	}//下一个实体
}
//大部分组行为都使用相似的临近半径，所以我们在调用任何组行为之前，
//先调用这个方法，从而节约一些时间
if (On(separation) || On(alignment) || On(cohesion))
{
	TagNeighbors(m_pVehivle, m_pVeicle->World()->Agents(), ViewDistance);
}

//提示：可以给智能体增加可视域 field-of-view
//比如 在操控智能体朝向的270范围内，你可以通过测试智能体的朝向向量与到潜在
//邻居的向量的点积，容易的实现这个功能
//FOV可以动态调整：战争游戏中，FOV受到疲劳的影响
```

---