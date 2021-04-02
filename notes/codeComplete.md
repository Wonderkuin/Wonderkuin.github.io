### [<主页](/index.html)

# Code Complete

# Laying the Foundation
打好基础

## Welcome to Software Construction
欢迎进入软件构建的世界

#### 什么是软件构建
```
软件开发过程中的不同活动activity：
定义问题problem definition
需求分析requirements development
规划构建construction planning
软件架构software architecture 或 高层设计high-level design
详细设计detailed design
编码与调试coding and debugging
单元测试unit testing
集成测试integration testing
集成integration
系统测试system testing
保障维护corrective maintenance
```
```
构建活动中的具体任务task：
验证有关的基础工作已经完成，因此构建活动可以顺利进行下去
确定如何测试所写的代码
设计并编写类class和子程序routine
创建并命名变量variable和具名变量named constant
选择控制结构control structure，组织语句块
对你的代码进行单元测试和集成测试，并排除其中的错误
评审开发团队其他成员的底层设计和代码，并让他们评审你的工作
润饰代码，自信进行代码的格式化和注释
将单独开发的多个软件组件集成为一体
调整代码tuning code，让它更快、更省资源
```
```
一些重要的非构建活动nonconstruction activity：
管理management
需求分析
软件架构设计
用户界面设计
系统测试
维护
```

#### 软件构建为何如此重要
```
构建活动是软件开发的主要组成部分
构建活动是软件开发中的核心活动
把主要精力集中于构建活动，可以大大提高程序员的生产率
构建活动的产物-源代码--往往是对软件的位移精确描述
构建活动是唯一一项确保会完成的工作
```

## Metaphors for a Richer Understanding of Software Development
用隐喻来更充分地理解软件开发
```
形象的比喻：
病毒 virus
特洛伊木马Trojan Horse
蠕虫 worm
臭虫 bug
逻辑炸弹 bomb
崩溃 crash
论坛口水战 flame
双绞线转换头 twisted sex changer?
致命错误 fatal error
```

#### The Importance of Metaphors
隐喻的重要性
```
重要的研发成功常常产自类比analogy
这种隐喻的方法叫做建模modeling

气体分子运动理论基于所谓的撞球billiard-ball
光的波动理论参考了声音的介质，命名为以太ether

模型的威力在于其生动性，隐隐暗示各种属性properties
关系relationships，以及需要补充查证的部分additional areas of inquiry
引用以太，是过度引申了模型

好的隐喻应该更简单
伽利略之前，信奉亚里士多德的人们看的石头绑在细绳的来回摆动，
想到重物从高处坠落，落向低处并静止，而伽利略想到了钟摆pendulum
他认为石头在不断重复几乎完全相同的运动

天文学中，托勒密到哥白尼的转变
类比70年代，“以计算机为中心computer-centered”
向 “以数据库为中心database-centered”的观点转变
过去把数据处理看作流经计算机flowing through a computer
的连续卡片流stream of cards
现在把焦点放到数据池pool of data上，计算机偶尔涉足其中
```

#### How to Use Software Metaphors
如何使用软件隐喻
```
隐喻的作用更像启示heuristic，而不是算法algorithm

算法是一套明确的指令，使你能完成某个特定任务

算法是可预测的predictable，确定性的deterministic
不易变换的not subject to chance

启发式方法，试探法，是一种帮助你寻求答案的技术
但它给出的答案具有偶然性subject to chance
仅仅告诉你如何去找，而没有告诉你找什么

对于编程来说，最大的挑战还是将问题概念化conceptualizing
```

#### Common Software Metaphors
常见的软件隐喻
```
编写软件是一门科学a science
艺术an art
一个过程a process
驾驶汽车driving a car
一场游戏a game
一个集市bazaar
园艺garding
拍摄白雪公主和七个小矮人
耕田，捕猎，跟恐龙淹死在焦油坑

Software Penmanship: Writing Code
软件中的书法，写作代码

写代码隐喻成写作，太简单了
像扔进废纸篓的草稿
试错代价太昂贵

Software Farming: Growing a System
软件中的耕作法，培植系统

每次写一点代码，做一点测试
可以把麻烦减到最小
但是暗示人们无法对开发软件的过程和方式进行任何直接的控制

Software Oyster Farming: System Accretion
软件的牡蛎养殖观点，系统生长

培育growing软件，指的是软件的生长accretion
跟生长密切相关的有：
增量incremental
迭代iterative
自适应adaptive
演进evolutionary

先做尽可能简单，但能运行的版本，用虚假的类，输入
骨架形成之后，再替换成真实的类，输入

增量式开发，演进式交付
敏捷编程agile programming

优势在于未做过度的承诺

Software Constuction: Building Software
软件构建：建造软件

与写作writing，培育growing软件而言
建造building软件更有用

建造不同的建筑物，过程很不同
建什么房子：问题定义problem definition
和某个建筑师architect探讨总体设计
和软件架构设计architectural design一样
建造过程，和软件的构建construction一样
刷油漆，装修，和软件优化optimization一样
监查人员检查工地，相当于软件复查reviews和审查inspections

主要的开销在人力上，推倒和移动建筑非常昂贵

精心计划，并不是过度计划。把房屋结构性的支撑structural support规划清楚
在日后再决定用什么地板

超大型的结构一旦出现问题，后果将非常严重。因此有必要对这样的结构进行
超出常规的规划与建设over-engineered

软件架构 建筑学，architecture
支撑性测试代码 脚手架，scaffolding
构建 建设，construction
基础类 founation classes
分离代码 tearing code apart

Applying Software Techniquees: The Intellectual Toolbox
应用软件技术，智慧工具箱

技术并不是规矩rule
它只是分析工具analytical tools
知识越多，越知道何时用哪些工具

Combining Metaphors
组合各个隐喻

同时使用生长accretion和建筑construction
使用隐喻是说不清楚的事情fuzzy business
过分引申会误导你

Additional Resources
更多资源

隐喻，模型model以及范型paradigm

科学结构的变革 The Structure of Scientific Revolutions
编程范型 The Paradigms of Programming

Key Points
要点

隐喻是启示而不是算法，它们往往有点随意sloopy
隐喻把软件开发过程和你熟悉的过程联系，帮助你理解
有些隐喻比其他隐喻更贴切
通过把软件构建比喻成房屋建设，我们发现，仔细准备是必要的，大型小型项目有差异
通过把软件开发中的实践比喻成智慧工具箱中的工具，我们发现，每个程序员都有很多工具，
但并不存在任何一个能试用所有工作的工具，选择合适的工具是有效编程的关键
不同的隐喻并不彼此排斥，应当使用对你最有益处的某种组合
```

## Measure Twice, Cut Once: Upstream Prerequisites
三思而后行：前期准备
```
//TODO
```

---

## [<主页](/index.html)
