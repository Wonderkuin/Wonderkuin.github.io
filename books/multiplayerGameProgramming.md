# 网络多人游戏架构与编程

---

# 第一章 网络游戏概述

```
星际围攻：部落 1998

1 非保障数据
	不是游戏所必须的数据，带宽有限时，首先丢弃这些数据
2 保障数据
	需要保证其准确性和到达顺序，用于对游戏至关重要的数据
	例如：标识玩家发起攻击的事件
3 最近的状态数据
	该类数据用于只有最新版本的数据才是重要数据的场合
	例如：一个特定玩家的生命值。
	如果游戏知道了玩家的生命值，那么他五秒之前的生命值就不重要了
4 最快保障数据
	该类数据具有最高的优先级，在可靠传输的基础上，保证尽快到达
	例如：玩家的移动信息
	该类信息在一个非常短的时间内极其重要，因此需要尽快传输

架构 客户端-服务器模型 client-server model
	对等网络模型peer-to-peer model需要 O(n^2)的带宽
	服务器只需处理O(n)量级的带宽

网络模型:
					游戏模拟层
	ghost管理器 移动管理器 事件管理器 其他
					流管理器
					连接管理器
					平台数据包模块

平台数据包模块
	数据包是在网络中传输的有一定格式的数据的集合
	平台数据包模块是最底层，只有这一层是针对特定平台的
	其实，这一层是标准套接字API的封装，可以构建和发送不同的数据包格式
	星际围攻：部落 使用UDP，实现了定制的可靠层
	但是，该可靠层不是在平台数据包模块中处理的，而是在更高层管理器
	如：ghost管理器 移动管理器 事件管理器中来增强可靠性

连接管理器 connection manager
	将网络中两台计算机的连接抽象化
	它从上层流管理器接收数据，再将数据传输给底层平台数据包模块
	连接管理器仍然是不可靠的，它不能保证数据的可靠传输。
	但是，连接管理器可以保证投递状态通知的正确传输
	也就是说，可以确认传输到连接管理器层的请求状态。
	这样，连接管理器的上层，流管理器，就可以知道指定的数据是否被成功传输
	投递状态通知使用滑动窗口中接受域的位字段实现

流管理器 stream manager
	流管理器的任务是将数据发送给连接管理器
	其中一个很重要的部分是决定允许数据传输的最大速率
	最大速率会根据网络连接质量有所不同
	因为许多其他系统要求流管理器发送数据，流管理器有责任把这些请求按照优先次序排好
	在带宽限制的情况下，移动管理器，事件管理器，ghost管理器会分派给连接管理器
	接着，高层管理器将会通过流管理器的投递状态得到通知

事件管理器 event manager
	事件管理器维持着一个由游戏模拟层产生的事件队列
	这些事件可以看作是远程过程调用 remote procedure call RPC 的一种简单形式
	RPC是可以在远程计算机上执行的程序
	事件管理器也会追踪每一个被标记为可靠数据的传输记录
	用这种方法，事件管理器很容易实现可靠性
	如果可靠记录没有被确认，那么事件管理器重新将该事件放入事件队列中，重新传输一次
	当然也有很多被标记为不可靠的数据，对于这些数据，则不需要追踪它们的传输记录

ghost管理器
	ghost管理器的工作是复制被认为与指定客户端相关的动态对象
	换句话说，服务器给客户端发送关于动态对象的信息，但是仅仅是服务器认为客户端需要知道的对象
	游戏模拟层负责决定客户端必须知道什么以及最好知道什么。
	这赋予了游戏对象一个固有的优先级：必须知道 最好知道
	不管相关对象的集合是如何计算出来的
	ghost管理器的任务是从服务器向客户端传输尽可能多的相关对象的状态
	ghost管理器来保证最近的数据总是能成功地传输到所有客户端，对于系统来讲是非常重要的
	这里游戏对象的信息通常包括健康状态，武器，弹药数量等，对于这些信息来说，只有最近的信息才有用
	当一个对象成为相关对象时，ghost管理器将该对象赋予一些信息，这里成为ghost记录
	该记录包括唯一的id，状态掩码，优先级，状态变换
	对于ghost状态的传输，对象的优先级首先由状态的变换决定，其次由优先级决定

移动管理器
	移动管理器的任务是尽快传输玩家的移动数据
	赋予移动管理器较高的优先级的另外一个重要原因是输入数据的捕捉速度是30FPS
	所以最近的数据应该尽快发送出去，更高的优先级意味着，当移动数据可用时，
	流管理器总是首先给出站数据包添加所有的移动管理器数据。
	每一个客户端负责传输他们的移动信息到服务器，之后服务器在游戏模拟中使用这些移动信息
	然后通知客户端这些移动信息的接收情况

其他系统
	例如，数据块管理器，本质上时游戏对象的传输，与ghost系统所处理的动态对象不同
	好比一个静态工具如炮塔，该对象不会移动，但是在与玩家交互方面时有用的
```

```
帝国时代 1997

确定性锁步网络模型 deterministic lockstep
所有计算机相互连接，即这个模型是对等网络模型
每个节点都同时运行一个有保证的确定性游戏模拟
锁步是因为节点之间使用通信机制来确保游戏过程中保持同步

RTS与FPS的不同在于相关节点数量
	星际围攻：部落中，即时有多大128个玩家，但是在任何特定的时间点
	与某个客户端相关的玩家只有一小部分，这意味着ghost管理器很少需要一次发送多于二三十个ghost
	帝国时代 正好相反，尽管玩家数量小得多，但是每个玩家可以控制大量的游戏单元
	8个玩家，每个玩家可以控制50个单元，意味着同时有400个活跃单元
	8个玩家，每个玩家控制200个单元，意味着同时可以有1600个活跃单元

	如果某种相关性系统可以减少需要同步的单元，但是如果需要同时同步所有活跃单元，将会怎么样？
	星际围攻的模型将很难保持同步。

	为了解决这个问题，帝国时代的工程师决定同步玩家的命令，而不是同步单元。
	即使是专业的RTS玩家，每分钟也不可能发出多于300条命令。对带宽的需求更加可控
	然而，因为游戏不再在网络中传输单元信息，游戏的每一个实例需要独立地执行每个玩家发出的命令
	因为每个游戏实例执行独立的模拟，那么极为重要的是，每个游戏实例需要与其他游戏实例保持同步
	这是在实现确定性锁步网络模型中最大的挑战

轮班计时器
	将命令存储在一个队列中，在轮班计时器方法中，首先选择轮班的长度
	帝国时代中，默认的时长是200ms
	200ms内的所有命令存储在一个缓冲区中，当200ms结束时，这个玩家的所有命令将
	通过网络传输给其他所有玩家。
	这个系统的另一个关键点是两个轮班之间执行延迟。
	例如，一个玩家在50轮发出的命令直到52轮才被执行。
	在200ms轮班计时器的情况下，这意味着输入延迟，即从一个玩家发出命令到其
	作用在屏幕上，可能需要高达600ms
	然而，两轮的宽限为其他玩家接收和确认某一轮的指令提供了充足的时间。
	在轮班计时器方法中，有一个非常重要的边界情况需要考虑
	如果其中的一个玩家经历了一个滞后尖峰，导致所有玩家跟不上200ms计时器，怎么办？
	一些游戏可能暂时停止模拟，来看是否可以解决滞后尖峰问题。
	最后，如果他继续减缓其他玩家的游戏速度，可能被强行退出。
	帝国时代也试图通过基于网络情况动态调整渲染帧率的方法来弥补这一情况，
	这样连接在一个非常慢的网络中，计算机可以为网络数据接收分配更多的时间
	而花更少的时间来渲染图像
	传输由客户端发出的命令还有一个好处，这种方法只需要很少的额外内存和工作了保存整个比赛过程中发出的命令。
	这使得实现可保存的比赛重播成为可能

同步
	仅仅使用轮班计时器方法不能保证节点之间的同步
	利用伪随机数生成器 pseudo-random number generator PRNG保证玩家的随机结果一样
	优点：减少了玩家作弊的机会，能立刻检测出不同步
	缺点：得到不可见的信息，每个实例都模拟游戏中的每个单元，地图作弊
```

---

# 第二章 互联网

```
电路交换 通过连续电路传输信息，一个时刻只能用于一个目的
分组交换 将传输的信息拆分为小块，成为分组（数据包）
	基于 存储转发 技术将他们发送到共享的线路中。
	每个节点存储到来的分组，然后转发给距离目的地更近的节点。

TCP/IP模型
	本书抽象为5层
	物理层 链路层 网络层 传输层 应用层

	每一层
	接收上一层数据
	通过添加头部，有时尾部，对数据进行封装
	将数据转发到下一层做进一步传输
	接收下一层传输来的数据
	去掉报头，解封装传输来的数据包
	将数据转发到上一层做进一步的处理

物理层
	最基本的支持层，物理介质
链路层
	真正发挥作用的开始，提供一种网络实体之间通信的方法
	即链路层必须提供一种方法，该方法可以实现源主机封装信息，
	通过物理层传输信息，目的主机接收封装好的信息并从中提取所需的信息
	链路层的数据传输单元成为 帧
	实体之间通过链路层彼此发送帧数据，更具体的说，链路层职责包括
		定义主机的唯一标识方法，方便帧数据对接收方进行编址
		定义帧的格式，包括目的地址的格式和所传输数据的格式
		定义帧的长短，以便确定上层每一次传输所能发送的数据大小；LB
		定义一种将帧转换为电子信号的物理方法，以便数据可以通过物理层传输并被接收方接收
	帧的传输是不可靠的，有许多因素影响
	链路层不做任何操作来确认帧是否抵达接收方或者保证如果帧没有抵达重新发送
	因此，链路层痛惜不可靠，需要顶层协议实现
	每种物理介质都需要对应的协议或协议族来提供链路层所需要的服务
	由于链路层实现和物理层介质关系紧密，一些网络模型将两个层合并
	然而，因为一些物理介质支持的链路层协议不止一种，所以不合并更好
	传输一个数据包可能同时用到多种介质和链路层协议
	以太网 Ethernet 链路层协议族 需要重点说明

Ethernet/802.3
	以太网不是一个协议，而是基于以太网蓝皮书的一组协议。
	现代以太网协议是在IEEE 802.3基础上定义的
	为了给每台主机一个唯一标识，以太网引入了介质访问控制地址
	media access control address MAC地址
	MAC地址理论上是一个48比特数字，唯一分配给连接在以太网网络
	中的每个硬件。
	这个硬件通常是网卡 network interface controller NIC
	为了保证MAC地址的唯一性，网卡生产商在硬件的生产过程中就将MAC地址
	烧到了网卡中
	MAC地址的前24bit叫做 组织唯一标识符
	organizationally unique identifier OUI
	是由IEEE给厂家分配的唯一代码，后24bit是厂家自己分配的，来保证唯一性
		MAC地址不再可靠，许多网卡允许软件修改MAC地址
		IEEE引入了64bitMAC地址，称为extended unique identifier EUI64
			通过在OUI右侧插入两个FFFE将48bit转换位64bit
	分配了MAC地址之后，定义了以太网链路层帧的格式
	Bytes
	0~7 前导序列 帧开始标志
	8~13 目的地址
	14-21 源地址 帧长度/类型域
	22~...数据 46-1500字节
	... 帧校验序列
		虽然没有统一标志，许多现代以太网网卡支持最大传输单元大于1500字节
		这些巨型帧jumbo frame通常可高达9000字节
		为了支持这个长度，他们在帧头指定一种以太网类型，然后基于传入数据
		同时依赖底层硬件来计算帧大小
	帧校验序列 frame check sequence FCS是由两个地址域，帧长度/类型域
	数据域和其他填充信息生成的循环冗余校验cyclic redundancy check
	CRC32值，尽管以太网是不可靠传输，但还是尽力防止错误数据的传输

	交换机会记录MAC地址，有时是IP地址，所以大部分数据包
	可以尽可能地选择最短路径到达接收方，不需要访问网络中每台主机

网络层
	1 MAC限制了硬件灵活性，需要一个基于MAC地址之上的，可轻松配置的地址系统
	2 链路层不支持将互联网划分成更小的局域网，大规模数据包将导致整个网络瘫痪
	3 链路层不支持使用不同链路层协议进行主机通信，再一次，需要地址系统
	网络层的任务是在链路层的基础上提供一套逻辑地址的基础设施

	IPv4 internet protocol version 4
		定义了一个为每台主机标识的逻辑寻址系统，一个定义地址空间的逻辑分段
		作为物理子网的子网系统，一个在子网之间转发数据的路由系统
		IPv4是32bit数字 xxx.xxx.xxx.xxx
		IPv4数据包
		Bits
		0~31 版本号 IP包头长度 服务类型 IP包总长
		32~63 标识符 标记 片便宜
		64~95 生存时间 协议 头部检验和
		96~127 源地址
		128~159 目标地址
		160~... 可选项

	直连路由和地址解析协议
		IPv4允许数据包的目标地址是IP地址，使用链路层发送数据包时，
		帧中包含的是链路层能够理解的地址。
		以太网模块不能处理只包含IP地址的数据包，因为IP不是链路层的概念。
		链路层需要一些方法来找出IP对应的MAC地址，链路层协议提供了一种方法
		地址解析协议 address resolution protocol ARP
			ARP在技术上是一个链路层协议， 因为它直接使用链路层的地址形式
			发送数据包，而不需要网络层提供的路由。
			然而，由于该协议包括了网络层的IP地址，破坏了网络层的抽象
			所以不仅仅是一个链路层协议
		当网络层需要使用链路层向一台主机发送数据包时，首先查询ARP对应表获取目标IP地址所对应的MAC地址。如果找到MAC地址，那么IP模块使用该MAC地址
		构造一个链路层帧，将该帧发送给链路层实现传输。
		但是，如果表中没有找到映射，那么ARP模块通过给链路层网络中所有可达的
		主机发送ARP报文，来获得对应的MAC地址

	子网和间接路由
		两个公司各有100台电脑， 如果直接在链路层相连，互联网流量会增加1倍。
		网络层引入了一种在链路层并不直接相连接的主机间路由数据包的机制。
		A1 18.19.100.2 A2 18.19.100.3 A3 18.19.100.4
				Host R 路由器
				HIC 0 IP: 18.19.100.1
				NIC 1 IP: 18.19.200.1
		B1 18.19.200.2 B2 18.19.200.3 B3 18.19.200.4

		子网掩码 subnet mask
			一个32bit数，通常写成xxx.xxx.xxx.xxx
			如果主机的IP地址和子网掩码做按位运算的结果相同
			那么这些主机在同一个子网中
		A1 18.19.100.1 255.255.255.0 18.19.100.0
		A2 18.19.100.2 255.255.255.0 18.19.100.0
		B1 18.19.200.1 255.255.255.0 18.19.200.0
		无类别域间路由 classless inter-domain routing CIDR
			18.19.100.0/24

		子网定义之后 IPv4规范提供了一种在不同网络的主机之间传输数据包的方法
		实现该方法的关键是每台主机IP模块中的路由表routing table
		具体来说，当一台主机的IPv4模块向另一台远程主机发送数据包时，
		首先决定是使用ARP表及直连路由，还是使用间接路由。
		为了辅助这个过程，每个IPv4模块包含一个路由表。
		对于每一个可达的目标子网，路由表包含一行信息，内容是如何将数据包发送
		到这个子网。

		路由表中，目标子网指的是包含目标IP地址的子网
		网关是指在当前子网通过链路层发送数据包的下一台主机的IP地址，要求这台主机通过直达路由可达
		如果网关域为空，表示整个子网是直达路由可达的，数据包可以直接通过链路层发送
		最后，网卡指的是转发数据包的网卡。通过这个机制，数据包可以通过一个链路层网络接收，再转发到另一个链路层网络

		首先要从互联网服务提供商 internet service provider ISP
		获得一个有效的IP地址和网关。
		假设ISP分配一个IP地址 18.181.0.29和网关18.181.0.1
		那么网管必须在主机R上安装额外的网卡，使用这个分配的IP地址进行配置
		A1 18.19.100.2 A2 18.19.100.3 A3 18.19.100.4
				Host R 路由器
				HIC 0 IP: 18.19.100.1
				NIC 1 IP: 18.19.200.1
		ISP-----NIC 2 IP: 18.181.0.29
		B1 18.19.200.2 B2 18.19.200.3 B3 18.19.200.4

		数据包每经过一个路由器，IPv4头部的生存时间TTL的值减1
		当TTL为0，路由器会丢弃IP数据包，避免无限循环和收发
		TTL为0不是数据包被丢弃的唯一原因，如果数据包到达路由器网卡
		各种原因，网卡来不及处理，可能会忽略他们
		网络层的所有协议，都是不可靠的
		不保证能到达目的地址，也不保证顺序

		127.0.0.1 回路地址 loopback 或者本地地址 localhost address
			不会发送到任何地方
		255.255.255.255 广播地址 zero network broadcast address
			数据包会发送到相同链路层网络所有主机，但不被路由器发送

	分片
		以太网帧的最大传输单元 maximum transmission units MTU 是1500字节
		IPv4包最大传输单元是 65535字节
		这带来一个问题：如果IP数据包必须封装为链路层帧来传输，IP数据包的长度比链路层最大传输单元长怎么办？
		分片 fragmentation
		分割成一些数据包长度为链路层最大传输单元的小片段
		IP分片数据包与普通的IP数据包类似，只需要在头部设置一些值
		片段标识符 fragment identification
		片标记 fragment flag
		片偏移 fragment offset
		当IP模块将IP数据包分割成一组小的片段时，为每一个片段创建一个新的IP数据包，并设置这些域的值
		版本号 version
		IP数据包包头长度 header length
		IP数据包的包总长 total length
		标识符 identification
		片标记 fragment flags
		片偏移 fragment offset
		生存时间 time to live
		协议 Protocol
		源地址 Source Address
		目标地址 Destination Address
		数据 Payload

		这些分片数据包被发送出去后，他们中的某些或者全部都有可能进一步被分片
		如果到达目的主机的道路上经过最大传输单元更小的链路层，就会发生这种情况

		尽管IP数据包分片技术使得发送大数据包称为可能，但存在两种低效的情况：
		第一：增加了网络上发送的数据量
		第二：如果一个分片丢失，那么整个数据包必须丢弃，
			这意味着大数据包丢失分片数据包的概率更大。
			所以，建议通过保证所有IP数据包长度都小于链路层的最大传输单元
			尽量避免使用分片技术
			游戏开发者做这样的近似：数据包MTU的最小值为1500字节。
			这1500字节必须包括20字节的IP头，IP数据和任何协议
			例如VPN或者IPSec，需要使用的额外数据。
			所有，最好将IP包的数据限制在1300字节内

	IPv6
		32bit地址IPv4允许40亿个不同的IP地址。
		通过本地网络和网络地址转换，比40亿更多的主机连接在互联网上。
		虽然如此，由于IP地址分配的方式和笔记本，移动设备，和物联网的发展，IPv4被用完了

		完整形式
		2001:4a60:0000:8f1:0000:0000:0000:1013
		前导零压缩法
		2001:4a60:0:8f1:0:0:0:1013
		双冒号法
		2001:4a60:0:8f1::1013

		前64bit表示网络 称为网络前缀 prefix
		剩下的64bit表示个体主机 称为 接口ID interface identifier
		每台主机有一个固定的ip地址很重要，例如当这台主机作为服务器时，
		网络管理员需要手动设置接口ID，与IPv4中手动设置IP地址一样。
		一台不需要远程客户端很容易找到的主机也可以随便设置接口ID，并向网络公布
		因为64bit地址空间发生冲突的概率很低。通常，接口ID自动设置为网卡的64bitEUI，因为它已经保证了唯一性。

		邻居发现协议 neighbor discovery protocol NDP
		代替了地址解析协议ARP和动态主机配置协议DHCP的一些功能
		使用NDP，路由器公布它们的网络前缀和路由表信息，主机查询和宣布它们的IP地址和链路层地址。
		针对IPv4的另一个改进是，IPv6不再支持路由层面的数据包分片技术。
		所以删除了IP头部所有与分片技术相关的域，节省了每个数据包的带宽。
		如果一个IPv6数据包到达路由器，发现对于链路层来说太大，那么路由器直接丢弃这个数据包，告知发送方数据包太大。
		由发送方来决定使用小一些的数据包重新发送。
```

```
传输层 transport layer
	网络层的任务是实现远程网络上两台遥远主机之间的通信
	传输层的任务是实现这些主机上单独进程之间的通信。
	因为一台主机上同时运行很多进程，只知道主机A给B发送了一个IP数据包远远不够。
	主机B收到这个数据包时，它需要发送给哪个进程做进一步处理。
	为了解决这个问题，传输层引入了端口 port 的概念
	端口 16bit无符号数，是一台特定主机的通信端点。
	一般一个进程绑定一个端口，如果多个进程绑定同一个端口，需要特定标识。
	为了避免进程争夺端口，互联网名称域数字地址分配机构
	Internet Corporation for Assigned Names and Numbers ICANN
	也成为 互联网数字分配机构
	Internet Assigned Numbers Authority IANA
	负责端口号的注册，任何协议和应用开发者都可以注册所需要的端口。
	每个传输层协议只能注册一个端口号。
	端口号1024~49151称为 用户端口 user port
	或者 注册端口 registered port
	任何协议和应用开发者可以向IANA申请这个范围的端口号
	审核之后，这个端口注册就被授予了。
	如果一个用户端口号已经被IANA注册给一个特定的应用或协议
	那么其他应用或协议想要绑定这个端口都是不合法的。
	尽管大部分的传输层协议没有保证这一条
	端口0~1023称为系统端口system port或者预留端口 reserved port
	这些端口和用户端口类似，但是IANA对这些端口的注册要求更近严格，
	需要彻底的审查。
	这些端口特殊，因为大部分操作系统只允许root级别的进程才能绑定系统端口
	端口49152~65535称为动态端口dynamic port
	IANA不负责这些端口的注册，任何进程使用它们都是公平的。
	如果一个进程试图绑定一个动态端口，发现该端口被占用，那么应该尝试
	查询其他动态端口，直到找到一个没被占用的 端口为止。

	应该仅仅在建立多人游戏时使用动态端口，必要时向IANA申请一个用户端口的注册。
	一旦应用程序已经确定了一个可以使用的端口，它必须使用一个传输层协议才能发送数据。
	作为游戏开发者，我们主要使用TCP UDP

	名称										缩写 协议号
	传输控制协议 transmission control protocol TCP 6
	用户数据报协议 user datagram protocol UDP 17
	数据报拥塞控制协议 datagram congestion control protocol DCCP 33
	流控制传输协议 stream control transmission protocol SCTP 132

	UDP
		轻量级协议，封装数据并将其从一台主机的一个端口发送到另一个主机的一个端口
		UDP数据报包含一个8字节的报头，后面跟着数据。

		UDP 报头
		Bits 0						16
		0~31 源端口号				目标端口号
		32~63数据报长度				检验和

		源端口号 Source Port 16bit
			标识数据发送方将UDP数据报发送出去的端口
			当数据报接收方需要响应的时候，这个域非常有用
		目标端口号 Destination Port 16bit
			数据报的目标端口
			UDP模块将数据报发送给与这个端口绑定的进程
		数据报长度 Length 16bit
			是指包括报头和数据部分在内的总字节数
		检验和 Checksum 16bit
			由UDP报头，数据部分，IP头的某些域计算得到
			可选项，如果不做计算，取值为0
			如果底层验证了数据，这个域可以被忽略

		UDP是一个非常廉价的协议。
		每个数据报都是一个独立的实体，两台主机之间没有依赖的共享状态。
		UDP不提供阻塞网络的流量限制服务，不保证数据顺序传输和准确到达。

	TCP
		在两台主机之间创建持久性的连接，提供可靠数据流传输。
		这里的关键词是可靠。
		TCP保证所有的数据都按序抵达接收方。
		为了做到这一点，它需要比UDP更大的头部数据和用于跟踪连接中每台主机的重要连接数据。
		接收者确认接收到的数据，发送者重新发送没有收到确认消息的数据。

		TCP的数据传输单元称为TCP报文段 segment，指的是TCP用于传输大量的字节流，
		底层数据包封装这个数据流的每个单独的报文段。
		一个报文段包含TCP首部和段内数据部分
		源端口号 Source Port 16bit
		目标端口号 Destination Port 16bit
		序列号 Sequence Number 32bit
			是一个单调递增数字，通过TCP所传输的每个字节都有一个连续的序列号
			用于这个字节的唯一标识。
			这样，发送方可以标记所发送的数据，接收方可以确认。
			报文段的序列号是本报文段所发送的数据的第一个字节的序号。
			有一个例外是建立初始连接，三次握手。

		TCP首部
		Bits 0		4		7				16
		0~31源端口							目的端口
		32~63					序列号
		64~95					确认号
		96~127数据偏移	保留	控制位		接收窗口
		128~159		检验和					紧急指针
		160~...					选项

		确认号 Acknowledgment Number 32bit
			包含发送方期望收到的下一个字节的序列号
			对所有序列号低于这个数字的数据做一个实际的确认：
			因为TCP保证所有数据都是按序传输，主机期望收到的下一个字节的序列号
			通常比刚刚收到的前一个字节的序列号多1
			一定要记住：
			这个数字的发送方并不是确认收到这个值对应的序列号，
			而是所有小于这个值的序列号
		数据偏移 Data Offset 4bit
			表示以32bit为单位的TCP头部大小。
			TCP允许头部的最后添加一些可选的头部元素，所以从头部开始到报文段
			可以取值从20到64字节
		控制位 Control Bits 9bit
			是关于头部的元数据
		接收窗口 Receive Window 16bit
			表示对于传入的数据，剩余缓冲空间的最大容量
			对于流控制非常有用
		紧急指针 Urgent Pointer 16bit
			表示TCP段数据的第一个字节和紧急数据的第一个字节之间的距离
			只有再控制位中URG标志设置了时才有效

		许多RFC，包括定义主要传输层协议的，都明确地规定8bit大小的数据块
		是一个位组 octet，而不是使用松散地定义一个字节为8bit
		一些使用时间过长而难以维护的平台使用包含比8bit多或少的字节
		位组的标准化帮助确保这些平台之间的兼容性
		这不重要，游戏开发者相关的所有平台中，一个字节都是8bit


	可靠性
		数据接收
			包含ACK
				记录确认的序列号
					包含期望的序列号
						转发给应用层
							期望的序列号加1
		存在很久之前发送的还没有被确认的数据
			重新发送
			发送新数据
				发送，记录发送的时间

	三次握手
		Send Next SDN.NXT 主机发送的下一个报文段的序列号
		Send Unacknowledged SND.UNA 主机发送的尚未确认的最早报文段的序列号
		Send Window SND.WND 在收到未确认数据的确认前，允许主机发送的数据量
		Receive Next RCV.NXT 主机期望收到的下一个报文段的序列号
		Receive Window RCV.WND 在缓冲区不溢出的情况下，当前主机能够接收的数据量

		Host A											Host B
		SND.NXT  RCV.NXT  SND.UNA						SND.NXT  RCV.NXT  SND.UNA
		1000
		发送
									Seq#: 1000
									SYN
														接收
														3000	 1001
														发送
									Seq#: 3000
									Ack#: 1001
									SYC, ACK
		接收
		1001	 3001	  1001
		发送
		1001	 3001	  1001
									Seq#: 1001
									Ack#: 3001
									ACK
														接收
														3001      1001      3001

		当TCP报文包含一个SYN标志或者FIN标志，序列号额外加1
		有时被称为TCP 幻影字节 TCP phantom byte

	数据传输
		为了传输数据，主机在每个即将发送的报文段中包含数据载荷。
		每个报文段标记为数据中的第一个字节的序列号。
		每个字节都有一个连续的序列号，这意味着报文段的序列号应该是上一个报文段
		的序列号加上上一个报文段的数据量。
		每次报文段到达目的地，接收方都要发送一个确认数据包，包含一个取值为期望收到
		的下一个序列号或者确认域。

		TCP保证数据按序到达，如果主机收到的序列号不是期望的
		可以直接丢弃数据包，等待重传
		可以缓存它，同时不确认这个报文，也不转发给应用层处理，等待之前的报文到达再确认这个数据报
			并发送到应用层处理

		实际上，TCP并不是等待报文确认之后再发送下一条，这样将完全无用。
		TCP允许一次有多个未被确认的报文同时传输。但是不得不限制报文段的数量，因为会带来一个问题：
			当传输层数据抵达主机，将存储在主机的缓冲区中，直到绑定在相应端口的进程来处理它。
			缓冲区是有大小的，发送过快，来不及处理，数据会被丢弃，将导致网络阻塞，浪费互联网资源。
		TCP实现了 流量控制 flow control
			防止一台快速传输的主机压制另外一台处理较慢的主机。
			TCP头部包含一个接收窗口域，指明数据发送方有多少可用的接收缓冲区。
			主机B总提醒主机A它可以接收的数据量，主机A绝不能发送比主机B可以缓存的数据量更多的数据。
			TCP数据流的理论带宽限制：
						接收窗口
			带宽限制 X ——————————
						往返时间
			接收窗口太小会称为TCP的瓶颈，这样理论带宽最大值总比链路层的最大传输速率要大。
		TCP实现了 拥塞控制 congestion control
			TCP模块主动限制网络中传输的未被确认的数据量。
			和流量控制非常相似，但不是设置目的主机的窗口大小限制，而是根据已经确认和丢弃的数据报的数量计算限制本身。
			具体的算法依赖于实现，但通常是某种 AIMD additive increase multiplicative decrease 系统
			例如：
			当连接刚刚建立时，TCP模块设置避免拥塞的限制为最大分段大小 MSS 很小的倍数，通常设置为2倍
			然后，每当确认一个报文段，将限制增加一个MSS
			对于一个理想的连接，这意味着每个往返时间RTT 能够确认的数据报数量为所设置的限制值，这导致
			限制的取值变为2倍。
			但是，一旦数据报被丢弃，TCP模块马上将限制值减少一半，怀疑这个丢失是由网络拥塞导致的。
			使用这种方式，最终会达到一个平滑状态，在没有发生数据包丢失的情况下，发送方尽可能快地发送。
			TCP还可以通过发送大小尽可能接近MSS的数据报来降低网络拥塞。
			因为数据报需要40byte的头部，所以发送小的报文段不如合并为一个大的数据块效率高。
			这意味着TCP需要维护一个向外发送的缓冲区来收集上层要发送的数据
			许多TCP的实现使用 纳格算法 Nagle's algorithm来决定什么时候收集数据，什么时候发送报文段，
				它是一些规则的集合。
			习惯上，如果有未被确认的数据传输，就收集数据，直到数据量大于最大分段大小MSS，或拥塞控制窗口，
				取这两个值中的最小值。

		小窍门：
			当游戏使用TCP作为传输层协议时，纳格算法是玩家的克星。
			尽管它减少了宽带的使用，但是明显增加了数据发送的延时。
			如果一个实时数据需要向服务器发送很少量的更新，在有足够的更新累加起来填充MSS之前
			游戏已经运行了许多帧了。
			这会使玩家感到游戏延时，仅仅是因为运行了纳格算法。
			因为这个原因，大部分的TCP实现提供一个选项来禁用这个拥塞控制功能。

	断开连接
		关闭TCP连接需要分别来自两端的终止请求和确认。
		当一台主机没有要发送的数据时，会发送一个FIN数据报，表示准备停止发送数据。
		在缓冲区等待的数据包括FIN数据报仍然会被传输以及在必要时重传，直到被确认。
		但是TCP模块不会接收来自上层的新数据。不过另外一台主机可以接收数据，并且
		确认所有收到的数据。当它没有要发送的数据时，也会发送一个FIN数据报。
		当之前准备关闭连接的主机收到这个FIN数据报和相应它自己FIN的数据报的ACK时，
		或者ACK超时，TCP模块都会完全关闭连接并删除连接状态
```

```
应用层 application layer
	DHCP 动态主机配置协议 dynamic host configuration protocol
		给子网中的每一台主机分配唯一的IPv4地址非常有挑战
		特别是当智能手机和笔记本电脑也接入
		动态主机配置协议 通过允许主机在接入网络时请求自动配置信息来解决这个问题

		在接入网络时，主机创建一个DHCPDISCOVER消息
		包含它自己的MAC地址
		并使用UDP协议以广播的方式发送到255.255.255.255:67
		这个消息会发送给子网中的每一台主机，任何DHCP服务器都会收到这个消息
		如果DHCP服务器有可以提供给客户端的IP地址，就会准备一个DHCP OFFER数据包。
		这个数据包包含可提供的IP地址和这个客户端的MAC地址。
		此刻这个客户端没有IP地址，所以服务器不能直接将数据包发送给它。
		而是服务器通过UDP 68端口把这个数据包广播到整个子网
		所有DHCP客户端都会收到这个数据包，检查消息里面的MAC地址来判断自己是否是期望的接收者
		当正确的客户端收到这个消息，读取所提供的IP地址，并决定是否接受这个分配
		如果接受，回复一个广播的DHCPREQUEST消息请求这个IP地址
		如果这个IP地址仍然可用，服务器再一次回复一个广播的DHCP ACK消息。
		这个消息与客户端确认IP地址已经分配，并传达其他必要的网络信息，
		如子网掩码，路由器地址和推荐可使用的DNS名称服务器

	DNS 域名系统 domain name system
		能够将域名和子域名翻译为IP地址。
		通常通过UDP协议发送，使用端口53
```

```
NAT 网络地址转换 network address translation
	直到现在，我们讨论的所有IP地址都是公开可路由的
	如果一个IP地址被称为 公开可路由 publically routable 的
	互联网上的任意正确配置的路由器都可以给这个IP地址所在的主机发送数据包。
	这需要任何公开可路由地址都是唯一分配给一台主机的，如果多台主机共享一个IP地址
	那么发送给一台主机的数据包有可能会到达另一台主机。
	为了保证公开可路由地址的唯一性，ICANN及其下属公司给大型机构分配独立的IP地址块
	多台设备公用一个专用的公开IP地址，怎么办呢？

	为了配置一个NAT网络，必须给网络中每台主机分配一个本地可路由privately routable
	的IP地址，IANA保留地址，不作为公开IP地址分发。
	IP地址范围						子网
	10.0.0.0~10.255.255.255			10.0.0.0/8
	172.16.0.0~172.31.255.255		172.16.0.0/12
	192.168.0.0~192.168.255.255		192.168.0.0/16

	路由器
	广域网网卡 IP:18.19.20.21----------互联网
	局域网网卡 IP:192.168.1.1

	电子游戏机			智能手机			笔记本电脑
	IP:192.168.1.2		IP:192.168.1.3		IP:192.168.1.4

	假设公开IP地址12.5.3.2运行游戏服务，端口200
	拥有本地IP地址192.168.1.2的电子游戏机运行着一个游戏，绑定端口100
	游戏机需要使用UDP协议向服务器发送一条消息，所以构建一个数据报

	游戏机
	NIC
	192.168.1.2
	Send
					Src:
					192.168.1.2:100
					Dest:
					12.5.3.2:200
	路由器
	LAN NIC			WAN NIC
	192.168.1.1		18.19.20.21
	Receive			Send
										Src:
										192.168.1.2:100
										Dest:
										12.5.3.2:200
	服务器
	NIC
	12.5.3.2
	Receive
	Send
					Src:
					12.5.3.2:200
					Dest:
					192.168.1.2:100	???
	没有NAT的路由器


	游戏机
	NIC
	192.168.1.2
	Send
					Src:
					192.168.1.2:100
					Dest:
					12.5.3.2:200
	路由器
	LAN NIC			WAN NIC
	192.168.1.1		18.19.20.21
	Receive
		Rewrite Src IP
					Send
										Src:
										18.19.20.21:100
										Dest:
										12.5.3.2:200
	服务器
	NIC
	12.5.3.2
	Receive
	Send
					Src:
					12.5.3.2:200
					Dest:
					18.19.20.21:100
	路由器
	LAN NIC			WAN NIC
	192.168.1.1		18.19.20.21
					Receive ???
	有NAT的路由器
	路由器需要一些机制来识别传入数据包的内部接收者。
	一种直观的方式是建立一个表来记录每个发出去的数据包的源IP地址
	但如果多台内部主机向同一台外部主机发送数据时，这种方法就失效了。

	所有现代NAT路由器采用的解决方案都暴力地破坏了网络层和传输层之间的抽象
	通过同时重写IP头部的IP地址 和 传输层头部的端口号
	路由器可以创建更精确的映射和标记系统
	NAT表中记录这些映射关系

	源地址				外部端口		目的地址
	192.168.1.2:100		50000			12.5.3.2:200

	为了更加安全，许多路由器将原始的目的IP地址和端口添加到NAT表的表项中。
	这样，当相应数据包到达路由器，NAT模块首先使用数据包的源端口查询表项
	然后证实相应数据包的源IP地址和端口与原始发送出去的数据包的目的IP地址
	和端口一致。如果不一致，那么发生了可疑的事情，数据包被丢弃不转发

NAT穿越
	对于互联网用户来说，NAT是奇妙福音
	但对于多人游戏开发者来说，却是一件令人头疼的事
	例如：
	玩家A有主机A，隐藏在NAT A后
	他想在主机A上运行一个多人服务器，想让他的朋友B连接到他的服务器
	玩家B有主机B，隐藏在NAT B后
	因为NAT的原因，玩家B没有办法创建于主机A的连接。
	如果主机B给主机A的路由器发送数据包试图连接，在主机A的NAT表中没有这一表项
	所以这个数据包直接被丢弃

	Host A					NAT A
	IP:192.168:10.2:200		WAN NIC IP:18.19.20.21
							LAN NIC IP:192.168.1.1
	Host B					NAT B
	IP:192.168.20:200		WAN NIC IP:12.12.6.5
							LAN NIC IP:192.168.1.1

	有一些解决该问题的方法
	一种方法是玩家A在路由器上手动配置端口转发。
	这需要一些技术和信心，并不适合强迫玩家完成这一操作
	第二种方法更加灵活，称为
	UDP对NAT的简单穿越方式 simple traversal of UDP through NAT,STUN

	当使用UDP时，主机与第三方主机通信，如Xbox Live或者PlayStation网络服务器
	第三方告诉主机如何彼此间创建连接，这样它们的路由器NAT表中就得到了所需要的项
	所以它们将可以进行直接通信

	主机A 告诉 主机N 它想成为服务器
	消息经过 路由A 路由A在NAT表中添加项
	路由A 将数据包转发给 主机N
	主机N 记录 玩家A 路由A地址端口 想要成为服务器

	主机B 告诉 主机N 想要连接玩家A 的游戏
	消息经过 路由B 路由B在NAT表中添加项
	路由B 将数据包转发给 主机N
	主机N 记录 玩家B 路由B地址端口 想要连接主机A

	此时 主机N 知道 路由A 路由B
	可以发送消息给 路由A 路由B
	但是 路由A 只接受主机N
		 路由B 只接受主机N

	幸好 主机N知道 路由器B IP端口号
	将信息发送给主机A
	主机A 使用 从主机N 收到的连接信息 给主机B 发送数据包
	这看似疯狂 因为 主机A是服务器 它试图连接客户端
	但更加疯狂 因为 路由器B 不接受 主机A 不允许通过
			   但是	这样做是为了在 路由A中添加一个表项

	这个额外的表项是关键，这个数据包可能永远无法到达主机B
	但是在这之后 主机N 可以告诉主机B 直接通过路由A端口 连接主机A
	主机B 发送消息 路由A发现确实是期望的来自主机B的数据包 发送给主机A
	从这时起 主机A 和主机B 可以直接通信

	Tips：
	并不是所有NAT都用上述的穿越技术
	有些NAT给内部主机分配不一致的外部接口，这样的NAT被称为对称NAT symmetric NAT
	在对称NAT中，每一个即将发出的请求收到唯一的外部端口，即使发出这个请求的源IP地址
	和端口已经在NAT表中了。
	这破坏了STUN，因为当主机A给主机B发送第一个数据包时，路由器A将使用一个新的外部端口
	当主机B使用之前主机A连接主机N时使用的外部端口连接路由器A时，在NAT表中找不到项，数据包被丢弃

	有时，不安全的对称NAT按序分配外部端口，所以机智的程序可以使用 端口分配预测
	port assignment prediction 方法在对称NAT上实现类似STUN的技术，安全一些的对称NAT使用随机端口分配
	这样不容易被预测

	STUN方法只适用于UDP协议
	TCP有 TCP打洞 TCP hole punching技术，前提时NAT路由器支持这种方式

	最后，还有一个流行的方法允许NAT路由器穿越
	因特网网关设备协议 internet gateway device protocol IGDP
	一些通用即插即用 universal plug and play, UPnP
	的路由器使用这个协议允许局域网主机手动配置外部端口与内部端口的映射关系
	但并不是所有的路由器都支持这种方式，并且人们对其缺乏兴趣
```

---

# 第三章 伯克利套接字

```
伯克利套接字应用程序接口 Berkeley Socket API
提供了进程与TCP/IP模型各个层之间通信的标准方法
网络编程中名副其实的标准

SOCKET socket(int af, int type, int protocol);

af表示协议族
AF_UNSPEC		未指定
AF_INET			IPv4
AF_IPX			网间分组交换
AF_APPLETALK	Appletalk协议，苹果系统早期协议
AF_INET6		IPv6
大部分游戏支持IPv4 同时应该支持IPv6

type指明通过socket发送和接收分组的形式
socket可以使用的每一个传输层协议都有对应的数据包分组和使用的方式
SOCK_STREAM			有序的，可靠的数据流分段
SOCK_DGRAM			离散的报文
SOCK_RAW			数据包头部可以由应用层自定义
SOCK_SEQPACKET		与STREAM类似，但是数据包接收时需要整体读取

protocol取值0 表示告诉操作系统为给定的socket类型选取默认实现协议

// 创建一个IPv4 UDP socket
SOCKET udpSocket = socket(AF_INET, SOCKET_DGRAM, 0);
// 创建一个TCP socket
SOCKET tcpSocket = socket(AF_INET, SOCKET_STREAM, 0);
//关闭socket 不需要考虑类型
int closesocket(SOCKET sock);
当关闭TCP socket时，很重要的一件事是保证所有发送和接收的数据都已经传输和确认
所有最好先停止socket传输，等待所有的数据被确认，所有传入的数据被读取，再关闭socket
在关闭之前，停止传输和接收，使用shutdown函数
int shutdown(SOCKET sock, int how)

对于参数how
SD_SEND 停止发送
SD_RECEIVE 停止接收
SD_BOTH 停止发送和接收
传入SD_SEND将产生FIN数据包，当所有数据发送后再发送这个数据包
	通知连接的另外一端可以安全关闭socket
	然后另一个FIN数据包作为相应 再返回给发送方
	一旦你的游戏收到FIN数据包，说明可以安全关闭socket
	这样关闭socket并返回给操作系统任何与之相关的资源
	当不需要socket时，一定要确保关闭它们

在大部分情况下，操作系统为通过socket发送的每一个数据包创建IP层头部和传输层头部
但是，通过创建 type SOCK_RAW 和 protocol 0 的socket，可以直接写这两层头部的值
这允许你直接设置通常不被编辑的头部阈值
例如：
	你可以轻松设置没有给即将发送的数据包的TTL
	Traceroute工具就是这样做的
手动写入不同头部字段的取值也是在这些字段插入非法取值的唯一方法，这在
模糊测试你的服务器时是非常有用的
因为raw socket允许头部字段的非法取值， 所有存在潜在的安全风险
大部分的操作系统仅仅在有较高安全凭据的程序中才允许创建raw socket
```

```
API 操作系统差异

SOCKET socket(int af, int type, int protocol);
中的SOCKET仅仅在基于Windows的平台上使用
是 UINT_PTR 的 typedef
它指向保存socket状态和数据的内存中的一块区域

在基于POSIX的平台，socket用一个int表示
本质上没有socket数据类型，socket函数返回一个整数
这个整数表示操作系统打开的文件和socket列表的一个索引
用这种方法，socket与POSIX文件描述符十分类似，可以传入很多操作系统函数作为文件描述符
以这种方式使用socket和专用socket函数相比失去了一些灵活性，
但是在许多情况下也提供了简单的图形将非网络程序移植为网络兼容的程序
socket函数返回一个int的最明显不足是缺乏类型安全性
因为编译器允许代码中传入一个整数表达式作为socket参数

不论是 int 还是 SOCKET 都是值传递


不同平台间的第二个差异是包含库声明的头文件
Windows的socket库是Winsock2 和旧版Winsock会造成命名冲突
为了避免这个问题，必须在#include Windows.h之前 #include WinSock2.h
或者在 Windows.h之前#define WIN32_LEAN_AND_MEAN
宏产生预处理将Windows.h中包含的Winsock忽略，避免冲突

WinSock2.h只包含和socket直接相关的功能
如：地址转换功能，需要包含Ws2tcpip.h


在POSIX平台，只有一个版本的socket库
通过包含sys/socket.h来使用
为了使用IPv4特有的功能，必须还要包含netinet/in.h
要使用地址转换功能，需要包含arpa/inet.h
要实现解析，需要包含netdb.h

初始化和关闭socket库，各个平台也不同
POSIX平台，库默认激活，不需要特意启动


WinSock2需要显式启动和关闭，并允许用户指定使用什么版本

使用WSAStartup激活Windows上的socket库
int WSAStartup(WORD wVersionRequested, LPWSADATA lpWSAData);
wVersionRequested是2byte的WORD
低字节表示主版本号，高字节表示Winsock实现的最低版本，
本书出版时，通常为MAKEWORD(2, 2)
lpWSAData指向Windows特定的数据结构
WSAStartup函数填入被激活的socket库的信息包括实现的版本
WSAStartup返回0或者错误码，0表示成功，错误代码表示不能启动的原因
进程必须首先成功执行WSAStartup，才能正确运行Winsock2中的函数

关闭时，调用WSACleanup
int WSACleanup();
没有输入参数，返回错误码。
在关闭Winsock前，最好确保所有socket都已经挂你并且没有在使用
WSAStartup是引用计数的，调用WSACleanup的次数与调用WSAStartup的次数必须一致


错误报告处理各平台略有不同
所有平台大部分函数错误返回-1
Windows上使用宏 SOCKET_ERROR代替-1
WinSock2提供了获取错误代码的WSAGetLastError函数
int WSAGetLastError();
这个函数仅返回当前运行线程的最近错误代码，
所以在socket库返回-1之后马上检查错误原因，这一点很重要。
在发生错误之后继续调用socket函数会因为第一个错误而造成第二个错误
这件改变WSAGetLastError的返回结果，掩盖问题的真正原因

类似的POSIX库也提供获取特定错误信息的方法
但是，它们使用c标准库中的全局变量errno报告错误代码
为了获取errno值，必须包含errno.h文件
之后可以像其他变量一样读取errno
与WSAGetLastError的返回结果一样，每次函数调用之后，errno的值都可以改变
发生错误之后马上检查该值非常重要

Tips:
	socket库中大部分与平台无关的函数仅使用小写字母，例如socket
	但是，大部分Windows下的WinSock2函数以大写字母开头，有时用WSA前缀
	来标记它们是非标准函数
	做Windows开发时，尽量将大写字母的WinSock2函数和跨平台的函数分离，
	这样移植起来更加容易

WinSock2特定函数时POSIX不支持的
后面我们仅仅讲标准的，跨平台的函数
```

```
socket 地址

每个网络层数据包都需要一个源地址和一个目的地址
如果数据包封装传输层数据，还需要一个源端口和一个目的端口
struct sockaddr {
	uint16_t sa_family;
	char sa_data[14];
;}
sa_family是一个常数，指定地址类型
	当在socket中使用这个socket地址时，sa_family字段应该与创建socket时使用的参数af一致
	sa_data是14字节，存储真正的地址，是通用的字节数组
		从技术上，可以手动填写字节，但这需要知道各种地址族的内存布局
		为了弥补，API为常用地址族提供了帮助地址初始化的专用数据类型。
		当socketAPI函数需要传入地址时，必须手动讲这些数据类型转换为sockaddr类型
		使用sockaddr_in类型创建一个IPv4数据包地址
		struct sockaddr_in {
			short sin_family;
			uint16_t sin_port;
			struct in_addr sin_addr;
			char sin_zero[8];
		};

		sin_family和sockaddr中的sa_family重叠，因此具有相同的含义
		sin_port存储地址中的16位端口部分
		sin_addr存储4字节的IPv4地址
			in_addr类型在不同的socket库之间有差异
			在一些平台上，它是简单的4字节整数
				IPv4通常不写成4字节整数，而是由英文句号分割的4个单独字节
			出于这个原因，一些平台提供结构体封装这个结构
			struct in_addr {
				union {
					struct {
						uint8_t s_b1, s_b2, s_b3, s_b4;
					} S_un_b;
					struct {
						uint16_t s_w1, s_w2;
					} S_un_w;
					uint32_t S_addr;
				} S_un;
			};
			这样用 s_b1,s_b2,s_b3,s_b4用人类可读的形式输入地址
		sin_zero不使用，仅仅为了填补sockaddr_in，使得sockaddr_in的大小与sockaddr一致
			为了一致性，全设为0

Tips:
	当实例化BSD socket结构体时，一种好的方法是使用memset讲所有的成员清零
	这有利于阻止来自未初始化字段的跨平台错误，当一个平台使用的字段在另外一个
	平台不使用的时候，会出现这种情况

	当利用4字节整数设置IP地址或者设置端口号时，很重要的一件事情是考虑TCP/IP协议族
	和主机有可能在多字节数的字节序上采用不同的标准
	第四章研究字节序
	uint16_t htons(uint16_t hostshort);
	uint32_t htonl(uint32_t hostlong);

	htons函数输入以主机本地字节序表示的任意无符号16位整数，将其转换位网络字节序表示的整数
	htonl函数针对32位整数执行同样的操作

	有些平台上主机的字节序和网络字节序相同，那么这些函数不做任何操作
	当开启优化程序时，编译器会识别这一情况并忽略这个函数调用，不产生额外代码
	如果主机字节序和网络字节序不同，返回值与输入参数有相同的字节，但是交换了顺序
	意思是：
		如果你使用了这样的平台，你使用调试器检查一个正确初始化的sockaddr_in 的sa_port
		字段，那里十进制表示的值与你预期的端口将不一样，而是你的端口号交换字节后的十进制值
	有时，如接收数据包时，socket库赋值给sockaddr_in结构体
	这时，sockaddr_in字段仍然时网络字节序，所以如果你想要提取并读懂它们
	应该使用ntohs函数和ntohl函数将网络字节序转换为主机字节序

	uint16_t ntohs(uint16_t networkshort);
	uint32_t ntohl(uint32_t networklong);

	例子：初始化sockaddr_in
	sockaddr_in myAddr;
	memset(myAddr.sin_zero, 0, sizeof(myAddr.sin_zero));
	myAddr.sin_family = AF_INET;
	myAddr.sin_port = htons(80);
	myAddr.sin_addr.S_un.S_sun_b.s_b1 = 65;
	myAddr.sin_addr.S_un.S_sun_b.s_b2 = 254;
	myAddr.sin_addr.S_un.S_sun_b.s_b3 = 248;
	myAddr.sin_addr.S_un.S_sun_b.s_b4 = 180;

	Tips:
		一些平台在sockaddr中添加额外的字段来存储结构体的长度。
		这是为了未来允许更长的sockaddr结构体
		在这些平台上，长度设置为sizeof所使用的结构体
		例如：Max OS X
		myAddr.sa_len = sizeof(sockaddr_in)
		来初始化

字符串初始化sockaddr
	inet_pton POSIX
	InetPton Windows
	int inet_pton(int af, const char* src, void* dst);
	int InetPton(int af, const PCTSTR src, void* dst);

	sockaddr_in myAddr;
	myAddr.sin_family = AF_INET;
	myAddr.sin_port = htons( 80 );
	InetPton(AF_INET, "65.254.248.180", &myAddr.sin_addr);

简单的DNS查询
	int getaddrinfo(const char *hostname, const char *servname, const addrinfo *hints,
		addrinfo **res);
	hostname是空字符结尾字符串 如 live-shore-986.herokuapp.com
	servname是空字符结尾字符串 存储端口号或者端口号对应的服务名称 80或者http
	hints是指向addrinfo结构体的指针，存储希望收到的结果
		可以使用这个参数设定一个理想的地址族或者其他要求，或者传入nullptr获取所有匹配结果
	res是一个指针的指针，指向新分配的addrinfo结构体链表的头部
		每个addrinfo表示来自DNS服务器响应的一部分

		hints 提示
		struct addrinfo {
			int			ai_flags;
			int			ai_family;//地址族 AF_INET AF_INET6 等
			int			ai_socktype;
			int			ai_protocol;
			size_t		ai_addrlen;//sockaddr的大小
			char		*ar_canonname;
			sockaddr	*ai_addr;
			addrinfo	*ai_next;//链表中的下一个addrinfo，因为一个域名可以对应多个IPv4和IPv6地址
								//所以应该遍历链表知道找到需要的sockaddr 或者通过ai_family筛选
		}

	因为getaddrinfo分配一个或者多个addrinfo结构体，所以一旦保存了需要的sockaddr，应该调用freeaddrinfo释放内存
	会遍历链表释放所有相关缓存
	void freeaddrinfo(addrinfo* ai);

	为了将主机名解析为IP地址，getaddrinfo创建一个DNS协议包，使用UDP或者TCP协议发送给操作系统配置指定的DNS服务器
	然后等待响应，解析响应，构建addrinfo结构体链表，返回调用者。
	这个过程依赖和远程主机的通信，所以需要大量时间
	getaddrinfo没有内置的异步操作 会阻塞线程
	Windows可以调用特有的GetAddrInfoEx函数，不需要手动创建线程
```

```
绑定socket
	通知操作系统socket将使用一个特定地址和传输层端口的过程称为绑定binding
	手动将一个socket绑定到一个地址和端口时，使用bind函数
	int bind(SOCKET sock, const sockaddr *address, int address_len);

	sock是待绑定的socket，之前通过socket函数创建
	address是socket应该绑定的地址。
		注意，这与socket发送数据包的目的地址无关。
		你可以把它看作是发送出去的数据包的源地址。
		你在指定一个返回地址，这似乎很奇怪，因为任何从这台主机发送的数据包显然是来自这台主机的地址。
		但是，记住一台主机是可以有多个网络接口，
		每一个网络接口有自己的IP地址通过一个特定地址的绑定允许你确定socket应该使用哪个接口。
		当主机作为路由器或网络之间的桥梁时，这将特别有用，因为不同的接口有可能连接完全不同的计算机。
		对于多人游戏，指定网络接口没那么重要，
		事实上通常需要为所有可用的网络接口和主机的所有IP地址绑定端口。
		为此，你可以给需要绑定的socket_in的sin_addr字段赋值为宏INADDR_ANY
	address_len应该存储address的sockaddr结构体的大小
	bind成功时返回0，出现错误时返回-1

	将socket与sockaddr绑定有两个作用
	第一，当传入数据包的目的地址和socket绑定的地址及端口一致时，告诉操作系统这个socket是数据包的目标接收者
	第二，为从socket发出的数据包创建网络层和传输层头部的时候，它指定了socket库使用的源地址和端口。
	通常，只能将一个socket绑定到一个给定的地址和端口。如果这个地址和端口已经被占用，那么bind返回一个错误。
	这种情况下，你可以反复尝试不同的端口，直到找到可用的端口。
	为了自动完成这个操作，可以给需要绑定的端口赋值为0，这将告诉socket库找一个未被使用的端口并绑定。

	socket在用于发送和接收数据之前必须要绑定。因此，如果一个进程试图使用一个未被绑定的socket发送数据，
	网络库将自动为这个socket绑定一个可用的端口。
	因此，手动调用bind函数的唯一原因是，指定绑定的地址和端口。
	当需要创建一台需要在公开地址和端口监听数据包的服务器时，这是必须的。
	但是对于客户端通常是没有必要的。客户端将自动绑定任何可用的端口
		当给服务器发送第一个数据包时，这个数据包包含自动选择的地址和端口
		服务器使用这些信息就可以正确返回数据包
```

```
UDP Socket
	一旦创建好socket，就可以通过UDP socket发送数据
	如果没有绑定，网络模块将在动态端口范围内找一个空闲的端口自动绑定。
	使用sendto函数发送数据：
	int sendto(SOCKET sock, const char *buf, int len, int flags, const sockaddr *to, int tolen);
	sock是数据包应该使用的socket
	buf是指向待发送数据起始地址的指针。它不必是真实的char*类型，可以是能被转换为char*的类型。
		void*更合适，当成void*对待非常有用。
	len是待发送数据的大小。通常包含8字节头部的UDP数据包最大长度是65535字节，因为头部长度仅用16比特。
		但是，数据链路层MTU决定了在不分片情况下可以发送的最大数据包。以太网MTU是1500字节，但这1500
		字节不仅包括游戏的负载数据，还包括多个头部和任何可能的封装数据。
		因为游戏程序员应当尽量避免分片，所以好的方法是避免发送数据大于1300字节的数据包。
	flags 是对控制发送的标志进行按位或运算的结果。大多数游戏代码中，该参数取值为0。
	to 是目标接收者的sockaddr。这个sockaddr的地址族必须与用于创建socket的地址族一致。
		参数to中的地址和端口被复制到IP头部和UDP头部作为目的IP地址和目的端口。
	tolen 是传入参数to的sockaddr的大小。对于IPv4，传入sizeof(sockaddr_in)即可
	如果操作成功，sendto函数返回等待发送的数据长度，否则返回-1.
	请注意，非0的返回值并不代表数据已经成功发送出去了，仅仅表示已经成功进入发送队列。

	使用recvfrom函数从UDP socket接收数据是一件很简单的事情：
	int recvfrom(SOCKET sock, char *buf, int len, int flags, sockaddr *from, int *fromlen);
	sock是查询数据的socket。
		默认情况下，如果没有发送到socket的未读数据，线程将被阻塞，直到有数据报到达。
	buf 是接受的数据包缓冲区。默认情况下，一旦数据包已经通过调用recvfrom函数复制到缓冲区，
		socket库将不再保存它的副本。
	len 是指定参数buf可以存储的最大字节数。为了避免缓冲区溢出错误，recvfrom函数不能向buf复制比这个参数更多的字节。
		到达的数据包中的剩余字节将被丢弃，所以需要确保使用的接收缓冲区能有你期望接收的最大数据包的大小。
	flags 是对控制接收的标志进行按位或运算的结果。大多游戏代码中，该值取值为0.
		一个偶尔使用的有用的标志是MSG_PEEK。它将一个接收的数据报复制到参数buf中，但是不删除输入队列中的数据。
		这样下一次调用recvfrom函数时可以重新读取相同的数据包，这个调用可能会准备更大的缓冲区。
	from 是一个指向sockaddr结构体的指针，recvfrom函数会写入发送者的地址和端口。
		请注意，这个结构体不需要提取使用任何地址信息进行初始化。
		一个常见的错误是，调用者希望通过设置这个参数来要求只接收来自特定地址的数据包，但这是不可能的。
		相反地，数据报按序交付给recvfrom函数，对每一个数据报，变量from都被设置为相应的源地址。
	fromlen是一个指向整数的指针，存储参数from所指向的sockaddr的大小。
		如果recvfrom函数不需要全部空间来复制源地址，它可以减少这个值。
	如果成功执行，recvfrom函数返回复制到buf的字节数。
	如果发送错误，返回-1。
```

```
TCP Socket
	UDP是无状态，无连接，不可靠的，所以每台主机只需要一个单独的socket来发送和接收数据。
	但是TCP是可靠的，需要发送数据之前，在两台主机之间建立连接。
	此外，必须维护和存储状态以重新发送丢失的数据包。
	在伯克利套接字API中，socket本身存储连接状态，意思是主机针对每个保持的TCP连接，都需要一个额外的，单独的socket。


	TCP需要三次握手启动客户端和服务器之间的连接。服务器要接收这三次握手中初始阶段的数据报，必须首先建立一个socket，
	绑定到指定的端口，然后才能监听传入的连接请求。使用socket和bind函数创建和绑定一个socket之后，才可以使用listen函数启动监听。
	int listen(SOCKET sock, int backlog);

	sock是设置为监听模式的socket。
		监听模式的socket每收到TCP握手的第一阶段数据包时，存储这个请求，直到相应的进程调用函数来接受这个连接，并继续握手操作。
	backlog 是队列中允许传入的最大连接数。一旦队列中的传入连接数量达到最大值，任何后续的连续都将被丢弃。
		输入SOMAXCONN表示使用默认的backlog值。
	函数执行成功时返回0，发生错误返回-1。


	接受传入的连接并继续TCP握手过程时，调用accept函数
	SOCKET accept(SOCKET sock, sockaddr* addr, int* addrlen);

	sock是接受传入连接的监听socket。
	addr是指向sockaddr结构体的指针，将被写入请求连接的远程主机的地址。
		与recvfrom函数中的地址类似，这个sockaddr结构体也不需要被初始化，它不能控制接受哪个连接，
		能做的仅仅是存储被接受连接的地址。
	addrlen是指向addr缓冲区大小的指针，以字节为单位。当真正写入地址之后，accept函数将更新这个参数。
	如果accept函数执行成功，将创建并返回一个可以与远程主机通信的新socket。
		这个新socket被绑定到与监听socket相同的端口号上。
		当操作系统受到一个目的端口是该绑定端口的数据包时，它使用源地址和源端口来确定哪个socket应该接收这个数据包：
		TCP要求每台主机针对每个保持的TCP连接都需要单独的socket。
	accept函数返回的新socket与发起连接的远程主机相关。它存储远程主机的地址与端口，跟踪所有发送出去的数据包，
		一旦丢失可以重发。它也是与远程主机通信的唯一socket：
		一个进程不应该试图使用处于监听模式的初始socket给远程主机发送数据。这将会失败，
		因为监听socket没有连接到任何主机，仅仅扮演程序调度者的角色，帮助创建新socket来响应传入的连接请求。
	默认情况下，如果没有待接受的传入连接，accept函数将阻塞调用线程，直到受到一个传入的连接，或者超市。


	监听和接受连接的过程是不对称的。只有被动的服务器需要一个监听socket。希望发起连接的客户端应该创建socket，
	并使用connect函数开始与远程服务器的握手过程。
	int connect(SOCKET sock, const sockaddr *addr, int addrlen);

	sock是待连接的socket。
	addr是指向目的远程主机的地址指针。
	addrlen是addr参数所指向地址的长度。
	函数执行成功时返回0，发生错误返回-1。
	调用connect函数通过给目的主机发送初始SYN数据包来启动TCP握手。
	如果目的主机有绑定到适当端口的监听socket，它将调用accept函数来继续握手的过程。


	通过连接的socket实现发送和接收
	连接TCP socket存储远程主机的地址信息。因此，进程不需要为创数数据的函数传入地址参数，而是使用send函数
	通过连接的TCP socket发送数据：
	int send(SOCKET sock, const char *buf, int len, int flags);

	sock是用于发送数据的socket。
	buf 是写入缓冲区。请注意，与UDP不同，buf不是一个数据包，不需要作为一个单独的数据单元传输。
		而是将数据放到socket的输入缓冲区中，socket库来决定在将来的某一时间发送出去。
		如果使用第二章介绍的纳格算法，需要积累到MSS大小的数据时再发送出去。
	len 是传输的字节数量。
		与UDP不同，不需要保持这个值低于链路层的MTU。只要socket的输出缓冲区有空间，网络库就可以将数据放到缓冲区中，
		然后等到缓冲区数据块大小合适时再发送出去。
	flags是对控制数据发送标志进行按位或运算的结果。大多数游戏代码中，该参数取值为0。
	如果send函数调用成功，返回发送数据的大小。如果socket的输出缓冲区有一些空余的空间，但不足以容纳整个buf时，
		这个值可能会比参数len小。如果没有空间，默认情况下，调用线程将被阻塞，直到调用超时，或者发送了足够的数据后产生空间。
		如果发送错误，send函数只能返回-1。
	请注意，非0的返回值并不代表数据已经成功发送出去了，只能说明数据被存入队列中等待发送。


	调用recv函数从一个连接的TCP socket接收数据
	int recv(SOCKET sock, char *buf, int len, int flags);

	sock是待接收数据的socket。
	buf是数据接收缓冲区。被复制buf中的数据从socket的接收缓冲区中删除。
	len是拷贝数据到buf中数据的最大数量。
	flags是对控制数据接收的标志进行按位或运算的结果。
		可用于recvfrom函数的标志都可以用于recv函数。大多数游戏代码中，该参数取值为0.
	如果recv函数调用成功，返回接收的数据大小。这个值小于等于len。
	根据远程调用的send函数的信息不可能预测接收数据的数量： 远程主机的网络库收集数据，然后发送它认为合适大小的报文段。
	当len非0时，如果recv返回0，说明连接的另外一端发送了一个FIN数据包，承诺没有更多需要发送的数据。
	当len为0时，如果recv返回0，说明socket上有可以读的数据。
		当有许多socket在使用时，这是检查是否有数据到来而不需要占用单独缓冲区的一个简便方法。
	当recv函数已经表明有可用的数据时，你可以保留一个缓冲区，然后再次调用recv函数，输入这个缓冲区和非0的len。
	如果发生错误，recv函数返回-1。
	默认情况下，如果socket的接收缓冲区中没有数据，recv函数阻塞调用线程，直到数据流中的下一组数据到达，或者超时。


	Tips：
	你可以在连接的socket上使用sendto函数和recvfrom函数。 但是，地址参数将被忽略，这很令人困惑。
	同样地，在一些平台上，可用在UDP socket上调用connect函数，在socket的连接数据中存储远程主机的地址和端口。
	这并没有建立一个可靠的连接，但是允许使用send函数给已保存的主机发送数据，而不需要每次都指定地址。
	这也会导致这个socket抛弃来自除这台已保存主机之外的数据报文。
```

```
服务器调用 socket bind listen 监听指定地址
客户端调用 socket connect 发送连接请求
服务器调用 accept 接收请求 建立好连接
```

```
阻塞和非阻塞I/O

从socket接收数据是典型的阻塞操作。如果没有可以接收的数据，线程被阻塞直到有数据到达。
在主线程等待数据包到达很糟糕。当socket还没准备好，发送，接受和发起连接操作同样可以被阻塞。
对于实时应用，例如游戏，需要检查到来数据而不降低频率，这种方式会造成许多问题。

多线程
	解决阻塞I/O的一种方法是给每一个可能的阻塞调用生成一个线程。

	启动主线程
	生成监听线程
	模拟游戏{			启动监听线程
	处理客户端输入		创建socket并绑定
	}
						监听
						接受{
							阻塞直到客户端发送
						为新socket生成线程
						}
												启动每个客户端线程
												接受{
													阻塞直到客户端发送
												给主线程发送数据
												}


	在启动后，监听线程创建一个socket，绑定，调用listen，然后调用accept。
	accept阻塞，直到有一个客户端尝试连接。
	当有一个客户端连接，accept返回一个新socket。
	服务器进程为这个socket生成一个新线程，循环调用recv。
	recv阻塞，直到客户端发送数据。
	当客户端发送数据，recv函数开启，非阻塞线程使用一些回调机制在循环回来再次调用recv之前，
	向主线程发送新的客户端数据。同时，在接受新连接的时候，监听socket保持阻塞，而主线程在模拟游戏运行。

	这种方式有一个缺点是每个客户端需要一个线程，当客户端数量增加时，不能得到很好的扩展。而且很难管理，
	因为所有的客户端数据在并行的线程中进入，这些数据需要以安全的方式输入模拟。最后，如果模拟线程试图从一个
	socket发送数据，此刻接受线程也从这个socket接受数据，那么将阻塞模拟线程。
	这些都不是不能克服的问题，但是有更简单的办法。


非阻塞I/O
	默认情况下，socket操作是阻塞模式，正如之前介绍的。
	但是socket也支持 非阻塞 non-blocking 模式，当非阻塞模式的socket被要求执行一个需要阻塞的操作时，它将立刻返回-1。
	它还设置了系统错误代码 errno 或 WSAGetLastError 分别返回 EAGAIN WASEWOULDBLOCK
	这个代码表示之前的socket行为已经阻塞了，没有发生就终止了。然后调用进程可以做出响应的反应。

	Windows下，使用ioctlsocket函数设置socket为非阻塞模式：
	int ioctlsocket(SOCKET sock, long cmd, u_long *argp);
	sock是socket
	cmd是用于控制的socket参数。在这种情况下，输入FIONBIO。
	argp是这个参数的取值。任意非零值将开启非阻塞模式，0将阻止开启。

	在POSIX兼容的操作系统下，使用fcntl函数：
	int fcntl(int sock, int cmd, ...);
	sock是socket
	cmd是发给socket的命令。在更新的POSIX上，必须首先使用F_GETFL获取当前与socket相关的标志，
	让它们与常数O_NONBLOCk按位或运算之后，使用F_SETFL命令更新socket上的标志。

	当socket处于非阻塞模式，调用任何阻塞函数都是安全的，因为我们直到如果它不能在没有阻塞的情况下完成，它会立刻返回。

	void DoGameLoop()
	{
		UDPSocketPtr mySock = SocketUtil::CreateUDPSocket(INET);
		mySock->SetNonBlockingMode(true);

		while(gIsGameRunning)
		{
			char data[1500];
			SocketAddress socketAddress;

			int bytesReceived = mySock->ReceiveFrom(data, sizeof(data), socketAddress);
			if (bytesReceived > 0)
			{
				ProcessReceivedData(data, bytesReceived, socketAddress);
			}
			DoGameFrame();
		}
	}

	socket设置为非阻塞模式，游戏可以检查每帧内是否有准备好的接受数据。如果有，游戏会先处理第一个挂起的数据报。
	如果没有，游戏立即进行到其余的帧，没有等待。
	如果你想要处理更多的数据报，可以添加循环来读取挂起的数据报，直到已经读取了最大数量，或者没有更多数据报可读。
	限制每帧读取数据报速度很重要，如果不这样做，一个恶意客户端可以发送大量单字节数据报，发送的速度大于服务器处理速度，
	阻碍服务器模拟游戏。


Select
	每帧轮询非阻塞socket是在不阻塞线程的情况下检查传入数据的一种简单直接的方式。
	但是，当需要查询的数量很大时，这种方式效率很低。
	作为替代方案，socket库提供了同时检查多个socket的方式，只要其中有一个socket准备好了就可以执行。
	使用select函数实现这个操作：
	int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, const timeval *timeout);
	nfds POSIX平台，nfds是待检查的编号最大的socket标识符。
			在POSIX平台，每个socket只是一个整数，所以直接将所有socket的最大值传入。
		 Windows平台，socket表示为指针，这个参数不起作用，可以忽略。
	readfds 是指向socket集合的指针，称为fd_set，包含要检查可读性的socket。
		按如下方法处理一个fd_set：当数据包到达readfds集合中的socket，select函数尽快将控制返回给调用线程。
		首先，将所有还没有收到数据的socket从集合中移出，然后当select函数返回时，
			仍然在readfds集合中的socket保证不会被读操作阻塞。
			给readfds传入nullptr来跳过任何socket可读性的检查。
	writefds是指向fd_set的指针，这个fd_set存储待检查可写性的socket。当select函数返回，保留在writefds中的所有socket
		都保证可写，不会引起调用线程的阻塞。给writefds传入nullptr来跳过任何socket可写性检查。
		通常，只有socket的输出缓冲区有太多数据时，socket才会阻塞写操作。
	exceptfds 是指向fd_set的指针，这个fd_set存储待检查错误的socket。
		当select函数返回，保留着exceptfds中的所有socket都已经发生了错误。给exceptfds传入nullptr来跳过任何socket错误的检查。
	timeout 是指向超时之前可以等待最长时间的指针。如果在readfds中的任意一个socket可读，
		writefds中的任意一个socket可写，或者exceptfds中的任意一个socket发生错误之前发生超时，情况所有集合，
		select函数将控制返回给调用线程。给timeout输入nullptr来表明没有超时限制。
	返回值 select函数返回执行之后保留在readfds writefds exceptfds中socket的数量。
		如果发生超时，这个值是0

	要初始化一个空的fd_set，首先在栈上声明，然后使用FD_ZERO宏将其赋值为0：
		fd_set myReadSet;
		FD_ZERO(&myReadSet);
	使用FD_SET宏给集合添加一个socket
		FD_SET(mySocket, &myReadSet);
	使用FD_ISSET宏检查在select函数返回之后，一个socket是否在集合中：
		FD_ISSET(mySocket, &myReadSet);
	select函数不是单一的socket的函数，所有它不适合作为类型安全的socket的方法。
		准确地说，它属于SocketUtil类的一个实用方法。

	// 运行一个TCP服务器循环
	void DoTCPLoop()
	{
		// 创建一个监听socket，并将其添加到待检查可读性的socket列表中。
		// 然后循环，直到应用程序通知它退出。
		TCPSocketPtr listenSocket = SocketUtil::CreateTCPSocket(INET);
		SocketAddress receivingAddress(INADDR_ANY, 48000);
		if (listenSocket->Bind( receivingAddress ) != NO_ERROR)
		{
			return;
		}
		vector<TCPSocketPtr> readBlockSockets;
		readBlockSockets.push_back(listenSocket);

		vector<TCPSocketPtr> readableSockets;

		while (gIsGameRunning)
		{
			// 循环使用select来阻塞，直到数据包到达readBlockSockets中的任意一个socket
			// 当数据包到达，Select保证readableSockets中只包含有传入数据的socket
			if (SocketUtil::Select(&readBlockSockets, &readableSockets,
									nullptr, nullptr, nullptr, nullptr))
				{
					//we got a packet-loop through the set ones...
					// 然后函数循环遍历每个被Select标识为可读的socket。
					for (const TCPSocketPtr& socket : readableSockets)
					{
						// 如果是监听socket，意味着远程主机调用了Connect
						if (socket == listenSocket)
						{
							//it's the listen socket, accept a new connection
							SocketAddress newClientAddress;
							//函数接收这个连接，在readBlockSockets中添加新的socket
							//通过ProcessNewClient通知应用程序。
							auto newSocket = listenSocket->Accept(newClientAddress);
							readBlockSockets.push_back(newSocket);
							ProcessNewClient(newSocket, newClientAddress);
						}
						else
						{
							// 如果不是监听socket，函数调用Receive获得新到达的数据块
							// 然后通过ProcessDataFromClient发送给应用程序
							//it's a regular socket-process the data...
							char segment[GOOD_SEGMENT_SIZE];
							int dataReceived = socket->Receive(segment, GOOD_SEGMENT_SIZE);
							if (dataReceived > 0)
							{
								ProcessDataFromClient(socket, segment, dataReceived);
							}
						}
					}
				}
		}
	}

	Tips:
		还有其他的方法来处理在多个socket上的传入数据，但是它们是特定于平台的，并且不常用。
		在Windows平台，当支持成千上万的并发连接时，I/O完成端口是一个可行的方案。
		关于更多I/O完成端口的知识，参考阅读资料。
```

```
其他socket选项
	int setsockopt(SOCKET sock, int level, int optname, const char *optval, int optlen);

	sock是待配置的socket
	level和optname描述被设置的选项。 level是一个整数，表示被定义选项的级别，optname定义选项。
	optval是指针，指向选项所设置的值。
	optlen是数据的长度。例如，如果指定的选项是整数，optlen应该是4.
	setsocketopt函数成功时返回0，发生错误时返回-1.

	SO_RCVBUF				int				指定这个socket分配给接收数据包的缓冲区
											传入的数据积累在接收缓冲区，直到相应的进程调用recv或recvfrom来接受它。
											TCP带宽受限于接收窗口的大小，而接收窗口不能比socket的接收缓冲区大。
											控制这个值可以很大程度上影响带宽
	SO_REUSEADDR			BOOL/int		表明网络层应该允许这个socket绑定到已经被另外一个socket所绑定的IP地址和端口上。
											这对调式或包嗅探程序是有用的。一些操作系统需要该调用进程具有提升的权限
	SO_RECVTIMEO			DWORD/timeval	指定时间（在Windows平台以毫秒为单位），在这个时间之后，被阻塞的接收调用超时并返回。
	SO_SNDBUF				int				指定这个socket分配给发送数据包的缓冲区。发送的带宽受限于链路层。
											如果进程发送数据的速度比链路层可以承受的速度快，socket将把数据存储在发送缓冲区。
											使用可靠协议的socket也使用发送缓冲区存储发送数据，直到收到接收者确认。
											当发送缓冲区满了，send和sendto函数阻塞，直到出现空余。
	SO_SNDTIMEO				DWORD/timeval	在这个时间之后，被阻塞的发送调用应该超时并返回。
	SO_KEEPALIVE			BOOL/int		仅当socket使用面向连接的协议，如TCP时有效。
											这个选项指定了socket应该给连接的另外一端自动定期发送保持连接的数据包。
											如果这些数据包没有被确认，那么socket产生一个错误状态，
											下一次进程试图使用这个socket发送数据时，将会被通知连接已经丢失。
											这不仅有利于检测丢失的连接，还有利于通过防火墙和NAT保持连接，
											否则可能超时。
	TCP_NODELAY				BOOL/int		指定这个socket是否应该忽略纳格尔算法。设置为true表示减少进程要求发送数据和真实
											发送数据之间的延时。但是，这可能会造成网络的拥塞。
```

---

# 第四章 对象序列化

```
为了实现多人游戏网络实体之间的对象传输，游戏必须给这些对象规定数据格式，这样它们才能通过传输层协议发送。
本章讨论一个鲁棒的序列号系统的必要性和使用方法，探索自引用数据，压缩，代码的易维护性等问题的处理方法，
同时满足一个实时系统的运行时性能要求。

class RoboCat: public GameObject
{
public:
	RoboCat(): mHealth(10), mMeowCount(3) {}

private:
	int32_t mHealth;
	int32_t mMeowCount;
};

无序列号传输简单对象
void NaivelySendRoboCat(int inSocket, const RoboCat* inRoboCat)
{
	send(inSocket,
		reinterpret_cast<const char*>(inRoboCat),
		sizeof(RoboCat), 0);
}

void NaivelyReceiveRoboCat(int inSocket, RoboCat* outRoboCat)
{
	recv(inSocket,
		reinterpret_cast<char*>(outRoboCat),
		sizeof(RoboCat), 0);
}

复杂对象无法这样传输
```

```
Stream

流指的是一种数据结构，封装了一组有序的数据元素，并允许用户对其进行数据读写。
流可以是输入流 output stream 输出流 input stream 或者两者都是。

输出流作为用户数据的输出槽，允许数据流用户顺序插入元素，但不能从中读取数据。
输入流作为数据源，允许用户提取元素，但是不提供插入数据的功能。
当一个流即是输入流也是输出流时，同时提供插入和读取数据元素的方法。

通常，一个流时其他数据结构或计算资源的接口。
例如：
	文件输出流 file output stream
		可以封装一个已经打开准备写的文件，提供顺序存储不同类型的数据到磁盘的简单方法。
	网络流 network stream
		可以封装一个socket，提供send 和recv函数的封装，专门用于与用户相关的特定数据类型。

内存流 memory stream
	封装了内存的缓冲区，通常时动态分配在堆上的缓冲区。
	输出内存流 output memory stream
		有顺序写入缓冲区的方法，同时提供对缓冲区本身进行读取访问的访问器。
		通过调用缓冲区访问器，用户可以立即将所有的数据写入流，并发送给另一个系统。


模板Write方法可以允许所有的数据类型，但是要防止非基本数据类型被序列化
记住非基本数据类型需要特殊的序列化
template<typename T> void Write(T inData)
{
	static_assert(std::is_arithmetic<T>::value || std::is_enum<T>::value,
		"Generic Write only supports primitive data types");
	Write(&inData, sizeof(inData));
}
不管选择哪种方法，建立一个辅助函数来自动选择字节数有助于防止因用户输入不正确的字节数而导致的错误。


使用输入内存流，可以实现更加鲁棒的RoboCat发送函数
void RoboCat::Write(OutputMemoryStream& inStream) const
{
	inStream.Write(mHealth);
	inStream.Write(mMeowCount);
	//no solution for mHomeBase yet
	inStream.Write(mName, 128);
	//no solution for mMiceIndices yet
}

void SendRoboCat(int inSocket, const RoboCat* inRoboCat)
{
	OutputMemoryStream stream;
	inRoboCat->Write(stream);
	send(inSocket, stream.GetBufferPtr(), stream.GetLength(), 0);
}


目的主机接收RoboCat需要一个对应的输入内存流和RoboCat::Read方法
void RoboCat::Read(InputMemoryStream& inStream)
{
	inStream.Read(mHealth);
	inStream.Read(mMeowCount);
	//no solution for mHomeBase yet
	inStream.Read(mName, 128);
	//no solution for mMiceIndices
}

const uint32_t kMaxPacketSize = 1470;

void ReceiveRoboCat(int inSocket, RoboCat* outRoboCat)
{
	char* temporaryBuffer = static_cast<char*>(std::malloc(kMaxPacketSize));
	size_t receivedByteCount = recv(inSocket, temporaryBuffer, kMaxPacketSize, 0);
	if (receivedByteCount > 0)
	{
		//temporaryBuffer所有权会转交给内存流
		InputMemoryStream stream(temporaryBuffer, static_cast<uint32_t>(receivedByteCount));
		outRoboCat->Read(stream);
	}
	else
	{
		std::free(temporaryBuffer);
	}
}

Tips:
	当一个完整游戏采用这种模式时，你不会每次数据包到达时都为流分配内存，因为内存分配很慢。
	而是你有一个预分配的最大尺寸的流，每当数据包到达，你会直接接收到该预分配流的缓冲区，
	然后从流中读完并处理数据，再设置mHead为0，这样当下一个数据包到达时，流已经准备好接收。
	在这种情况下，给InputMemoryStream添加功能来允许它自己管理自己的内存是很有用的。
	一个以最大容量为参数的构造函数可以分配流的mBuffer，然后返回mBuffer的访问器允许缓冲区被直接传递到recv。

流解决了序列化的第一个问题：
	它提供了一种简单的方法来创建缓冲区，使用源对象中的各个字段的值来填充缓冲区，给远程主机发送这个缓冲区，顺序提取数据
	将它们插入到目标对象的合适字段。此外，这个过程没有干扰到目标对象中不应该被改变的任何字段，例如虚函数表指针
```

```
字节存储次序的兼容性
	0x12345678
	little-endian 0x78 0x56 0x34 0x12 英特尔x86 x64 苹果IOS
	big-endian 0x12 0x34 0x56 0x78 Xbox360 PS3 IBM PowerPC

	inline uint16_t ByteSwap2(uint16_t inData)
	{
		reutrn (inData >> 8) | (intData << 8);
	}

	可以处理基本的给定大小的无符号整数，但是不能处理需要字节交换的其他类型
		例如 float, double, signed integers, large enums等
	为了实现这些类型的字节交换，需要一些巧妙的类型别名
	template <typename tFrom, typename tTo>
	class typeAliaser
	{
	public:
		TypeAliaser(tFrom inFromValue):mAsFromType(inFromValue) {}
		tTo& Get() { return mAsToType;}

		union
		{
			tFrom mAsFromType;
			tTo mAsToType;
		};
	};

	模板化
	template <typename T, size_t tSize> class ByteSwapper;

	template <typename T>
	class ByteSwapper<T, 2>
	{
	public:
		T Swap(T inData) const
		{
			uint16_t result = ByteSwap2(TypeAliaser<T, uint16_t>(inData).Get());
			return TypeAliaser<uint16_t, T>(result).Get();
		}
	}

	调用模板化的ByteSwap函数创建一个ByteSwapper的一个实例，根据参数的大小来模板化。
	然后这个实例使用TypeAliaser来调用合适的ByteSwap函数。
	理想情况下，编译器优化中间的调用过程，仅仅留下一些操作在寄存器中交换一些字节的顺序。

	注释：
		平台的字节序与流的字节序不匹配并不意味着所有的数据都需要字节交换。
		例如，单字节字符的字符串不需要字节交换，因为即使该字符串是多个字节，但是每个字符仅有一个字节。
		只有基本的数据类型应该被字节交换，而且交换需要与其大小匹配。

	template<typename T> void Write(T inData)
	{
		static_assert(std::is_arithmetic<T>::value || std::is_enum<T>::value,
			"Generic Write only supports primitive data types");

		if (STREAM_ENDIANNESS == PLATFORM_ENDIANNESS)
		{
			Write(&inData, sizeof(inData));
		}
		else
		{
			T swappedData = ByteSwap(inData);
			Write(&swappedData, sizeof(swappedData));
		}
	}
```

```
比特流
	内存流的限制是，它们只能读写整数字节的数据。
	当写网络代码时，通常希望用尽可能少的比特来表示数值，这就需要以比特的精度来读写。
	为此实现内存比特流 memory bit stream 非常有帮助。

	OutputMemoryBitStream mbs;

	mbs.WriteBits(13, 5);
	mbs.WriteBits(52, 6);

	13: 值 0 0 0 0 1 1 0 1
		位 7 6 5 4 3 2 1 0

	52: 值 0 0 1 1 0 1 0 0
		位 7 6 5 4 3 2 1 0

	当代码运行完，mbs.mBuffer所指向的内存包含这两个值
				>					     <  52
				  <  13   >
	值		1 0 0 0 1 1 0 1    0 0 0 0 0 1 1 0
	位		7 6 5 4 3 2 1 0    7 6 5 4 3 2 1 0
	字节		   0				  1
```

```
引用数据
	std::vector<int32_t> mMiceIndices;

	有时，网络代码必须序列化这样的成员变量，即它们引用的数据不与其他对象共享。
	RoboCat类中的mMiceIndices就是一个很好的例子。
	它是一个整数vector，跟踪RoboCat感兴趣的各种老鼠的索引。
	因为std::vector<int>是一个黑匣子，所以使用标准的OutputMemoryStream::Write函数
	从std::vector<int>的地址复制到流中式不安全的。
	这样做的结果是序列化std::vector中所有指针的值，当远程主机反序列化时将指向错误的地方。

	一个自定义的序列化函数应该只写vector所包含的数据，而不是序列化vector本身。
	RAM中的数据可能实际上与RoboCat本身的数据相差很远。
	然而，当自定义函数序列化它时，它将数据嵌入到RoboCat中一起写入流。
	因此，这个过程被称为内联inlining 或嵌入 embedding


	GameObject* mHomeBase;

	有时，序列化的数据需要被一个以上的指针引用，例如mHomeBase
	如果两个RoboCat共享一个大本营，那么使用目前的工具无法表示。
	嵌入方法在序列化时仅仅将同一个大本营的副本嵌入到RoboCat中。
	在反序列化时，将生成两个不同的大本营！
	其他时候，数据以这种方式构成，不能用嵌入方法，例如HomeBase类：
	class HomeBase: public GameObject
	{
		std::vector<RoboCat*> mRoboCats;
	}
	这样RoboCat引用大本营，大本营引用RoboCat，将无限递归。

	解决方案是给每个多处引用的对象一个唯一标识符，然后通过序列化标识符
	实现这些对象引用的序列化。
	传唯一id。

	注释：
		当完全实现后，链接系统和使用它的游戏代码必须容忍收到这样的网络ID，即没有与之映射的对象。
		因为数据包可以丢失，游戏可能收到一个对象，它的成员变量引用了一个尚未发送的对象。
		有许多不同的方法来解决这个问题：
		游戏可以忽略整个对象，或者反序列化这个对象并链接任何可用的引用，将丢失的引用置为空。
		更复杂的系统可用跟踪空链接的成员变量，这样收到给定网络ID的对象时可以链接它。
```

```
压缩
	使用可以序列化所有数据类型的工具，就可以编写通过网络来回发送游戏对象的代码。
	然而，在网络本身所施加的带宽限制内，它不一定是高效的代码。
	必须关心如何尽可能高效地使用带宽。

	一个大型游戏世界可以有上百个运动物体，即使是最高速的带宽连接，给数以百计的连接玩家发送这些对象
	的所有实时数据也足以将之耗尽。


	稀疏数组压缩
	去除任何不需要通过网络发送的信息。
	一个寻找此类信息的好地方是任何稀疏的和不被完全填充的数据结构，
	比如 RoboCat类的mName字段，是char[128]，如果明智地设计，应该只序列化所需要的数据，而不是128个完整字节。


	熵编码 entropy encoding
	这是信息论的一个主题，它利用数据的不确定性进行数据压缩。
	根据信息论，含有期望数据的数据包比含有非期望数据的数据包蕴含更少的信息或者熵。
	因此，代码在发送期望数据时应当比发送非期望数据需要更少的比特数。
	在大多数情况下，花费CPU周期模拟实际游戏比计算数据包中熵的确切取值达到最佳压缩率更重要。
	但是，有一种非常高效的简单形式的熵编码。当序列化某一种成员变量时更有用，
	这种变量的某一特定取值比其他取值的频率都高。
	例如，
	RoboCat类的mPosition字段，它是Vector3类型，有XYZ，Y是高度
	if (inVector.mY == 0)
	{
		Write(true);
	}
	else
	{
		Write(false);
		Write(inVector.mY);
	}

	两种常用值
	if (inVector.mY == 0)
	{
		Write(true);
		Write(true);
	}
	else if (inVector.mY == 100)
	{
		Write(true);
		Write(false);
	}
	else
	{
		Write(false);
		Write(inVector.mY);
	}
	这时，就要考虑是否值得增加第二个特殊值了。
	假设分析表明，猫在天花板上的时间占7%
	PonGround * BitsonGround + PinAir * BitsinAir + PonCeiling * BitsonCeiling
	= 0.9 * 2 + 0.07 * 2 + 0.03 * 33 = 2.93
	平均比特数是2.93，比第一次优化少了1.3比特，因此优化值得。

	这是最简单的熵编码，复杂的常用的 赫夫曼编码 算术编码 gamma编码 行程编码


	定点
	32位浮点数的快速计算是当前计算时代的基准。
	游戏模拟执行浮点运算，但并不意味着通过网络发送时需要所有的32位来表示这些数字。

	例如，游戏世界是4000x4000，游戏世界原点为中心
	坐标的取值为[-2000, 2000]，精确度为0.1个游戏单位
	(Maxvalue - MinValue) / Precision + 1 = (2000 - -2000) / 0.1 + 1 = 40001
	这意味着序列化该分量时有40001个可能的取值。如果有一个从小于40001的整数到相应可能的浮点数取值的映射，
	那么该方法就可以直接通过序列化合适的整数来实现X和Z分量的序列化。

	一种称为定点 fixed point数的方法可以将这个任务变得相当简单。
	只需要存储小于40001的整数。因为log40001是15.3，所以只需要16位来序列化X和Z分量。

	注释：
		一些CPU，例如Xbox360和PS3的PowerPC，实现浮点数和整数之间的转换计算非常昂贵。
		但是，与所节省的带宽相比，这往往值得。
		正如大多数的优化，是一个权衡，由所开发的游戏的特性决定。


	几何压缩
	定点压缩利用游戏特定的信息来实现使用尽可能少的比特进行数据序列化。
	这里再次用到信息论：因为变量的可能取值有了约束，所以需要较少的比特就可以表示那个信息。
	序列化任何数据结构时，只要它的内容有约束，就可以使用这种技术。
	许多集合数据类型就属于这种情况。比如 四元数 和 变换矩阵。

	四元数 quaternion是一种数据结构，包括四个浮点数，用于表示三维空间中的旋转。
	当表示一个旋转时，四元数是归一化的，每个分量都在-1和1之间，所有分量的平方和是1.
	因为所有分量的平方和是固定的，所以序列化四元数只需要序列化四个分量中的三个，
	同时使用一个比特表示第四个分量的符号。
	反序列化代码可以通过1减去其他分量的平方来得到最后一个分量。
	取值在-1到1，16位精度通常就足够了。

	仿射变化矩阵 如果矩阵是未缩放的，通过一个bit标识
	如果缩放均匀，用另外一个bit标识，仅序列化一个分量，而不是所有三个分量。
```

```
可维护性
仅仅关注带宽效率可能会导致丑陋的代码。需要权衡，牺牲一点效率换取代码可维护性。

抽象序列化方向
	每个数据结构或压缩技术都同时需要读方法和写方法。
	这不仅意味着为每个新的功能都实现两个方法，而且这些方法必须彼此保持同步。
	如果改变了一个成员变量的写法，那么必须改变它的读法。
	每个数据结构都有两个这样的耦合的方法令人头疼。
	如果每个结构体只有一个能同时处理读写的方法，代码会更清晰。
	幸运的是，可以通过使用继承和虚函数来实现。
	class MemoryStream
	{
		virtual void Serialize(void* ioData, uint32_t inByteCount) = 0;
		virtual bool IsInput() const = 0;
	};

	class InputMemoryStream: public MemoryStream
	{
		virtual void Serialize(void* ioData, uint32_t inByteCount)
		{
			Read(ioData, inByteCount);
		}

		virtual bool IsInput() const {return true;}
	};

	class InputMemoryStream: public MemoryStream
	{
		virtual void Serialize(void* ioData, uint32_t inByteCount)
		{
			Write(ioData, inByteCount);
		}

		virtual bool isInput() const {return false;}
	}

	template<typename T> void Serialize(T& ioData)
	{
		static_assert(std::is_arithmetic<T>::value || std::is_enum<T>::value,
			"Generic Serialize only supports primitive data types");

		if (STREAM_ENDIANNESS == PLATFORM_ENDIANNESS)
		{
			Serizlize(&ioData, sizeof(ioData));
		}
		else
		{
			if (IsInput())
			{
				T data;
				Serialize(&data, sizeof(T);
				ioData = ByteSwap(data);
			}
			else
			{
				T swappedData = ByteSwap(ioData);
				Serialize(&swappedData, sizeof(swappedData));
			}
		}
	}

	警告：
		这种方式比之前的方法低效，因为需要进行虚函数调用。
		该系统还可以使用模板代替虚函数，这样可以提高一些性能。


数据取动的序列化
	大部分对象序列化代码采用相同的模式：
		对于对象类中的每一个成员变量，序列化该成员变量的值。
		这里可能有一些优化，但是代码的一般结构都是相同的。
	事实上，它们很相似，如果游戏中有一些关于运行时对象成员取值的数据，
		它可以使用单一的序列化方法来处理大部分序列化需求。
	一些语言，例如C#和Java，有内置的反射系统，允许运行时访问类结构。
	然而，C++在运行时反射类成员需要一个自定义构建的系统。
	幸运的是，构建一个基本的反射系统并不复杂。

	// 表示成员变量的基本类型，该系统只支持整形，浮点型，字符串类型
	enum EPrimitiveType
	{
		EPT_Int,
		EPT_String,
		EPT_Float,
	};

	// 表示复合数据类型中的一个单独的成员变量。
	class MemberVariable
	{
	public:
		MemberVariable(const char* inName, EPrimitiveType inPrimitiveType, uint32_t inOffset):
			mName(inName), mPrimitiveType(inPrimitiveType), mOffset(inOffset) {}

		EPrimitiveType GetPrimitiveType() const {return mPrimitiveType;}
		uint32_t GetOffset() const {return mOffset;}
	private:
		// 成员变量的名字，用于调试
		std::string mName;
		// 基本类型
		EPrimitiveType mPrimitiveType;
		// 在父数据类型中的内存偏移，存储偏移非常关键，序列化代码可以为给定对象的基址添加偏移，
		// 用于寻找那个对象的成员变量值在内存中的位置。
		uint32_t mOffset;
	};

	// DataType类包含一个特定类的所有成员变量
	// 对于每一个支持数据取动的序列化方法的类，都有一个相应的DataType实例。
	class DataType
	{
	public:
		DataType(std::initializer_list<const Membervariable&> inMVs) :
			mMemberVariables(inMVs) {}

		const std::vector<MemberVariable>& GetMemberVariables() const
		{
			return mMemberVariables;
		}
	private:
		std::vector<MemberVariable> mMemberVariables;
	};


	// 有了反射的基本实现，下面的代码为一个类加载反射数据：

	// 这个宏为每个成员变量计算合适的偏移量
	// 内置在C++中的offsetof宏对于非POD类有未定义的行为
	// 所以，实际上当offsetof用于包含虚函数或其他非POD类型时，一些编译器会报错
	// 只要类中没有自定义一元运算符&，类的层次结构中没有使用虚继承和引用型成员变量
	// 该自定义宏就能用
	#define OffsetOf(c, mv) ((size_t) & (static_cast<c*>(nullptr)->mv))

	class MouseStatus
	{
	public:
		std::string mName;
		int mLegCount, mHeadCount;
		float mHealth;

		static DataType* sDataType;

		// 必须在某个时刻被调用，以初始化mMemberVariables实体
		static void InitDataType()
		{
			sDataType = new DataType(
			{
				MemberVariable("mName", EPT_String, OffsetOf(MouseStatus, mName)),
				MemberVariable("mLegCount", EPT_Int, OffsetOf(MouseStatus, mLegCount)),
				MemberVariable("mHeadCount", EPT_Int, OffsetOf(MouseStatus, mHeadCount)),
				MemberVariable("mHealth", EPT_Float, OffsetOf(MouseStatus, mHealth)),
			});
		}
	}

	// 理想情况下，不必在反射数据中手写代码，最好有工具能够分析C++头文件并自动为类生成反射数据。
	void Serialize(MemoryStream* inMemoryStream, const DataType* inDataType, uint8_t* inData)
	{
		for(auto& mv: inDataType->GetMemberVariables())
		{
			void* mvData = inData + mv.GetOffset();
			switch(mv.GetPrimitiveType())
			{
				EPT_Int:
					inMemoryStream->Serialize(*(int*))mvData);
					break;
				EPT_String:
					inMemoryStream->Serialize(*(std::string*)mvData);
					break;
				EPT_Float:
					inMemoryStream->Serialize(*(float)mvData);
					break;
			}
		}
	}
	// 每个成员变量的GetOffset方法计算指向这个成员变量实体数据的指针。
	// 接着 GetPrimitiveType的switch语句将数据分到合适的类型中，让类型化的Serialize函数实现真正的序列化。

	// 总的来说，该方法以性能换可维护性，有更多的分支可能导致流水线清空，但是代码更少了，因此错误减少了。
	// 除了网络序列化，反射系统作为一个额外的福利，还可以用于许多地方，例如实现磁盘的序列化，垃圾收集，GUI对象编辑器等等。
```

---

# 第五章 对象复制

```
世界状态 world state
通过在每台主机上构建一个世界状态，并交换任何所需要的信息来保持主机之间状态的一致性，多人游戏提供了这种共享体验。

复制对象 replication
从一台主机向另一台主机传输对象状态的行为称为复制。复制比序列化要求更苛刻。
为了成功复制一个对象，主机必须在序列化对象的内部状态之前实现三步预处理。
1 标记数据包为包含对象状态的数据包
2 唯一标识复制对象
3 指明被复制对象的类型

首先发送方将数据包标记为包含对象状态的数据包。
主机之间的通信可能不仅仅是为了对象复制，所以假设每个传入的数据包都包含对象赋值数据是不安全的。
因此，创建一个枚举类型PacketType来标识每个数据包的类型是很有帮助的。

enum PacketType
{
	PT_Hello,
	PT_ReplicationData,
	PT_Disconnect,
	PT_MAX
};

对于每个要发送的数据包，主机首先序列化相应的PacketType到数据包的MemoryStream
这样，接收方可以从每个到达的数据包中立即读取数据包类型，然后决定如何处理。
习惯上，主机之间交换的第一个数据包被标记为"hello"数据包，用于建立连接，分配状态，
并有可能开始一个验证过程。
PT_Hello作为传入数据包的第一个字节表示了数据包的这种类型。
同样地，PT_Disconnect作为第一个字节表明请求断开处理。
PT_MAX用在后续需要知道数据包枚举类型中元素最大数量的代码中。
为了复制对象，发送方序列化PT_ReplicationData作为数据包的第一个字节。

接下来，发送方需要给接收方标识序列化的对象，这样接收方可以确定它是否已经有这个传入对象的副本。
如果有，可以使用序列化的状态更新这个对象，而不需要实例化一个新的对象。

class LinkingContext
{
public:
	LinkingContext():
	mNextNetworkId(1)
	{}

	uint32_t GetNetworkId(const GameObject* inGameObject, bool inShouldCreateIfNotFound)
	{
		auto it = mGameObjectToNetworkIdMap.find(inGameObject);
		if (it != mGameObjectToNetworkIdMap.end())
		{
			return it->second;
		}
		else if(inShouldCreateIfNotFound)
		{
			uint32_t newNetworkId = mNextNetworkId++;
			AddGameObject(inGameObject, newNetworkId);
			return newNetworkid;
		}
		else
		{
			return 0;
		}
	}

	void AddGameObject(GameObject* inGameObject, uint32_t inNetworkId)
	{
		mNetworkIdToGameObjectMap[inNetworkId] = inGameObject;
		mGameObjectNetworkIdMap[inGameObject] = inNetworkId;
	}

	void RemoveGameObject(GameObject *inGameObject)
	{
		uint32_t networkId = mGameObjectToNetworkIdMap[inGameObject];
		mGameObjectToNetworkIdMap.erase(inGameObject);
		mNetworkIdToGameObjectMap.erase(networkId);
	}

	//unchanged...
	GameObject* GetGameObject(uint32_t inNetworkId);
private:
	std::unordered_map<uint32_t, GameObject*> mNetworkIdToGameObjectMap;
	std::unordered_map<const GameObject*, uint32_t> mGameObjectToNetworkIdMap;
	uint32_t mNextNetworkId;
}

当主机准备好将inGameObject的标识符写入对象状态数据包，
它调用mLinkingContext->getNextworkId(inGameObject, true)，
告诉链接上下文linking context有必要的话生成一个网络标识符。
然后将这个标识符写入到数据包中PackageType的后面。
当远程主机收到这个数据包，它读取标识符，并使用自己的链接上下文来查找引用对象。

如果接收方找到一个对象，直接用反序列化数据更新对象。
如果没有找到对象，则需要创建它。
为远程主机创建对象，需要带创建对象的类信息。
发送方通过在对象标识符后面序列化某种类标识符来提供该信息。
实现它的一种粗暴方法是使用动态类型转换从集合中选择一个硬编码的类标识符，
然后接收方使用switch语句根据类标识符实例化正确的类。

void WriteClassType(OutputMemoryBitStream& inStream, const GameObject* inGameObject)
{
	if(dynamic_cast<const RoboCat*>(inGameObject))
	// 注意，必须用' 不能用"
		inStream.Write(static_cast<uint32_t>('RBCT'));
	else if (dynamic_cast<const RoboMouse*>(inGameObject))
		inStream.Write(static_cast<uint32_t>('RBMS'));
	else if (dynamic_cast<const RoboCheese*>(inGameObject))
		inStream.Write(static_cast<uint32_t>('RBCH'));
}

GameObject* CreateGameObjectFromStream(InputMemoryBitStream& inStream)
{
	uint32_t classIdentifier;
	inStream.Read(classIdentifier);
	switch(classIdentifier)
	{
		case 'RBCT':
			return new RoboCat();
			break;
		case 'RBMS':
			return new RoboMouse();
			break;
		case 'RBCH':
			return new RoboCheese();
			break;
	}
	return nullptr;
}

该方法虽然可行，但是还有许多不足之处。
首先，使用dynamic_cast通常需要C++的内置RTTI是启动状态。
RTTI在游戏中往往是被禁用的，因为对于每一个多态类型他都需要额外的存储空间。
更重要的是，该方法的不足是因为它使得游戏对象系统和复制系统相互依赖。
每次添加一个新的可能被复制的游戏类，都必须在网络代码中同时编辑
WriteClassType和CreateGameObjectFromStream函数
这是很容易被忘记的，从而导致代码不同步。
另外，如果想要在新游戏中重新使用你的复制系统，需要完全重写这些函数，
因为这些函数里面引用了旧游戏中的游戏类代码。
最后，这种耦合使得单元测试变得更加困难，因为在没有加载游戏单元时不能加载网络单元。
通常情况下，最好是游戏代码依赖网络代码，但是网络代码应该几乎不依赖游戏代码。
减少游戏代码和网络代码之间的耦合的一种清晰的方式是使用对象创建注册表将对象识别和创建例程从复制系统中抽象出来。
```

```
对象创建注册表 object creation registry
一个对象创建注册表 是将一个类标识符映射到一个函数，该函数创建一个特定类的对象。
使用该注册表，网络模块可以通过id查找创建函数，并且执行它来创建想要的对象。
每个可复制类都必须为对象创建注册表做准备。
首先，给每个类赋予一个唯一的标识符，并将其存储在名为kClassId的静态常数中。
每个类可以使用GUID来保证标识符之间不重复，当然考虑到需要复制的类不多，128位标识符没必要。
另外一个好的选择是使用基于类名的四字符文本，然后在类提交给注册表的时候检查名字是否冲突。
最后一个是选择在编译时使用构建工具创建类标识符，构建工具自动生成代码来保证唯一性。

警告：
	四字符文本时依赖于实现的。
	使用四字符文本'DXT5' 'GOBJ' 来指定32位值时实现有良好区分性标识符的一种简单方法。
	它们非常好，因为当它们出现在你的数据包内存转储时，它们依然保持易见性。
	因此，许多第三方引擎，从Unreal到C4，都使用它作为标记和标识符。
	不幸的是，在C++标准中，它们被归类为依赖于实现的，意思是并不是所有的编译器都使用
	同样的方式实现字符到整数的转换。
	大部分编译器，包括GCC和Visual Studio，使用相同的转换方式，但是如果你在使用不同编译器编译的进程之间
	进行多字符文本通信时，首先运行一些测试示例来保证两个编译器以同样的方式转换文本。

每个类都有唯一标识符后，为GameObject添加一个虚函数GetClassId
为每个GameObject的每个字类重写该函数，以便返回类标识符。
最后，为每个字类添加一个静态函数，用于创建和返回一个类的实例。

class GameObject
{
public:
	enum{kClass = 'GOBJ'};
	virtual uint32_t GetClassId() const {return kClassId;}
	static GameObject* CreateInstance() {return new GameObject();}
};

class RoboCat: public GameObject
{
public:
	enum{kClassId = 'RBCT'};
	virual uint32_t GetClassId() const {return kClassId;}
	static GameObject* CreateInstance() {return new RoboCat();}
};

注意每个字类都需要实现GetClassId虚函数。虽然代码看起来一模一样，但返回值不同，因为常量kClassId不同。
因为每个类的代码都是相同的，一些开发者喜欢使用预处理宏来生成它。
复杂的预处理宏通常时不推荐的，因为限制的调试器不能处理的很好，但是可以减少来回复制和粘贴代码所造成错误的机会。
此外，如果需要修改复制的代码，只改变宏就可以将这个修改传播到所有类。

#define CLASS_IDENTIFICATION(inCode, inClass)\
enum{kClassId = inCode};\
virtual uint32_t GetClassId() const {return kClassId;}\
static GameObject* CreateInstance() {return new inClass();}

class GameObject
{
public:
	CLASS_IDENTIFICATION('GOBJ', GameObject)
};

class RoboCat: public GameObject
{
	CLASS_IDENTIFICATION('RBCT', RoboCat)
};

宏定义中，每行结尾的反斜杠告诉编译器该定义延伸到下一行。
在类识别系统的合适位置，创建类ObjectCreationRegistry来保存类标识符到创建函数的映射。
游戏代码完全独立于复制系统，并可以用可复制类来填充它。
ObjectCreationRegistry从技术上来说并不一定需要是单例，它仅仅需要保证游戏代码和网络代码能够访问它。

// 函数指针，对应每个类中CreateInstance静态成员函数。
typedef GameObject* (*GameObjectCreationFunc)();

class ObjectCreationRegistry
{
public:
	static ObjectCreationRegistry& Get()
	{
		static ObjectCreationRegistry sInstance;
		return sInstance;
	}

	// 模板，用于防止类标识符与创建函数之间的不匹配。
	template<class T>
	void RegisterCreationFunction()
	{
		assert(mNameToGameObjectCreationFunctionMap.find(T::kClassId) ==
			mNameToGameObjectCreationFunctionMap.end());
		mNameToGameObjectCreationFunctionMap[T::kClassId] =
			T::CreateInstance;
	}

	GameObject* CreateGameObject(uint32_t inClassId)
	{
		GameObjectCreationFunc creationFunc =
			mNameToGameObjectCreationFunctionMap[inClassId];
		GameObject* gameObject = creationFunc();
		return gameObject;
	}

private:
	ObjectCreationRegistry() {}
	unordered_map<uint32_t, GameObjectCreateFunc>
		mNameToGameObjectCreationFunctionMap;
};

// 在游戏启动代码的某个位置，创建注册表
void RegisterObjectCreation()
{
	ObjectCreationRegistry::Get().RegisterCreationFunction<GameObject>();
	ObjectCreationRegistry::Get().RegisterCreationFunction<RoboCat>();
}

有了这个系统，当发送方需要为GameObject写类标识符时，仅仅调用它的GetClassId方法即可。
当接收方需要创建一个给定类的实例时，直接调用对象创建注册表的Create，并传入类标识符。

实际上，这个系统代表了C++的RTTI系统的一个定制版本。因为它是为此目的而手动创建的，
所以在内存使用，类型标识符大小和交叉编译器的兼容方面比只使用C++的typeid操作符有更多的控制权。

小窍门：
	如果你的游戏使用了反射系统，你可以在那个系统上添加，不需要这里的方法。
	仅仅在每个GameObject中添加一个GetDataType虚函数，返回对象的DataType，而不是类标识符
	然后为每个DataType添加唯一的标识符和实例化函数。
	与从类标识符到创建函数的映射不同，对象创建注册表变成了数据类型注册表，实现了从数据类型标识符
	到DataType的映射。
	要复制一个对象，先通过GetDataType方法获得其DataType，再序列化DataType的标识符。
	要实例化一个对象，先在注册表中通过标识符查找DataType，然后使用DataType的实例化函数。


一个数据包中的多个对象
	发送大小与MTU尽可能接近的数据包是非常高效的。并不是所有的对象都很大，所以在一个数据包中发送多个对象是可以提升效率的。
	为了做到这一点，一旦主机已经标记一个数据包为PT_ReplicationData数据包，仅需要为每个对象重复执行以下步骤。
	1 写对象的网络标识符
	2 写对象的类标识符
	3 写对象的序列化数据
	当接收端完成反序列化一个对象，数据包中剩下的任何未使用的数据必然是另一个对象的。
	所以，主机重复这个接收过程，知道没有剩余未使用的数据。
```

```
朴素的世界状态复制方法
	如果世界足够小，整个世界状态可以完全装入一个数据包中。

class ReplicationManager
{
public:
	void ReplicateWorldState(OutputMemoryBitStream& inStream, const vector<GameObject*>& inAllObjects);
private:
	void ReplicateIntoStream(OutputMemoryBitStream& inStream, GameObject* inGameObject);
	LinkContext* mLinkingContext;
};

//利用链接上下文写每个对象的网络ID，利用虚方法GetClassId写对象的类标识符。
//接着根据游戏对象的虚函数Write序列化真实的数据
void Replicationmanager::ReplicateIntoStream(OutputMemoryBitStream& inStream, GameObject* inGameObject)
{
	//write game object id
	inStream.Write(mLinkingContext->GetNetworkId(inGameObject, true));
	//write game object class
	inStream.Write(inGameObject->GetClassId());
	//write game object data
	inGameObject.Write(inStream);
}

// 公有函数，调用者可以用来将一组对象复制数据写入输入流中。
// 首先标记数据为复制数据，然后使用私有方法ReplicateIntoStream分别写每个对象
void ReplicationManager::ReplicateWorldState(OutputMemoryBitStream& inStream, const vector<GameObject*>& inAllObjects)
{
	//tag as replication data
	inStream.WriteBits(PT_ReplicationData, GetRequiredBits<PT_MAX>::Value);
	//write each object
	for (GameObject* go: inAllObjects)
	{
		ReplicateIntoStream(inStream, go);
	}
}

序列化值时使用必要的比特
	记住比特流允许使用任意数量的比特来序列化一个字段的值。
	当序列化枚举类型时，编译器可以真实地计算在编译阶段所需要的最佳比特数，排除从枚举类型中添加或删除元素时产生错误的可能。
	一个小技巧时保证让枚举的最后一个元素以_MAX结尾。这样，增删元素时，_MAX元素总是自动增加或减少，更容易追踪枚举类型最大值。
	ReplicateWorldState方法将最后的枚举值作为模板参数传给GetRequiredBits，计算标识数据包类型最大值所需要的比特数。
	为了处理更高效，在编译阶段，使用称为 模板元编程 template metaprogramming的方法，这是C++工程中的黑色艺术。
	C++模板语言很复杂，以至于它是图灵完备的。只要在编译时输入已知，编译器就可以计算任意函数。
	例如，计算表示数据包类型最大值所需比特数的代码可以写成：

	template<int tValue, int tBits>
	struct GetRequiredBitsHelper
	{
		enum {Value = GetRequiredBitsHelper<(tValue >> 1), tBits + 1>::Value};
	};

	template<int tBits>
	struct GetRequiredBitsHelper<0, tBits>
	{
		enum {Value = tBits};
	};

	template<int tValue>
	struct GetRequiredBits
	{
		enum {Value = GetRequiredBitsHelper<tValue, 0>::Value};
	};

	模板元编程没有显示的循环功能，所以必须使用递归来代替迭代。
	这样 GetRequiredBits依赖递归的GetRequiredBitsHelper来得到参数值的最高比特位，接着计算用于表示所必须的比特数。
	这通过每次将tValue右移一位时便将tBits加1来实现。
	当tValue最终为0时，模板特例化被调用，直接返回存储在tBits中的累计值。

	随着C++11的出现，关键字constexpr允许以较小的复杂性实现一些模板元编程的功能。
	本书出现的较早，编译器尚未支持完整，因此使用模板。


	当接收端检测到状态复制数据包，将它传给复制管理器，循环访问数据包中每个序列化的游戏对象。
	如果游戏对象不存在，客户端创建这个对象并反序列化状态。
	如果游戏对象存在，客户端找到这个对象并反序列化到对象中。
	当客户端处理完这个数据包，销毁所有未出现在数据包中的本地数据对象，因为数据包中未出现表明这个
	游戏对象不再存在于发送端的世界中。

	代码 略
```

```
世界状态中的变化
	因为每台主机都会保存自己的世界状态副本，所以没必要在一个数据包中复制整个世界状态。
	实际上，发送方创建表示世界状态变化的数据包，然后接收方在自己的世界状态中更新这些变化。
	这样，发送方可以使用多个数据包与远程主机同步一个非常大的世界。

	当以这种方式复制世界状态时，可以称每个数据包都包含世界状态增量 world state delta
	因为世界状态由对象状态组成，所以世界状态增量包含需要改变的每个对象的 对象状态增量 object state delta
	每个对象状态增量表示以下三种复制行为中的一种
	1 创建 2更新 3 销毁
	复制对象状态增量与复制整个对象状态类似，除了发送方需要向数据包中写入对象行为。
	这时，序列化数据的前缀变得复杂，以至于我们需要创建包含对象网络标识符，对象行为和必要类信息的复制头。
	enum ReplicationAction
	{
		RA_Create,
		RA_Update,
		RA_Destroy,
		RA_MAX
	};

	class ReplicationHeader
	{
	public:
		ReplicationHeader() {}
		ReplicationHeader(ReplicationAction inRA, uint32_t inNetworkId, uint32_t inClassId = 0):
			mReplicationAction(inRA),
			mNetworkId(inNetworkId),
			mClassId(inClassId)
		{}

		ReplicationAction 		mReplicationAction;
		uint32_t				mNetworkId;
		uint32_t				mClassId;

		// 方法Read和Write辅助将复制头在对象数据之前序列化到数据包的内存流中。
		// 需要注意的是，在对象销毁时，不需要序列化对象的类标识符。
		void Write(OutputMemoryBitStream& inStream);
		void Read(InputMemoryBitStream& inStream);
	};

	void ReplicationHeader::Write(OutputMemoryBitStream& inStream)
	{
		inStream.WriteBits(mReplicationAction, GetRequireBits<RA_MAX>::Value);
		inStream.Write(mNetworkId);
		if (mReplicationAction != RA_Destroy)
		{
			inStream.Write(mClassId);
		}
	}

	void ReplicationHeader::Read(InputMemoryBitsStream& inStream)
	{
		inStream.Read(mReplicationAction, GetRequiredBits<RA_MAX>::Value);
		inStream.Read(mNetworkId);
		if (mReplicationAction != RA_Destroy)
		{
			inStream.Read(mClassId);
		}
	}


	当发送方需要复制对象状态增量的集合时，创建内存流，并标记为PT_ReplicationData数据包
	然后针对每个改变序列化ReplicationHeader和恰当的对象数据。
	ReplicationManager应该包含三种不同的方法，分别实现复制创建，复制更新和复制销毁

	ReplicationManager::ReplicateCreate(OutputMemoryBitsStream& inStream, GameObject* inGameObject)
	{
		ReplicationHeader rh(RA_Create,
				mLinkingContext->GetNetworkId(inGameObject, true),
				inGameObject->GetClassId());
		rh.Write(inStream);
		inGameObject->Write(inStream);
	}

	void ReplicationManager::ReplicateUpdate(OutputMemoryBitStream& inStream, GameObject* inGameObject)
	{
		ReplicationHeader rh(RA_Update,
				mLinkingContext->GetNetworkId(inGameObject, false),
				inGameObject->GetClassId());
		rh.Write(inStream);
		inGameObject->Write(inStream);
	}

	void ReplicationManager::ReplicateDestroy(OutputMemoryBitStream& inStream, GameObject* inGameObject)
	{
		ReplicationHealder rh(RA_Destroy,
				mLinkingContext->GetNetworkId(idGameObject, false));
		rh.Write(inStream);
	}

	当接收方处理数据包时，必须采取恰当的动作
	void ReplicationManager::ProcessReplicationAction(InputMemoryBitStream& inStream)
	{
		ReplicationHeader rh;
		rh.Read(inStream);

		switch(rh.mReplicationAction)
		{
			case RA_Create:
			{
				GameObject* go = ObjectCreationRegistry::Get().CreateGameObject(rh.mClassId);
				mLinkingContext->AddGameObject(go, rh.mNetworkId);
				go->Read(isStream);
				break;
			}
			case RA_Update:
			{
				GameObject* go = mLinkingContext->GetGameObject(rh.mNetworkId);
				// we might have not received the craete yet,
				// so serialize into a dummy to advance read head
				if (go)
				{
					go->Read(inStream);
				}
				else
				{
					// 接收方仍然需要处理数据包中剩余数据，所以必须跳过内存流中恰当数量的数据
					// 如果这样效率太低，或者由于对象的构造的方式导致这种方法不可能实现
					// 可以在对象复制头添加一个字段记录序列化数据的大小。跳过内存流中对应量的数据
					uint32_t classId = rh.mClassId;
					go = ObjectCreationRegistry::Get().CreateGameObject(classId);
					go->Read(inStream);
					delete go;
				}
				break;
			}
			case RA_Destroy:
			{
				GameObject* go = mLinkingContext->GetGameObject(rh.mNetworkId);
				mLinkingContext->RemoveGameObject(go);
				go->Destroy();
				break;
			}
			defualt:
				//not handled by us
				break;
		}
	}
```

```
局部对象状态的复制
	当发送对象更新时，不需要发送对象的每个属性。只想发送改变了的属性。
	为了做到这一点，可以使用位域来表示序列化属性。
	每一个位表示一个需要序列化的属性或属性组。
	enum MouseStatusProperties
	{
		MSP_Name		 = 1<<0,
		MSP_LegCount	 = 1<<1,
		MSP_HeadCount	 = 1<<2,
		MSP_Health		 = 1<<3,
		MSP_MAX
	};

	这些枚举值可以通过按位或运算组合在一起表示多个属性。
	例如，包含mHealth和mLegCount的对象状态增量可以使用
	MSP_Health|MSP_LegCount
	需要注意的是，位域中的每一位都是1表示所有的属性都需要被序列化。

	void MouseStatus::Write(OutputMemoryBitStream& inStream, uint32_t inProperties)
	{
		inStream.Write(inProperties, GetRequiredBits<MSP_MAX>::Value);
		if ((inProperties & MSP_Name) != 0)
			inStream.Write(mName);
		if ((inProperties & MSP_LegCount) != 0)
			inStream.Write(mLegCount);
		if ((inProperties & MSP_HeadCount) != 0)
			inStream.Write(mHeadCount);
		if ((inProperties & MSP_Health) != 0)
			inStream.Write(mHealth);
	}

	void MouseStaus::Read(InputMemoryBitStream& inStream)
	{
		uint32_t writtenProperties;
		inStream.Read(writtenProperties, GetRequiredBits<MSP_MAX>::Value);
		if ((writtenProperties & MSP_Name) != 0)
			inStream.Read(mName);
		if ((writtenProperties & MSP_LegCount) != 0)
			inStream.Read(mLegCount);
		if ((writtenProperties & MSP_HeadCount) != 0)
			inStream.Read(mHeadCount);
		if ((writtenProperties & MSP_Health) != 0)
			inStream.Read(mHealth);
	}


	双向的，数据驱动的局部对象更新
	void Serizalize(MemoryStream* inStream, const DataType* inDataType,
					uint8_t* inData, uint32_t inProperties)
	{
		inStream->Serialize(inProperties);

		const auto& mvs = inDataType->GetMemberVariables();
		for (int mvIndex = 0, c = mvs.size(); mvIndex < c; ++mvIndex)
		{
			if (((1 << mvIndex) & inProperties) != 0)
			{
				const auto& mv = mvs[mvIndex];
				void* mvData = inData + mv.GetOffset();
				switch(mv.GetPrimitiveType())
				{
					case EPT_Int:
						inStream->Serialize(*reinterpret_cast<int*>(mvData));
						break;
					case EPT_String:
						inStream->Serialize(*reinterpret_cast<string*>(mvData));
						break;
					case EPT_Float:
						inStream->Serizalize(*reinterpret_cast<float*>(mvData));
						break;
				}
			}
		}
	}
		//获取inProperties的值之后立刻调用Serialize将其序列化
		//对于输出流，这将位域写入流中。
		//但是，对于输入流，会读取写属性位域到这个变量中，并覆盖传入的位域数据。
		//这是正确的行为，因为输入操作需要使用序列化的位域数据，该数据对应每个序列化的属性。
```

```
RPC作为序列化对象
	在复杂多人游戏中，主机之间发送的可能不仅是对象状态。
	比如，主机想要发送一个爆炸的声音，或者屏幕闪烁。
	这种动作的传输最好使用 远程过程调用 remote procedure call

	远程过程调用是一台主机可以在另一台或多台远程主机上执行程序的动作。
	有许多应用层协议支持它，从基于文本的，如XML-RPC 到二进制的，如ONC-RPC
	然而，如果游戏已经支持对象复制系统，那么在此基础上添加一个RPC层非常容易。

	每个过程调用都可以被认为是一个唯一的对象，每个参数对应一个成员变量。
	要调用一个远程主机上的RPC，调用主机复制一个恰当类型的对象，为目标主机正确填写成员变量。

	void PlaySound(const string& inSoundName, const Vector3& inLocation, float inValume);

	如果大量使用RPC，会导致代码复杂错乱，而且带来大量不必要的网络对象标识符，因为RPC调用对象不需要被唯一标识。
	一种清晰的解决方案是为RPC系统创建一个模块化的封装器，将其与复制系统集成。
	为了实现这种方法，首先添加一个复制动作类型RA_RPC。这个复制动作表示序列化的数据是一个RPC调用。
	允许接收端直接将其传递给专用的RPC处理模块。
	同时也告诉ReplicationHeader序列化代码这个动作不需要网络标识符，即不需要序列化网络标识符。
	当ReplicationManager的ProcessReplicationAction检测到RA_RPC动作，应当将数据包直接传递给RPC模块做进一步处理。

	typedef void (*RPCUnwrapFunc)(InputMemoryBitStream&)

	class RPCManager
	{
	public:
		void RegisterUnwrapFunction(uint32_t inName, RPCUnwrapFunc inFunc)
		{
			assert(mNameToRPCTable.find(inName) == mNameToRPCTable.end());
			mNameToRPCTable[inName] = inFunc;
		}

		void ProcessRPC(InputMemoryBitStream& inStream)
		{
			uint32_t name;
			inStream.Read(name);
			mNameToRPCTable[name] (inStream);
		}

		//每个RPC被一个四字符无符号数标识，虽然字符串更灵活，但是需要更多带宽。
		unordered_map<uint32_t, RPCUnwrapFunc> mNameToRPCTable;
	};
	// 和对象注册表很相似，通过哈希表注册函数是一种解耦表明相关系统的常用方法

	当ReplicationManager检测到RA_RPC动作时，将受到的内存流传给RPC模块做处理
	为了支持这个操作，必须为每个RPC注册一个解封装函数

	// 一个胶水函数，处理反序列化参数和用这些参数调用PlaySound
	void UnwrapPlaySound(InputMemoryBitStream& inStream)
	{
		string soundName;
		Vector3 location;
		float volume;

		inStream.Read(soundName);
		inStream.Read(location);
		inStream.Read(volume);
		PlaySound(soundName, location, volume);
	}

	void RegisterRPCs(RPCManager* inRPCManager)
	{
		inRPCManager->RegisterUnwrapFunction('PSND', UnwrapPlaySound);
	}

	最后，为了调用RPC，需要发送RPC
	void PlaySoundRPC(OutputMemoryBitStream& inStream, const string& inSoundName,
						const Vector3& inLocation, float inVolume)
	{
		ReplicationHeader rh(RA_RPC);
		rh.Write(inStream);
		inStream.Write(inSoundName);
		inStream.Write(inLocation);
		inStream.Write(inVolume);
	}

	手工生成封装和解封胶水函数，与RPCManager注册，保持它们的参数与底层函数同步是一项
	工作量繁重的任务。所有，大部分支持RPC的引擎使用构建工具，自动生成胶水函数并于RPC模块注册。

	注释：
		有时，主机希望在一个特定对象上调用一个方法，而不仅仅调用一个自由函数。
		虽然类似，但是这种方法在技术上称为 远程方法调用 remote method invocation RMI
		而不是RPC。
		支持它的游戏可以在ObjectReplicationHeader中为使用网络标识符来标识RMI的目标对象。
		标识符为0表示一个自由RPC函数，不为0表示一个特定游戏对象的RMI
		此外，为节省带宽，以牺牲代码长短为代价，新的复制动作RA_RMI强调了网络标识符字段，
		而动作RA_RPC仍然继续忽略它。
```

```
总结
	对象复制不仅涉及从一台主机向另一台主机发送序列化数据。
	首先，应用层协议必须定义所有可能的数据包类型，网络模块应该标记数据包为包含对象数据的数据包，等等。
	每个对象需要一个唯一的标识符，这样接收者可以将传入的状态指向恰当的对象。
	最后，每个对象的类需要一个唯一的标识符，这样如果接收者不存在这个对象，可以基于正确的类创建对象。
	网络代码不依赖游戏类，所以使用某种形式的间接映射来向网络模块注册类和创建函数。

	小规模游戏可以通过在输出数据包中装入世界中的每一个对象来实现主机之间共享一个世界。
	大规模游戏没办法，必须使用协议来支持世界状态的增量的传输。
	每个增量包含复制动作，即创建对象，更新对象和销毁对象。
	为了提高效率，更新对象所发送的数据可以是对象属性的子集。

	有时，游戏需要复制的不仅仅是主机之间的对象状态数据。
	经常需要在彼此的主机上触发远程过程调用。
	支持RPC调用的一种简单方法是引入RPC复制动作，并将RPC数据插入到复制数据包中。
	RPC模块可以处理RPC封装，解封装和调用函数的注册，复制管理器可以将任何传入的RPC请求传递给RPC模块。
```

---

# 第六章 网络拓扑和游戏案例

```
网络拓扑 network topology
	前面五章关注的问题是 两台计算机在互联网上通信及分享消息，以一种有利于网络游戏的方式进行。

	客户端-服务器
		一个游戏实例被指定为服务器，其他所有游戏实例被指定为客户端。
		每个客户端只能和服务器通信，同时服务器负责与所有客户端通信。
		在客户端服务器的拓扑结构中，给定n个客户端，共有O(n^2)个连接。
		但这种结构是不对称的，服务器有O(n)个连接，但每个客户端与服务器只有一个连接。
		就带宽而言，如果有n个客户端，每个客户端每秒发送b字节数据，服务器必须有足够的带宽处理
		每秒b*n个字节的传入数据。
		同样，如果服务器需要每秒发送c字节的数据给每个客户端，那么服务器必须支持每秒c*n个字节的输出。
		然而，每个客户端仅仅需要支持每秒c个字节的下载流和b个字节的上传流。
		这意味着客户端增加时，服务器的带宽要求是线性增加的。
		理论上，随着客户端的增加，客户端的带宽要求不变。
		但是，实际上支持更多的客户端会导致需要复制的世界对象数目增加，导致每个客户端的带宽要求略有增加。

		尽管不是C-S的唯一方式，大部分游戏使用一台 权威authoritative 服务器。
		这意味着我们认为游戏服务器上的游戏模拟是正确的。
		采用权威服务器方案，意味着客户端的行为会有一些滞后或延迟。
		导致这种延迟的一个重要原因是往返时间 round trip time RTT
		即数据包从发送端到目的主机，再从目的主机到发送端总共经历的时间 一般用毫秒表示
		理想情况，RTT是100毫秒或者更少，尽管有现代互联网连接，也有许多因素可能不允许如此低的RTT

		假设一个游戏有一台服务器和两个客户端，
		如果客户端A扔一个球，那么包含扔球请求的数据包首先被传递给服务器。
		接着，服务器再给客户端A和B反馈结果之前先处理这个扔球请求。
		这种情况下，客户端B经历的是最坏的网络延迟，等于客户端A的RTT的1/2，加上服务器的处理时间，
		再加上客户端B的RTT的1/2。
		在快速网络的情况下，这可能不是一个问题，但实际上，大多数游戏必须使用多种技术来隐藏这个延迟。

		服务器还可以被细分。
		一些服务器是专用 dedicated 的，意思是它们只运行游戏状态并与所有客户端通信。
		专用服务器进程与运行游戏的所有客户端进程是完全分开的。
		这意味着专用服务器是无外设的，实际上不显示任何图像。
		这种类型的服务器通常用于预算较多的游戏，如战地Battlefield
		允许开发者在一台高性能的计算机上运行多个专用服务器进程。

		专用服务器的另外一种替代方案是监听服务器 listen server
		在这个设置中，服务器也是游戏本身的积极参与者。
		监听服务器方案的一个优点是降低部署成本，因为不需要再数据中心租用服务器。
		相反地，玩家可以使用自己的计算机既作为服务器也作为客户端。
		但是，监听服务器的不足是作为监听服务器的计算机性能必须足够高，
		而且需要足够快的网络连接以应付服务器的额外负载。
		监听服务器方案有时被错误地称为 对等网络连接，
		但是一个更准确的方法是 对等托管 peer hosted
		仍然有一台服务器，只是恰巧由游戏玩家托管。

		需要注意的是，我们假设监听服务器是权威的，保存完整的游戏状态。
		这意味着运行监听服务器的玩家可能使用该信息来欺骗。
		进一步，再C-S模型中，通常只有服务器知道所有活动客户端的网络地址。
		如果服务器断开，不管是网络原因，还是玩家恶意退出游戏，都导致巨大问题。
		一些使用监听服务器的游戏实现一种 主机迁移 host migration 的概念
		意思是如果监听服务器断开，客户端中的一个被晋升为新的服务器。
		但是，要想实现这一点，客户端之间需要有一定量的通信。
		这意味着主机迁移需要有一个结合C-S拓扑和对等网络拓扑的混合模型。

对等网络
	在对等网络拓扑中，每个单独的参与者都与其他所有的参与者连接。
	意味着客户端之间有大量数据来回传输。
	连接的数量是一个二次函数。
	给定n个对等体，每个对等体必须有O(n-1)个连接，所有网络中产生O(n^2)个连接。
	这也意味着，每个对等体的带宽需求增加到与连接到游戏中的对等体个数一致。
	但是和C-S不同，带宽需求是对称的，所以每个对等体需要上传和下载的可用带宽数量是一样的。

	在对等网络中，权威的概念更加模糊。
	一种可行的方法是，某些对等体对游戏的某些部分有权限，但是实际中，这样的系统难以实现。
	在对等游戏中，更常见的作法是每个对等体共享所有动作，每个对等体都模拟这些动作的执行。
	这种模式有时也被称为输入共享 input sharing 模型。

	对等网络拓扑让输入共享更可行的一个方面是更少的延迟。
	与客户端-服务器模型不同，对等网络的客户端之间没有中介，所有的对等体彼此之间直接通信。
	这意味着最坏情况下，对等体之间的延迟是RTT的1/2，但是仍存在一定的延迟，
	这可能会导致对等网络游戏中最大的技术挑战：确保所有对等体保持彼此同步。

	第一章讨论的确定性锁步模型展示了这种方法。
	帝国时代Age of Empires 的实现中，游戏被细分为200毫秒一轮。
	这200毫秒中所有的输入命令被放入队列中，当200毫秒结束，再将这些命令发送给所有对等体。
	此外，有一个一轮的延迟，这样当每个对等体在显示第一轮结果时，队列中的命令等到第三轮再被执行。
	尽管这种轮同步的方式概念上很简单，但是真正实现的细节非常复杂。

	更重要的是，保证所有对等体之间的游戏状态一致。
	这意味着游戏的实现需要非常确定。一组给定的输入必须始终得到同样的输出。
	该问题的几个重要方面包括，使用检验和来验证对等体之间的游戏状态的一致性和在所有对等体之间同步随机数发生器。
	本章后面将详细讨论这两个问题。

	对等网络的另外一个问题是连接新玩家。
	因为每个对等体都必须知道其他所有对等体的地址，所以理论上新玩家连接到任意一个对等体即可。
	但是，一般来说提供当前可玩游戏的游戏匹配服务通常只接受一个地址，
	在这种情况下，只有一个对等体被选为主对等体，可以欢迎新玩家的唯一对等体。

	最后，C-S模型中考虑的服务器断开问题在对等网络中不存在。
	通常情况下，如果一个对等体通信中断，游戏暂停几秒，将该对等体从游戏中去除。
	一旦这个对等体断开了，剩下的对等体继续玩这个游戏。
```

```
C-S的实现
	Robo Cat Action第一版
	重大假设：网络延迟很小或者没有，并且所有的数据包都能到达目的地。
	对于所有网络游戏，这显然不切实际，随后的章节会讨论和删除这些假设。

	服务器和客户端代码分离
		C-S模型的一个基础是，服务器代码和客户端代码不同。
		RoboCat函数有些客户端需要，有些服务器需要，有些都需要。
		这里用继承和虚函数，RoboCatServer RoboCatClient
		使用虚函数的方式可能无法得到最高性能，但是从易用性的角度看，
		继承的体系结构可能是最简单的。

		注释：
			因为服务器和客户端有两个分离的可执行代码，所以为了测试游戏
			必须分别运行这两部分代码。
			服务器需要一个命令行参数指定接收连接的端口。
			RoboCatServer 45000
			这指明服务器应该在45000端口监听客户端连接请求。
			客户端可执行代码需要两个命令行参数：服务器详细地址包括端口，和请求连接的客户端名字
			RoboCatClient 127.0.0.1:45000 John
			这指明客户端想要连接本地端口为45000的服务器，玩家名字为"John"。
			当然多个客户端可以连接到一台服务器，该游戏没有使用很多资源，一台机器可以测试多个实例。

NetworkManager
	读入可用数据包到一个数据包队列等待处理
	处理新客户端加入游戏 支持多人玩家随时加入和退出
		1 当一个客户端想要加入游戏，首先向服务器发送"hello"数据包
		  该数据包仅仅包含文本"HELO"和表示玩家名字的序列化字符串
		  客户端在收到服务器应答前持续发送hello数据包
		2 一旦服务器接收到hello数据包，服务器就给新玩家分配一个玩家ID，
		  同时做一些记录工作，例如将传入SocketAddress与玩家ID关联。
		  然后服务器向该客户端发送一个"welcome"数据包。
		  该数据包包含文本"WLCM"和分配给玩家的ID
		3 当客户端收到welcome数据包，它保存自己的玩家ID，并开始给服务器发送
		  复制信息和接收服务器的复制信息。
		4 在未来的某个时刻，服务器会给新客户端和已有的客户端发送任何由新客户端产生的对象信息。

		在这种特殊情况下，为数据包丢失构建冗余系统是非常简单的。
		如果客户端没有收到welcome数据包，它会继续给服务器发送hello数据包。
		如果服务器收到来自文件中已有SocketAddress的客户端的hello数据包，它只会重新发送welcome数据包

		服务器直接使用套接字地址，所以可能产生NAT穿越问题。
		当服务器首先收到一个数据包，它到地址映射表中查找发送方是否已知。
		如果发送方未知，那么服务器检查数据包是否为hello数据包。
		如果数据包不是hello数据包，那么直接忽略。
		此外，服务器为新客户端创建一个代理，并发送一个welcome数据包

		客户端给服务器发送输入事件，然后服务器接收这个输入数据包，将输入保存到客户端代理 client proxy
		客户端代理是服务器用于跟踪特定客户端的一个对象。
		最后，当服务器更新游戏模拟的时候，将会考虑存储在客户端代理中的所有输入。

		InputState类
		// 由InputManager每帧更新一次。
		// 对于大多数游戏，以相同的频率给服务器发送InputState是不现实的
		// 理想情况下，将经理几个帧的InputState合并为一个动作
		// 为了简单起见，当前游戏没有以任何形式合并InputState
		// 而是每个x秒，抓取一次当前的InputState，并将其保存为Move

		Move类
		实质上是InputState的封装，增加了两个浮点数，一个用于跟踪Move的时间戳，
		一个用于跟踪当前动作与上一次动作的时间差
		仅仅是InputState加上时间变量的轻量级封装，做出这样的区别是为了在帧到帧的基础上
		允许更清晰的代码。
		InputManager以帧的频率轮询键盘，并将数据保存到InputState中。
		只有当客户端实际需要创建也给Move，时间戳才有意义。

		MoveList类
		接下来，一系列动作存储在MoveList中。
		这个类包含动作的列表和列表上一个动作的时间戳。
		在客户端这边，客户端确定了它应该存储一个新的动作，他就会将这个动作添加到动作列表中。
		然后NetworkManagerClient将会在合适的时间将动作序列写出到输入数据包。
		需要注意的是，通过假设不会同时写三个以上的动作，写动作序列的代码优化到了比特级别。
		它可以根据设定动作和输入数据包的频率这样的恒定因素来做这个假设。

		void NetworkManagerClient::SendInputPacket()
		// 发送输入
		// 使用MoveList的数组索引操作符
		// MoveList内部使用deque数据结构。所以这个操作是常数时间。
		// 在冗余方面，SendInputPacket的容错的确不是很好
		客户端只发送一次动作消息。
		例如：如果输入数据包包含一个 扔球 的输入请求，但是这个数据包没有到达服务器，那么客户端永远不会 扔球
		很显然，这在多人游戏中是不合理的。
		第七章会给输入数据包添加冗余。为了给服务器三次识别动作的机会，每个动作会被发送三次。
		这增加了服务器端的复杂度。因为服务器在收到输入动作时需要判断是否已经处理过了。

		ClientProxy类
		正如之前所提到的，客户端代理是服务器用于跟踪每个客户端状态的。
		客户端代理的最重要职责之一是它为每个客户端创建一个单独的复制管理器。
		允许服务器完全掌握它已经有了什么信息，或者还没有发送给客户端什么信息。
		因为服务器不是每帧都给每个客户端发送一个复制数据包，所以每个客户端有一个单独的复制管理器十分必要。
		这一点在添加冗余时尤其重要。因为服务器可以知道需要给特定客户端重发的确切变量。
		每个客户端代理也保存了对应玩家的套接字地址，名字和ID。
		客户端代理还存储对应客户端的动作信息。
		当收到输入数据包时，与客户端相关的所有动作被添加到客户端代理中。


		RoboCatServer::Update()
		使用未被处理的动作数据，
		需要注意的是
			// 每次调用ProcessInput和SimulateMovement时，传入的增量时间，是根据
			// 两次动作之间的增量时间，而不是服务器帧的增量时间。
			// 这使得服务器即使在一个数据包中收到多个动作
			// 也可以试图保证尽可能模拟客户端行为。
			// 这样还允许服务器和客户端以不同的帧率运行。
		对于需要以设定步长模拟的物理对象，这样做可能会增加一些复杂性。
		如果你的游戏中存在这种情况，你会锁定物理对象的帧率，与其他帧率分隔开。
```

```
对等网络的实现
	RoboCatRTS 尽管这两个游戏都是用UDP
	RoboCatRTS使用的网络模型和RoboCatAction游戏不同。
	作为动作游戏，RTS的最初版本假设不存在数据包丢失。
	但是，由于锁步回合的本质，游戏仍然存在一定量的延迟，如果延迟太高，体验肯定下降。
	因为 RoboCatRTS使用对等网络模型，所以不需要将代码分离成多个工程。
	每个对等体都使用相同的代码，这样减少了文件的数量，也意味着游戏中所有玩家使用相同可执行文件。

	注释：
		有两种不同的方式启动RoboCatRTS，虽然两者都使用相同的可执行文件。
		要初始化为对等主体，那么指定一个端口号和玩家的名字
		RoboCatRTS 45000 John
		要初始化为普通对等体，指定主对等体的详细地址(包括端口号) 和玩家的名字
		RoboCatRTS 127.0.0.1:45000 Jane
		需要注意的是，如果指定的地址不是主对等体，玩家仍然能连接成功，但是指定主对等体更快一些。

	但是，RoboCatRTS使用了 主队等体 master peer 的思想。
	主队等体的主要目的是提供游戏中已知对等体的IP地址。
	当使用了游戏匹配服务来保存已知的可用游戏列表时，这是非常重要的。
	此外，只有主对等体可用给新玩家分配玩家ID。
	这主要是为了避免在两个不同玩家同时连接多个对等体时所产生的竞争。
	除了这一特例，主对等体与其他所有对等体行为一致。
	因为每个对等体独立存储整个游戏的状态，所以如果主对等体断开了，
	游戏仍然可继续。


	欢迎新对等体和开始游戏
		对等网络游戏的欢迎过程更复杂。
		新对等体首先发送一个带有各自玩家名字的"hello"数据包
		但是，这里的hello数据包可以有以下三种响应中的一种
		1 Welcome 'WLCM'
		  意味着主对等体已经收到hello数据包
		  并欢迎新对等体进入游戏。
		  这个欢迎数据包包括新对等体的玩家ID，主队等体的玩家ID和游戏中玩家个数（不包括新玩家）
		  此外，这个数据包还包括所有对等体的名字和IP地址。
		2 Not joinable 'NOJN'
		  意味着游戏已经在进行中，或者游戏玩家已满。
		  如果新玩家收到这个数据包，游戏退出。
		3 Not master peer 'NOMP'
		  如果hello数据包发送给了非主队等体，就会收到这个响应。
		  在这种情况下，数据包将包括主队等体的地址，使得新玩家可以给主队等体发送hello数据包
		  然而，即使新对等体收到了欢迎数据包，这个过程还是不完整的。
		  新对等体还要负责给游戏中的其他所有对等体发送一个介绍数据包 'INTR'
		  该数据包包括新对等体的玩家的ID和名字。
		  这样保证了游戏中每个对等体都在它们用于跟踪游戏玩家的数据结构中存储了这个新对等体。
		  因为每个对等体存储的地址是根据传入数据包收集的地址，所以当一个或多个对等体连接在局域网时
		  就可能出现潜在的问题。
		  例如：假设Peer A是主对等体，Peer B与Peer A在同一个局域网中。
		  这意味着Peer A的对等体地图中将包括Peer B的局域网地址。
		  现在假设一个新的对等体Peer C，通过外部IP连接到Peer A。
		  Peer A将欢迎Peer C加入游戏，并向Peer C发送Peer B的地址。
		  但是所提供的Peer B的地址是不能被Peer C访问的，因为Peer C与Peer A和Peer B在不同的局域网。
		  这样Peer C不能与Peer B通信，所以不能正确地加入游戏中。

		  回想一下 NAT穿越的方式解决这个问题。其他的方法还包括以某种方式使用外部服务器。
		  在一种方法中，外部服务器，有时被称为集中服务器 rendezvous server
		  仅仅用于对等体之间的初始化通信。

		  一些游戏服务使用的另外一种方法是有一个中央服务器处理对等体之间的整个数据包路由过程。
		  意思是所有对等体的数据首先通过中央服务器，然后再被路由到正确的对等体。
		  虽然第二种方法需要一个更强大的服务器，但是保证了任何一个对等体都不知道其他对等体的公网IP地址。
		  从安全的角度看，这可能是优选的。
		  例如：它会阻止一个对等体通过分布式拒绝服务攻击的方式来断开另外一个对等体。

		  另外一个值得考虑的边缘情况是，如果一个对等体只能连接游戏中的部分玩家，将会发生什么。
		  即使在集中服务器或者中央服务器路由数据包的情况下也会发生。
		  最简单的解决方案是不让这个对等体进入游戏，但是你需要额外的代码来跟踪这种情况。
		  因为本章假设不担心连接的问题，所以没有提供处理这个问题的代码。
		  但是商业对等网络游戏绝对需要包含处理这一情况的代码。

		  当主对等体按下回车，将给游戏中的每一个对等体发送一个开始数据包 'STRT'
		  通知所有对等体进入3秒倒计时状态，一旦倒计时为0 比赛正式开始。

		  这种开始方法非常简单，因为计时器并没有真正弥补主队等体和其他对等体之间的延迟。
		  因此，主队等体总会在其他对等体之前开始游戏。
		  由于锁步模型，所有不会影响游戏的同步，但是可能意味着主对等体的游戏不得不暂停等待其他对等体。
		  解决这个问题的一种方法是每个对等体从计时器种减去1/2的RTT时间。
		  所以如果到Peer A的主队等体的RTT是100毫秒，那么Peer A从总计时时间种减去50毫秒，
		  这样允许它更好地同步。


	命令共享和锁步回合制
		RoboCatRTS以30帧每秒的速率运行，同时锁定的增量时间是~33毫秒。
		意思是某个对等体以多于33毫秒的时间渲染一帧，但是模拟仍然认为它是以33毫秒运行。
		游戏将33毫秒标记为 子轮。 每一轮有三个子轮，这样，每一轮的长度是100ms，换句话说， 每秒有10轮。

		理想情况下，轮和子轮的持续时间应该根据网络和性能条件而变。
		事实上，这是Bettner和Terrano在讨论 帝国时代 的论文中探讨的问题之一。
		但是为了简化，本例不会调整轮和子轮的持续时间。

		对于复制而言，每个对等体运行游戏世界的完整模拟。这意味着，对象不以任何方式进行复制。
		而是在游戏中只传输 轮 数据包。这些数据包包含在某一轮中每个对等体发出的一系列命令和其他一些关键数据。
		应当指出的是， 命令 和输入之间有清晰的界限。
		例如，左键单击一只猫是选中这只猫。
		但是，因为这个选择动作不会以任何方式影响到游戏状态，所以不会产生命令。
		另外，如果选中了一只猫，并单击右键，那么意味着玩家想让这只猫移动或发出攻击。
		因为这两个动作将影响到游戏状态，所以都会产生命令。

		此外，命令不是在发出的一瞬间就被执行了。而是，每个对等体将某一轮中发生的所有命令存储到队列中。
		在这轮结束的时候，每个对等体给其他所有对等体发送自己的命令列表。 这个命令列表在未来的某一轮中被执行。
		具体而言，是第x轮发生的命令将在第x+2轮被执行。允许每个对等体在大约100毫秒的时间里接收和处理数据包。
		这意味着，在正常的情况下，从命令发出到执行有高达200毫秒的延迟。
		但是，因为延迟是一致的，不会影响游戏体验。

		Command类
		类的实现是自解释的。
		实际执行命令时，使用纯虚函数ProcessCommand
		StaticReadAndCreate首先从内存比特流中读取命令类型的枚举。
		然后基于枚举类型的取值，构建一个合适子类的实例，并调用子类的Read函数
		这个例子里只有两个子类。 move将猫移动到目标位置，attack告诉猫攻击敌人的猫
		移动命令有一个Vector3存储移动的目标
		每个命令都已一个静态的工厂方法 StaticCreate，创建shared_ptr
		StaticCreate的参数是接收命令猫的网络ID和目标位置，该函数验证以确保发给了真实存在的游戏对象。

		CommandList
		每个对等体的输入管理器都有一个CommandList的实例。
		当本地对等体使用键盘或鼠标请求命令时，输入管理器将这个命令添加到列表中。

		TrunData类
		用于对每个完成的100ms的轮

		void NetworkManager::UpdateSendTurnPacket()
		// 传入TurnData构造函数的两个参数 随机数和CRC 将在下一节介绍
		// 目前需要注意的是，对等体准备轮数据包，内容包括从现在开始两轮之后执行的所有命令的列表
		// 然后将这个轮数据包发送给所有对等体，此外，对等体在清空输入管理器的命令列表前，本地保存自己的轮数据
		// 最后，有代码来检查十分存在负数的轮序号
		// 游戏开始时，轮序号为-2。这样在第-2轮发出的命令将会在第0轮执行。
		// 这意味着在头200ms，没有命令执行，但是，没有办法避免这种初始延迟，它是锁步回合制的一个属性

		void NetworkManager::TryAdvanceTurn()
		// 如此命名的原因是不能保证轮能够继续推进
		// 因为它的责任是确保轮的锁步属性。
		// 事实上，如果当前是第x轮，并且已经收到第x+1轮的所有数据，那么函数将推进到第x+1轮
		// 如果第x+1轮的有些数据丢失了，那么网络管理器将进入延迟状态。
		当处于延迟状态时，不能更新游戏世界种的对象。而是网络管理器等待需要接收的数据。
		在延迟状态时，每当接收了新的轮数据包，网络管理器都会再一次调用TryAdvanceTurn
		希望新的轮数据包填补轮数据的缺口。
		重复这个过程，知道收到了所有需要的数据。
		同样的，如果在延迟时连接被重置了，游戏中删除重置的对等体，其他对等体将试图继续


保持同步
	同步伪随机数生成器
		伪随机数生成器 pseudo-random number generator PRNG
		是计算机获取看似随机的数字的唯一方法。
		在对等网络游戏中，有必要保证在特定的轮，两个对等体总是从一个随机数生成器收到相同的结果。
		为了保证每个对等体产生相同的数字，主要有两件事情需要做：
			1 为每个对等体的随机数生成器设置相同的种子。当主对等体发送开始数据包时，选择一个种子。
			开始数据包中包含这个种子，所以每个对等体都知道开始游戏时的种子取值。
			2 必须保证，每个对等体在每一轮总调用相同次数的伪随机数生成器，同样的顺序，在代码中同样的位置。
			这意味着几乎不能创建游戏的不同版本，这些版本使用伪随机数生成器的次数有差异。
			例如为跨平台运行中不同硬件编写的版本。
			3 还有第三个看起来不太明显的问题。函数rand和srand不太适合用于保证同步。
			不同平台的C语言库的不同实现不能保证使用的是同一个伪随机数生成器算法。
			这样只保证随机数种子相同没有意义。
			在过去，定义不清的rand函数意味着大多数游戏都要实现自己的伪随机数生成器。
			C++11引入了标准化和高质量的伪随机数生成器。
			尽管没有考虑安全加密（安全加密意味着随机数作为安全协议的一部分使用时是安全的）
			但是游戏使用绰绰有余。
			具体而言，RoboCatRTS的代码实现了伪随机数生成器的梅森旋转算法Mersenne Twister PRNG algnorithm
			被称为MT19937的32位梅森旋转算法取值区间为2^19937，意味着事实上在给定游戏的过程中随机数序列是不会重复的。
			C++11随机数生成器的接口比之前的rand和srand函数要复杂一点，所以封装到RandGen中。

		RandGen类
		// 当RandGen首先被初始化时，它使用random_device类产生随机数种子
		// 这将产生一个平台特定的随机数。
		// 随机数设备可以被用作产生一个随机数生成器的种子，但是设备本身不应该被用作生成器
		// 在一个函数中使用uniform_int_distribution类简单地说就是指定一个范围，并获得这个范围内的伪随机数

		void NetworkManager::TryStartGame()
		// 主对等体生成一个随机数，用作倒计时开始时的新种子。
		// 这个随机数被发送给所有其他的对等体，来保证当第-2轮开始时，所有对等体的生成器种子都相同

		此外，在每轮结束时创建轮数据包，每个对等体生成一个随机数。
		这个随机数作为轮数据包中轮数据的一部分被发送。这样便于对等体验证在轮推进的过程中，所有的随机数生成器保持同步。

		游戏需要其他不影响游戏状态的随机数生成器，其他程序员必须理解什么时候用哪个伪随机数生成器。

	检查游戏同步
		不同步的其他原因可能就没有伪随机数生成器那么明显了。
		例如，浮点数的实现时确定的，但是根据硬件实现的不同有出入。
		例如，更快的SIMD指令可能和普通的浮点指令产生不同的结果。
		通常也可以在处理器上设置不同的标志来改变浮点数的行为，例如是否严格遵循IEEE 754的实现。

		同步的其他问题可能只是由程序员引起的一个意外错误。
		也许程序员不知道同步如何工作，或者只是犯了一个错误。
		无论哪种，重要的是，游戏中有用于定期检查同步的代码。
		这样，在引入不同步之后有希望尽快找到不同步的错误原因。

		一种常见的方法是使用检验和 checksum，与网络中用于保证数据包数据完整性的检验和类似。
		本质上，在每一轮结束时，就计算好了游戏状态的检验和。
		检验和被放入轮数据包中并发送，这样每个对等体可以验证所有游戏实例在每轮结束时是否计算得到相同的检验和。

		检验和的算法选择方面，有很多不同选项。RoboCatRTS使用循环冗余检验 cyclic redundancy check CRC
		生成一个32位的检验和值。
		这个游戏不是从头实现CRC函数，而是使用来自开源库zlib的函数crc32。
		这是很方便的方法，因为为了使用PNG图像文件，我们已经依赖zlib了。
		此外，因为zlib是被设计为高效处理大规模数据的，所以按理说CRC的实现既是经过检验的，也是高效的。

		uint32_t NetworkManager::ComputeGlobalCRC()
		// 使用OutputMemoryBitStream类
        // 游戏世界中的每个对象通过函数WriteForCRC写其相关的数据到所提供的比特流中。
        // 按照网络ID的升序作为整体来计算CRC
        // 有几个额外的事项需要考虑：
        // 首先，并不是每个游戏对象的每个值都需要写入流中。
        // 在RoboCat类的例子中，写入的值包括玩家的ID，网络ID，位置，生命值，状态和目标网络ID
        // 一些其他成员变量，例如投球之间的冷却变量，不被同步。这种选择降低了计算CRC所花费的时间。
        // 此外，因为CRC函数可以增量计算，所以实际上没有必要在计算CRC之前将所有的数据写入流中。
        // 事实上，复制数据可能比立即计算CRC的每个值更低效。甚至有可能写出类似OutputMemoryBitStream的接口
        // 本质上只是计算被写入CRC的类的实例，但是并不将数据保存到内存缓冲区。
        // 但是，为了保持代码简洁，重用现在的OutputMemoryBitStream

		void NetworkManager::TryAdvanceTurn()
		// 在推进轮的时候调用函数CheckSync。
		// 这个函数循环遍历每个对等体轮数据的所有随机数和CRC值，保证每个对等体在发送轮数据包时，
		// 计算得到相同的随机数和相同的CRC值
		// 如果CheckSync检测到不同步，RoboCatRTS直接立刻结束游戏
		一个更鲁棒的系统将采用投票的形式，假设有四个玩家，三个还是同步的，这时游戏还可能继续。

		在相同情况下，一个更复杂的方法是为了重新同步游戏而复制整个游戏状态到不同步的玩家。
		根据游戏的数据量，这可能不切实际。但是需要记住，当玩家不同步时，不让他们退出游戏是很重要的。

	警告：
		当开发独立模拟形式的对等网络游戏时，不同步时焦虑的根源。
		不同步的错误往往是最难解决的。最重要的是实现一个日志系统，用于查看每个对等体所执行的命令的细节。

		在开发RoboCatRTS实例时，如果猫移动时客户端进入延迟状态，那么将发生不同步。
		原因是当延迟的玩家继续游戏时，他们错过了一个子轮。
		这是确定的，幸亏有日志系统，可以记录什么时候对等体执行了一个子轮，以及在每个子轮结束时每只猫的位置。
		这样就可以查看那个错过子轮的对等体。
		如果没有日志，查找和解决问题非常耗时。
```

---

# 第七章 延迟 抖动 可靠性

```
网络游戏处于非常恶劣的环境，在过时的网络上竞争带宽，给遍布世界的服务器和客户端发送数据包
这导致了通常不会发生在本地网络的数据丢失和延迟。
本章介绍了一些多人游戏面临的网络问题，并提出这些问题的解决方案。
包括如何在UDP传输协议的基础上建立一个自定义的可靠层。
```

```
延迟 latency

不同上下文有不同的含义
	在电脑游戏的上下文中，指的是从可观察的原因看到结果的时间
	根据游戏的类型，可以是从RTS real-time strategy中鼠标单机和单元响应命令之间的时间间隔
	到用户移动头部和VR virtual reality显示更新之间的时间间隔
一定量的延迟是不可避免的，并且不同的游戏类型对于延迟的容忍程度也不同。
	虚拟现实VR对延迟最敏感，因为人类只要头转了，眼睛就想看到不同事物。
		保证用户感觉在VR世界中延迟少于20ms
	格斗游戏，FPS和其他动作频繁的游戏对于延迟是第二敏感的。
		这些游戏的延迟范围可以从16ms到150毫秒，不考虑帧速率，这么少的延迟下用户是感觉不到的。
	RTS游戏是对延迟容忍度最高的，这个容忍度通常很有用，延迟可以高达500ms，而不影响用户体验。

非网络延迟
	输入采样延迟 input sampling latency
		从用户按下一个按钮到游戏检测这个按钮之间的时间可以很长。
		平均情况下，从按下按钮到游戏响应那个按钮有半帧时间的延迟。
	渲染流水线延迟 render pipeline latency
		GPU不是在CPU批量发布绘制命令后马上执行这些命令。
		事实上，驱动程序将这些命令插入到命令缓冲区，GPU在将来的某个时刻执行这些命令。
	多线程渲染流水线延迟 multithreaded render pipeline latency
		多线程游戏将更多的延迟引入到了渲染流水线中。
		通常，一个或多个线程运行游戏模拟，更新游戏时间时要发送给一个或多个渲染线程。
		然后再模拟线程准备模拟下一帧时，这些渲染线程批量处理GPU请求。
	垂直同步 VSync
		为了避免画面撕裂，通常做法是仅仅在显示器的垂直消隐间隙改变由视频卡显示的图像。
		如果视频卡没有准备好，后台缓冲区内容显示到显示器的命令将延迟，等待下一个间隙。
	显示延迟 display lag
		大部分HDTV和LCD显示器在真正显示图像之前，都会在一定程度上处理输入，很容易增加几十毫秒延迟。
	像素响应时间 pixel response time
		LCD的问题是像素亮度的改变需要时间。通常是几毫秒，但是老的显示器很容易添加额外半帧延迟。

	非网络延迟是一个很严重的问题，John Carmark "我给欧洲发送一个IP数据包都比向屏幕发送一个像素还快。
		怎么会这样？"
	在单人游戏中已经有一定量的延迟了，当引入多人游戏功能时，我们需要尽可能减轻网络延迟。
	要做到这一点，理解网络延迟的根源很重要。

网络延迟
	1 处理延迟 processing delay
		网络路由器的工作是：读取来自网络接口的数据包，检查目的IP地址，找出应该接收数据包的下一台机器，
		然后从合适的接口将数据包转发出去。
		检查源地址和确定合适路由器的时间称为处理延迟。
		处理延迟也包括路由器提供的其他功能，例如NAT或者加密。
	2 传输延迟 transmission delay
		路由器转发数据包时，必须有一个链路层接口允许它通过一些物理介质传输数据包。
		链路层协议控制写入物理介质的平均速率。
		例如，1MB以太网连接允许大约每秒向以太网电缆写入100万bit
		这样向1MB的以太网电缆写一个bit需要花费1秒的百万分之一，即1微秒
		因此写一个1500bit数据包需要12.5ms
	3 排队延迟 queuing delay
		路由器在一个时间点只能处理有限个数据包。
		如果数据包到达的速度比路由器处理的速度快，那么数据包将进入接收队列，等待被处理。
		同样，网络接口一次只能输出一个数据包，所以数据包被处理之后，如果合适的网络接口繁忙。
		那么数据包将进入传输队列。在队列中消耗的时间被称为排队延迟。
	4 传播延迟 propagation delay
		物理介质的延迟 发送数据包的延迟至少是0.3ns/m

	这些延迟有些可以优化，有些不能被优化。
	处理延迟是很小的因素，因为现在的路由器非常快。
	传输延迟在互联网边缘时是最大的。升级告诉互联网可以很好地降低网络延迟。
	发送尽可能大的数据包也会有帮助，因为可以减少数据包头部的数据量。
	排队延迟是数据包等待被传输和处理的结果。最小化处理延迟和传输延迟有助于最小化排队延迟。
	发送少量大数据包来代替小数据包可以降低总的排队延迟。
	传播延迟通常是优化的良好对象，它依赖于主机之间交换数据的电缆长度，最好的方法是移动主机使彼此接近。
		在对等网络游戏中，这意味着匹配玩家时优先优化几何位置。
		在C-S游戏中，这意味着要保证游戏服务器离客户端近。
	在网络上下文中，工程师有时用延迟描述上面四种延迟的组合。
	所以游戏开发者经常讨论 往返时间 round trip time RTT
	这不仅反映了两个方向的处理延迟，排队延迟，传输延迟，传播延迟，还反映了远程主机的频率，
	因为这影响了它发送相应包的速度。
	注意，在每个方向上传输的速度不一定相同，RTT除二是不准确的。尽管这样，游戏往往也用一半RTT来近似。
```

```
抖动
	RTT一般围绕着一个基于平均延迟的特定值变化。但是这些延迟随着时间的推移会变化，
	导致RTT与期望值有偏差。这个偏差称为抖动 jitter

	这四个网络延迟都能导致抖动
		处理延迟 processing delay
			是网络延迟中最小的组成部分，所以对抖动的贡献也是最小的。
			次要问题
		传输延迟和传播延迟 transmission delay and propagation delay
			都是数据包所采用的路由导致的，链路层协议决定了传输延迟，路由长度决定了传播延迟
			当路由器动态进行负载均衡和改变路由以避免严重拥堵区域，这些延迟会改变。
			这在网络堵塞时可以迅速波动，路由改变可以显著改变往返时间。
		排队延迟 queuing delay
			是路由器必须处理多个数据包导致的。
			突发的网络流量将导致排队延迟，并改变往返时间。

	抖动会影响RTT抑制算法，更糟的是，会导致数据包乱序到达。
	为了避免乱序到达引起的错误，必须使用TCP，或者自定义的系统进行包重组。

	由于抖动会导致问题，应该尽量减少抖动来提高游戏体验。
	减少抖动的技术与降低延迟总体类似：
		发送尽可能少的数据包来保持低流量。
		将服务器布置在玩家附近来降低严重拥堵的可能性。

	帧率也会影响RTT，所以帧率的巨大变化也会给客户端带来负面影响。
	保证复杂的操作合理分散在多个帧，防止由帧率导致的抖动。
```

```
数据包丢失 packet loss
	数据包需要很长时间到达，和数据包永远不能到达是两码事。

	不可靠的物理介质 unreliable physical medium
		如松动的连接或者附近有微波炉在工作。
	不可靠的链路层 unreliable link layer
		因为链路层不保证可靠性，所以这是完全可接受的一个响应。
	不可靠的网络层 unreliable network layer
		数据包到达路由器的速度比处理数据包的速度快。
		就会将数据包插入接收队列中，队列满了，路由器开始抛弃数据包。

	数据包丢失是无法改变的事实，设计网络架构时必须考虑到这点。
		使用离玩家近的服务器中心，较少的路由器和电缆意味着较低的数据丢失可能性。
		发送尽可能少的数据包，大部分路由的处理能力是以数据包个数为基础的，而不是数据总量。
			并不是所有路由器都这样，有些路由器是根据输入带宽给资源分配空间的，这种情况小包更好。
			有些路由器优先丢弃UDP报文
		了解数据中心和目标市场ISP附近的路由器配置有助于调整数据包类型和传输模式。
		最后，减少数据包丢失的最简单办法是保证服务器有快速稳定的互联网连接，并离客户端尽可能近。
```

```
可靠性： TCP还是UDP
	TCP主要优点：
		提供了经得起时间考研的，鲁棒的，稳定的可靠性实现。
		没有额外工作，保证所有数据按序送达。
		提供了复杂的拥塞控制功能，通过以不会阻塞中间路由器的速率发送数据来限制数据包丢失。
	TCP主要缺点：
		发送的所有东西必须被可靠发送并按序处理。
		在游戏状态瞬息万变的多人游戏中，有三种不同常见可能造成问题
			1 低优先级数据的丢失干扰高优先级数据的接收
				爆炸声不重要，爆头重要，爆炸声丢失，妨碍了爆头的处理
			2 两个单独的可靠有序数据流互相干扰
				聊天重要，爆头重要，聊天丢失，妨碍了爆头的处理
			3 过时游戏状态的重传
				玩家B发送位置，服务器每秒发送5个数据包，如果一个丢失，全部重传。
				这样玩家A看到玩家B的位置非常过时。导致看到之前就被B击中。玩家A不能接受
	TCP其他缺点：
		尽管拥塞控制有利于防止丢失数据包，但并非所有平台都是统一可配置的。
		有时可能导致游戏发送数据包的速度比你期望的慢。
		Nagle算法起到了非常不好的作用，因为它在将数据包发送出去之前延迟长达半秒。
		游戏通常禁用Nagle算法来避免这个问题，同时放弃了它提供的减少数据包数量的优势。

		TCP为了管理连接和跟踪所有可能的重传数据分配了很多资源。
		这些分配通常由操作系统管理，游戏需要时很难通过自定义内存管理器的方式跟踪和路由。

	UDP的特点：
		可以自定义可靠系统，允许发送可靠的，不可靠的数据。分离可靠有序数据流的交错。
		可以丢包时只发最新消息，而不是重传丢失数据。
		可以自己管理内存，精细控制。
		所有这些都增加了开发和测试的时间。不会像TCP那样成熟和没有错误。
		可以使用第三方UDP网络库来减少一些风险，如RakNet或Photon，尽管需要牺牲一些灵活性。
		使用UDP会增加丢包风险。因为路由器可能优先丢弃UDP包。

	游戏发送的每个数据都需要被接收吗？需要以完全有序的方式进行处理吗？
	如果时，应该考虑TCP。在回合制游戏往往这样。
	大多数游戏中，TCP都不是完美适合你的游戏。应该用UDP自定义可靠系统。
```

```
数据包传递通知
	使用UDP实现一个可靠系统。
	可靠性的首要要求是，有能力知道数据包是否到达目的地。
	要做到这一点，需要创建某种形式的传递通知模块。
	该模块的任务是帮助高层依赖它的模块将数据包发送到远程主机，然后通知这些模块数据是否到达。
	它自己不实现重传，而是允许每个依赖模块仅仅重传它决定重传的数据。
	这是基于UDP的可靠传输所提供灵活性的主要来源，而TCP并不能提供。

	DeliveryNotificationManager 是上述模块的一种可能实现
	受到星际围攻：部落 Statsiege:Tribes的连接管理器启发。
		1 当传输时，必须唯一标识和标记每个发送出去的数据包，
		  这样可以将传递状态与每个数据包关联，并将这个状态以一种有意义的方式传递给依赖模块。
		2 在接收端，必须检查传入的数据包，并针对每个它决定处理的数据包发送一个确认。
		3 回到发送端，必须处理传入的确认，并通知依赖模块哪个数据包被接收了和哪个数据包被丢弃了。

		作为额外的奖励，这个特殊的UDP可靠系统也保证了数据包不会被乱序处理。
		就是说，如果旧的数据包在新数据包之后到达，DeliveryNotificationManager会假装这个数据包被丢弃，并忽略它。
		这是非常有用的，因为它防止了更新的数据包中的新数据被包含在就数据包中的过时数据意外掩盖。
		这有点超出了DeliveryNotificationManager的目的。
		但是在这层实现这个功能是非常常见和有效的。

标记传出的数据包
接收数据包发送并确认
	每个数据包都有一个连续递增的序列化，所以接收主机很容易预测传入数据包中应该包含什么序列号。
		传入的序列号与预期的一致。
		传入的序列号比期望的序列号小。
			丢弃数据包，处理序号到最大值后的回绕。
		传入的序列号比期望的序列号大。
			一个或多个数据包丢失或延迟时就会发生这种情况。
			还是应该处理这个数据包并确认。
			和TCP不同，这里仅仅承诺不乱序处理，并且丢失时发出报告。

	收到数据包不直接发送任何确认，这样做是为了提高效率。
	即使是TCP，也允许每个一个数据包才确认一次。
	在多人游戏中，在客户端给服务器发送反馈数据之前，服务器可能需要给客户端发送几个MTU大小的数据包。
	在这种情况下，最好积累所有必要的确认，并将它们放入客户端要发送给服务器的下一个数据包中。
	多确认：通常只有一个待发送的确认，数据包丢失的情况下，可以写所有待发送的确认。但这增加了数据包大小。
	如果确定游戏中永远不需要一次确认多个数据包，也可以完全删掉多确认系统。
接收确认并传递状态
	一台主机一旦发送了一个数据包，它必须监听并相应地处理确认。
	当预期的确认到达，通知适当的依赖模块发送成功。
	当预期的确认没有到达，通知适当的依赖模块交付失败。
	警告：
		确认的缺失并不真正标识数据包丢失。数据可能已经成功送达，但是包含确认的数据包丢失了。
		原始发送的主机没有办法区分这两种情况。
		再TCP中，这不是问题，因为重传数据包使用与之前发送相同的序列号。
		如果TCP模块收到重复数据包，忽略即可。
		对于DeliveryNotificationManager，不是这样的。因为丢失的数据包不一定被重传，所以每个数据包都唯一标识。
		这意味着客户端模块可能根据丢失的确认来决定重传一些可靠数据，同时接收端可能已经有这些数据了。
		这种情况下，由依赖模块唯一地识别数据本身，防止重复。
		例如：如果ExplosionManager依赖DeliveryNotificationManager发送爆炸数据，它应该唯一标识这个爆炸。
	你可能想知道之前被报告为丢失的数据包之后是如何被确认的。
		如果确认花费了很长时间才到达，就会发生这种情况。
		正如TCP中确认不及时的情况下重传数据包一样，DeliveryNotificationManager也会超时重传。
		每帧检测超时，跟踪数据包投递率。如果丢失率高，可以通知适当降低传输率。
```

```
对象复制可靠性
	为了改进相对于TCP版本的可靠性，你不需要重传每个丢失的数据，而是仅仅发送丢失数据的最新版本。
	扩展第五章的ReplicationManager，以支持可靠地重传最新数据，这个方法受到星际围攻：部落的ghost管理器的启发。
	为了可靠地发送数据，ReplicationManager知道携带可靠数据的数据包丢失之后，需要能够重传这些数据。
	为了支持这一点，主机应用程序需要定期询问ReplicationManager，提供给它一个待发的数据包，并询问它是否有数据想要写入数据。
	这样每当ReplicatioManager丢失了可靠数据时，它可以向被提供的这个数据包中写任何数据。
	主机可以根据估计的带宽，数据包丢失率，或者其他任何启发信息，选择提供数据包的频率。
	如果ReplicationManager仅在周期性提供待发送数据包时才填充数据，这样每当有改变的数据要复制，不是由游戏系统创建
	一个数据包，而是将改数据通知给ReplicationManager，由它下次有机会写数据。
	这样很好地创建了游戏系统和网络代码之间的另一层抽象。
	游戏代码不再需要关心数据包或网络。它仅仅通知ReplicationManager，ReplicationManager周期性向数据包写入变化。

	考虑三种基本请求：创建，更新，销毁。
		当游戏系统为目标对象发送一个复制命令给ReplicationManager，ReplicatonManager使用这个命令和对象向待发送数据包写入。
		然后它将该复制命令，目标对象的指针和写入的状态作为传输数据存储在相应的InFlightPacket中。
		如果ReplicationManager知道数据包丢了，它可以找到匹配的InFlightPacket，查找最初写数据时使用的命令和对象，
		然后使用同样的命令，对象和状态位向新数据包写数据。
		相比于TCP，这是一个巨大的提升，因为ReplicationManager并不使用原始的，可能过时的数据来重新写数据包，
		而是使用目标对象的最新状态，可能比原始数据包中的状态要新半秒钟。
```

```
根据传输中的数据包优化
	考虑：一辆车在游戏世界中行驶1秒，如果服务器以每秒20次的频率给客户端可靠地发送状态，
	每个数据包将包括汽车行驶的不同位置。
	如果在0.9秒时发送的数据包丢失了，那么可能在200ms后，服务器的ReplicationManager才意识到并尝试重传数据。
	到那个时候，汽车已经停止了。
	因为在汽车行驶时服务器在不断发送更新，所以已经有包含汽车新位置的新数据包再给客户端发送的途中。
	所以当最新数据已经在发送的途中时，服务器重传当前的位置是浪费的。
	如果有其他方式让ReplicationManager知道途中的数据，那么它就能避免发送多余的状态。

	当ReplicationManager首先知道丢失的数据时，遍历缓存找到对象和属性，如果状态数据已经在传输途中，
	那么便不需要重传了。

	这个优化需要相当多的处理。但是，鉴于通常情况下丢包是低频率的，同时带宽往往比处理能力宝贵，所以这样做有利。
```

```
模拟真实世界的条件
	创建一个测试环境来适当地模拟延迟，抖动和丢包非常重要。
	为了更精确模拟，考虑加入这一事实：数据包丢失或延迟往往相继发生。
	当随机决定一个数据包应该丢弃时，你可以使用另一个随机数来决定它影响了后面多少个数据包。
```

---

# 第八章 改进的延迟处理

```
作为一个多人游戏开发者，延迟是你的天敌。

沉默的客户端 dumb terminal
	谈到CS架构，服务器是唯一拥有真实和正确游戏状态的主机。
	这是所有反欺骗客户端-服务器设置的一个传统需求：服务器是唯一运行最重要模拟的主机。
	着意味着一个玩家产生动作到这个玩家观察到该动作导致的真实游戏状态，总有一些延迟。
	如果玩家观察的仅仅是服务器复制给客户端的真实模拟状态，那么玩家对游戏世界状态的感知至少比
	服务器的真实世界状态晚半个往返时间。
	根据网络流量，物理距离和中间硬件不同，这个时间可以高达100ms或者更多。

	尽管输入和响应之间有明显延迟，早期多人游戏就仅仅用这种实现方式。
	这些游戏中的客户端被称为 沉默的终端，因为它们不需要对模拟有任何了解，唯一的目的就是发送输入，接收结果，显示给用户。
	因为它们仅有服务器发出的状态，所以绝对不会显示给用户错误的状态。尽管有延迟，但是所有状态都是那个时间附近正确的状态。
	这种网络方法被称为保守算法 conservative algorithm

	除了延迟，还有另一个问题。
	鉴于高性能GPU，客户端A能60fps运行，服务器也每秒60帧运行。
	但是由于服务器和客户端A之间的连接宽带限制，服务器只能以每秒15次的频率发送状态。
	如果客户端只听服务器的，意味着有连续4帧，都是一样的。即使GPU很强，也没有用，玩家很不愉快。

	第三个问题，FPS中很难瞄准，敌人的位置是100ms之前的。令人沮丧。
```

```
客户端插值 client side interpolation
	当使用客户端插值时，客户端游戏不是自动将对象移动到服务器发送来的新位置，而是每当客户端收到一个对象的新状态时，
	它使用被称为本地感知过滤器 local perception filter的方法根据时间平滑地插值到这个状态。

	让IP表示以毫秒为单位的插值周期 interpolation period，即客户端从旧状态插值到新状态需要的时间。
	让PP表示以毫秒为单位的数据包周期，即服务器在发送两个数据包之间需要等待的时间。
	在数据包到达之后IP毫秒时，客户端完成到这个数据包状态的插值。
	这一如果IP小于PP，那么客户端在新数据包到达之前将停止插值，玩家仍然会感觉到卡顿。
	为了保证客户端的状态每帧都平滑地变化，插值不应该停止，则IP不能小于PP。
	通过这种方式，每当客户端完成插值到一个给定状态，它都已经接收到了下一个状态，并再一次启动这个过程。

	记住没有插值的沉默终端始终比服务器滞后半个RTT。
	如果状态到达了，但是客户端没有马上显示这个状态，那么在玩家看来游戏会更加滞后。
	使用客户端插值的游戏给玩家展示的状态比服务器上的真实状态滞后大约1/2RTT+IP毫秒。
	这样，为了最小化延迟，IP应该尽可能小。考虑到为了避免卡顿，IP必须大于等于PP。
	这意味着IP应该正好等于PP。

	服务器可以通知客户端它打算发送数据包的频率，或者客户端凭经验根据数据包到达的频率计算PP。
	注意，服务器应该根据带宽，而不是延迟， 来设置数据包周期。
	服务器可以根据它认为的客户端和服务器之间的网络情况来以尽可能高的频率发送数据包。
	这意味着使用客户端插值方式的游戏玩家感知到的延迟时网络延迟和网络带宽综合的一个因素。

	继续之前的例子，如果服务器每秒发送15个数据包，数据包周期是66.7ms。
	这意味着在已经50ms的1/2RTT上又加了66.7ms。
	但是，带有插值的游戏更平滑，延迟就没那么重要了。

	这里，允许玩家操纵摄像机的游戏有一个潜在优势，就是帮助减少额外的延迟感。
	如果摄像机的姿势对于游戏模拟不重要，那么游戏可以在客户端完全处理它。
	走动和射击需要客户端与服务器进行一次往返交换，因为它们直接影响游戏模拟。
	摄像机不影响游戏模拟，不需要等待服务器响应。可以给玩家实时反馈。

	客户端插值仍然是一个保守算法，尽管它有时表示的状态不完全是服务器复制过来的，
	仅仅是服务器真正模拟的两个状态之间的状态。
	客户端平滑状态之间的变换，但是从来没有猜测服务器正在做什么，因此不会得到一个错的离谱的状态。
```

```
客户端预测 client side prediction
	客户端插值即使是微小的插值周期，玩家看到的状态仍然至少滞后半个RTT。
	为了展示更近的游戏状态，你的游戏需要从插值转为推测。
	通过推测法，客户端可以接收略旧的状态，并在显示给玩家之前推测近似的最新状态。
	这种推测技术通常被称为 客户端预测

	为了推测当前的状态，客户端必须能运行与服务器相同的模拟代码。
	当客户端收到一个状态更新，它知道该更新的是1/2RTT之前的。
	为了使状态更近，客户端只需运行额外1/2RTT的模拟。
	接着，当客户端给玩家显示结果时，就会更接近服务器当前模拟的真正游戏状态。
	为了保持这种近似，客户端继续每帧运行模拟，并将结果显示给玩家。
	最终，客户端收到来自服务器的下一个状态数据包，内部运行额外1/2RTT的模拟得到新状态。
	此刻理想情况是该新状态与客户端根据上一次接收状态计算得到的当前的状态完全一致。

	为了执行1/2RTT的推测，客户端必须首先能够粗略估计RTT。
	因为服务器和客户端的时钟不一定同步，最简单的方法，即服务器给数据包打上时间戳，
	然后客户端检查时间戳的年龄，是不可行的。
	相反，客户端必须计算整个RTT，然后除以2.

	客户端给服务器发送一个包含基于客户端本地时钟的时间戳的数据包。
	接收到这个数据包时，服务器复制该时间戳到新的数据包，并发送回客户端。
	当客户端收到这个新数据包时，它根据自己的时钟，从当前的事件中减去旧的时间戳。
	这样就得到了客户端首次发送数据包到收到响应之间的确切时间——RTT的定义。

	对于每个到达的输入数据包，服务器调用HandleInputPacket
	针对鼠标中的每个移动，HandleInputPacket调用移动列表的AddMoveIfNew
	AddMoveIfNew检查每个移动的时间戳，看是否比最近收到的移动更新。
	如果是，将该移动添加到移动列表中，并更新列表的最新时间戳。
	一旦AddMoveIfNew添加了移动，HandleInputPacket便将最新的时间戳标记为脏，
	这样NetworkManage就知道该时间戳应该发送给客户端。当NetworkManager需要给客户端
	发送数据包时，它检查该客户端的时间戳是否为脏。
	如果是，它将移动列表缓存的时间戳写入数据包。
	当另一端的客户端收到该时间戳，它从自己的当前时间中减去该时间戳，得到了从它
	给服务器发送它的输入到收到对应响应的确切时间度量。
```

```
航位推测法 dead reckoning
	客户端没有办法知道远程玩家将发起什么行为。这种情况下，客户端最好的解决方案是先做一个经过训练的猜测，
	然后当来自服务器的更新到达时，如果必要就更正该猜测。

	在网络游戏中，航位预测法是基于实体继续做当前正在做的事情这一假设，进行实体行为预测的过程。
	如果这是一个奔跑的玩家，意味着假设玩家会保持相同方向奔跑。
	如果这是一架转弯的飞机，意味着假设它会继续转弯。

	当被模拟的对象被玩家控制时，航位推测需要运行与服务器相同的模拟，但是模拟过程中玩家的输入是不变的。
	这意味着除了复制被玩家控制的对象外，为了计算将来的位置，服务器必须复制用于模拟的所有变量。
	这包括速度，加速度，跳跃状态，或者更多。取决于你的游戏细节。

	只要远程玩家不断地做当前正在做的事情，航位推测就能保证客户端游戏能够准确预测当前服务器上的真实世界状态。
	但是，当远程玩家采取了意外行动，客户端模拟就会与真实状态产生分歧，必须被纠正。
	因为航位预测法并没有获取所有数据，而是对服务器上的行为做了假设，所以航位模拟被认为是不保守的算法。
	称为乐观算法optimistic algorithm
	它希望做到最好，能才对大部分情况，但是有时是完全错误的，必须纠正。

	假设RTT为100ms，帧率是60fps。
	在50ms时，客户端A收到消息：玩家B在位置(0,0)，正沿着X轴的正方向以每毫秒1个单位的速度奔跑。
	因为该状态滞后1/2RTT，所以它模拟玩家B继续以该速度奔跑50ms，然后显示玩家B的位置为(50,50)
	然后，在等待下一个状态数据包的四帧中，客户端A继续每帧模拟玩家B的奔跑。
	在第4帧，即117ms时，它已经预测玩家B应该在(117,0)
	接着，客户端A收到来自服务器的数据包，得到玩家B的速度是(1,0)，位置是(67,0)
	客户端继续向前模拟1/2RTT，并发现该位置与预期的位置一致。

	一切都很好，客户端A继续模拟下一个4帧，预测玩家B的位置是(184,0)
	但是，在该时刻，它收到来自服务器的新状态，指示玩家B的位置是(134,0)，速度变为(0,1)
	玩家B很有可能停止向前跑，并开始扫射。
	向前模拟1/2RTT得到位置(134,50)，根本不是客户端航位推测之前所预测的结果。
	玩家B发生了意想不到的，不可预知的行为。
	正因为如此，客户端A的本地预测与真实的世界状态发生了分歧。

	当客户端检测到它的本地模拟发生错误时，有三种方式来弥补：
		即时状态更新 instant state update
			只需立即更新到最新状态。
			玩家可能发现对象跳来跳去，但这样也许也好过错误的数据。
			记住即使是即时更新，来自服务器的状态仍然滞后1/2RTT，
			所以客户端应该使用航位推测和最新的状态来模拟额外的1/2RTT
		插值 interpolation
			从客户端插值的方法可以看到，你的游戏可以在一定数量的帧内平滑地插值到新状态。
			这意味着对于每个错误状态（位置，旋转等）都要计算和存储一个偏移量，
			用于每一帧。或者只将对象移动一部分路程，使其更接近正确位置，等待将来的服务器状态继续进行纠正。
			一种流行的方法是使用三次样条插值创建路径，以实现位置和速度同时平滑地从预测状态过渡到正确状态。
		二阶状态调整 second-order state adjustment
			如果一个几乎静止的对象突然加速，即使插值也可能发生抖动。
			为了更精细地处理，你的游戏可以调整二阶参数，例如加速度，非常平缓地对模拟进行同步修正。
			这在教学上有些复杂，但是可以使得纠正最不明显。
	通常游戏继续差异的幅度和游戏特性，将使用这些方法的组合。
	快节奏的射击游戏通常为小错误使用插值，为大错误使用瞬间移动。
	慢节奏的游戏，如飞机模拟或巨型机器人争霸，可能使用二阶状态调整处理除了最大错误之外的所有错误。

	航位推测对于远程玩家非常有效，因为本地玩家实际上并不知道远程玩家在做什么。
	当玩家A看到玩家B的虚拟人跑过屏幕，每次玩家B改变方向，模拟都会发生分歧。
	但是玩家A很难察觉到这一点。
	除非玩家B在同一个房间，玩家A实际上并不知道玩家B什么时候改变的输入。
	在大多数情况下， 他看到的模拟是一致的，即使客户端应用程序总在服务器告知状态的基础上向前猜测至少1/2RTT
```

```
客户端移动预测和重放
	航位推测不能为本地玩家隐藏延迟。
	考虑以下情况，客户端A的玩家开始向前跑。
	航位推测使用服务器发送过来的状态进行模拟，所以从玩家A发起动作，需要1/2RTT将输入传给服务器，然后服务器调整它的速度。
	然后需要1/2RTT将该速度返回给客户端A，这时游戏可以使用航位推测推断最新状态。
	在玩家按下按钮到玩家看到结果仍然存在RTT的延迟。

	有一个更好的办法。玩家A将她发起的所有输入直接给客户端A，所以客户端A的游戏可以使用这些输入模拟她的虚拟人。
	只要玩家A按下按钮开始向前跑，客户端就开始模拟向前跑。当输入数据包到达服务器，服务器也开始模拟，相应地更新玩家A的状态。

	但并非一切都那么简单。当服务器给客户端A发送包含玩家A的复制状态时，问题出现了。
	记得当使用客户端预测时，所有的传入状态应该被模拟额外1/2RTT以赶上真实世界状态。
	当模拟远程玩家时，客户端可以假设输入没有变化，仅仅使用航位推测，来更新状态。
	通常情况下，更新的传入状态与客户端已经预测的状态一致，如果不一致，
	客户端可以通过插值方法将远程玩家平滑地过渡。
	该方法对于本地玩家不可行。
	本地玩家知道他们在哪儿，会注意到插值。当他们改变输入时，他们不能容忍漂移和平滑。
	理想情况下，走动对于本地玩家的感觉应该是它在玩单机游戏，而不是网络游戏。

	一个可能解决方案是：
	对于本地玩家，完全忽略服务器的状态。客户端A只从本地模拟得到玩家A的状态，
	玩家A将有一个平滑的移动体验， 没有延迟。
	不幸的是，这将导致玩家A的状态与服务器的真实状态产生分歧。
	如果玩家B碰到了玩家A，客户端A没办法准确地预测服务器的碰撞结果。
	只有服务器知道玩家B的真实位置。
	客户端A只有玩家B的航位推测近似值，所以不会与服务器采用完全相同的方式解决碰撞。
	玩家A可能在服务器上因掉入火坑而死亡，而在客户端上毫发无伤，这非常混乱。
	因为客户端A忽略了玩家A的所有传入状态，所以客户端和服务器没有办法保持同步。

	一个更好的解决方案：
	当客户端A收到来自服务器的玩家A的状态，客户端A可以使用玩家A的输入重新模拟（重放）
	从服务器计算该传入状态起玩家A发起的所有状态改变。
	客户端不是使用航位推测模拟1/2RTT，而是使用玩家A的精确输入来模拟1/2RTT
	通过引入移动move的概念，输入状态与时间戳关联在一起，客户端可以指出在计算该状态时
	服务器还没收到哪些移动，然后本地应用这些移动。
	除非遇到一个意外的，远程玩家发起的事件，客户端的预测状态将于服务器保持一致。

	为了让RoboCatAction支持移动重放，客户端要做的第一步是在移动列表中保存这些移动。
	直到服务器将它们用于模拟状态。
	SendInputPacket不再一发送数据包，就立刻清空移动列表。
	相反，它会保存这些移动，这样可以用于在收到服务器状态之后的移动重放。
	因为现在移动持续的时间超过一个数据包，所以作为额外的好处，客户端发送移动列表中三个最近的移动。
	这样，如果任何一个数据包在发往服务器的途中丢失了，它还有两次机会能够到达。
	这不能保证可靠性，但是显著地增加了可能性。

	当客户端收到状态数据包，使用ReadLastMoveProcessedOnServerTimestamp处理服务器可能返回的所有移动时间戳。
	如果发现了时间戳，它从当前时间中减去该时间戳，计算得到RTT，用于航位推测。
	然后调用RemovedProcessedMoves删除该时间戳以及之前的移动。
	这意味着在ReadLastMoveProcessedOnServerTimestamp执行完成后，客户端本地移动列表只包含服务器还没有看到的移动。
	因此应该用于来自服务器的任何传入状态。

	重放移动：
	Read方法开始时存储对象的当前状态，这样如果后续任何地方需要平滑调整，它可以知道。然后如前面章节所描述的。
	通过从数据包中读取状态来更新状态。更新之后，应用客户端预测将复制状态向前模拟1/2RTT。
	如果复制的对象是被本地玩家控制的，那么调用DoClientSidePredictionAfterReplicationForLocalCat运行移动重放。
	否则，调用DoClientSidePredictionAfterReplicationForRemoteCat运行航位推测。

	DoClientSidePredictionAfterReplicationForLocalCat
	首先执行检查来保证位置信息以及被复制了。如果没有，则不需要向前模拟。
	如果有位置信息，该方法遍历移动列表中的所有剩余移动，并将它们用于本地的RoboCat
	这模拟了服务器还没有纳入到自己模拟中的所有玩家行为。
	如果服务器上没有意外情况发生，那么该函数得到的本地猫状态应该严格是调用Read方法之前的准确状态。

	DoClientSidePredictionAfterReplicationForRemoteCat
	使用猫的最新状态向前模拟。这包括调用SimulateMovement，在没有任何ProcessInput调用的情况下模拟合适的时间。
	同样，如果服务器上没有意外情况发生，那么计算的状态与Read方法开始之前的准确状态也应该一致。
	但是，与本地猫不同，很可能发生意外，远程玩家总时执行一些诸如改变方向，加速或者减速等动作。

	执行客户端预测之后，Read方法最后调用InterpolateClientSidePrediction来处理任何可能已经改变的状态。
	通过传入旧状态，插值方法可以决定在需要的时候如何从旧状态平滑过渡到新状态。
```

```
通过技巧和优化隐藏延迟
	对于玩家来说，移动的延迟并不是延迟的唯一指示。
	当玩家按下按钮来射击，它希望枪立刻被触发。当它尝试释放一个攻击咒语，它希望虚拟人马上扔一个大火球。
	移动重放并不处理这种情形，所以需要其他方法。
	如果让客户端像服务器一样创建抛射体并接管其状态就太复杂了——有个更简单的办法。

	几乎所有的视频动作游戏都有通知tell，或者视觉线索来指示事情将要发生。
	在血浆喷射前枪口闪烁，在喷射火焰前法师挥动双手并嘴里嘟囔。
	这些通知持续至少服务器与客户端通信一个来回的时间。
	乐观地将，这意味着客户端可以通过在本地执行适当的模拟和特效给玩家任何输入提供及时反馈，
	同时等待服务器更新真实模拟。
	这并不意味着客户端产生抛射物，但是它可以开始播放释放动画和声音。
	如果一切顺利，在施法过程中，服务器接收输入数据包，产生火球，并将它复制给客户端，正赶上显示施法的结果。

	航位推测代码向前模拟1/2RTT的抛射动作，玩家看起来好像它在扔火球，没有延迟。
	如果发生问题，例如，服务器知道该玩家最近被沉默，但尚未将这个信息通知给玩家。
	那么该优化就失去了意义，施法动画开始了但是没有出现抛射物。
	这是一种罕见的情况，但和这种方法提供的好处相比，是可以容忍的。
```

```
服务器端回退
	仍然有一种常见的游戏动作是客户端预测不能很好处理的：长距离的即时射击。
	当玩家匹配狙击步枪，准确瞄准另一位玩家，扣动扳机，它希望有一次完美命中。
	但是，由于航位推测的不准确性，客户端上完美的瞄准射击对于服务器可能不准。
	这对于依赖实时，即时的射击武器的游戏来说，是一个问题。

	有一个解决方案，被Valve的SourceEngine推广流行，被Counter-Strike反恐精英之类的游戏采用。
	来给玩家带来准确无误的射击体验。
	它的核心是：当瞄准和开火时，让服务器状态回退到玩家感受到的那个状态。
	这样，如果玩家感觉它瞄的很准，那么它百分百能打中。

	为了实现这个技巧，游戏必须在客户端预测的基础上做一些修改。
	1 远程玩家使用客户端插值，而不是航位推测。
		服务器需要准确知道客户端玩家每个时刻看到了什么。
		因为航位推测依赖客户端基于假设的向前模拟，将给服务器带来额外的复杂度，因此不应该开启。
		为了避免数据包之间的抖动或卡顿，客户端转而使用本章前面介绍的客户端插值方法。
		插值周期应该精确等于数据包周期，这一周期被服务器牢牢控制。
		客户端插值引入了额外的延迟，但是鉴于移动重放和服务器端回退算法，它不会被玩家明显感觉到。
	2 使用本地客户端移动预测和移动重放
		尽管客户端预测对远程玩家是关闭的，但是对本地玩家仍然是保留的。
		没有本地移动预测和移动重放，本地玩家会立即注意到来自网络和客户端插值所增加的延迟。
		但是，通过即时模拟玩家移动，不管存在多少延迟，玩家都不会感觉到。
	3 发送给服务器的每个移动数据包中保存客户端视角
		客户端应该在每个发送的数据包中记录客户端当前插值的两个帧的ID，以及插值进度百分比。
		这给服务器提供了客户端当时所感知世界的精确指示。
	4 在服务端，存储每个相关对象最近几帧的位置
		当传入的客户端输入数据包中包含射击时，查找在射击时刻用于插值的两帧。
		使用数据包中的插值进度百分比将所有相关对象回退到客户端扣动扳机的那一刻。
		然后从客户端的位置采用光线投射法来确定是否击中。

	服务端回退保证如果客户端准确瞄准了，那么在服务器端一定会被击中。
	这给玩家带来了很满意的体验。
	但是，仍然存在一些不足。因为服务器回退的时间时根据服务器和客户端之间的延迟决定的，对于被击中的玩家
	会造成一些意想不到和令人沮丧的体验。
	玩家A可能以为自己已经安全地躲在了角落里，躲开了玩家B。
	但是，如果玩家B的网络延迟很大，它看到的世界比玩家A之后了300ms。
	这样，在他的计算机上，玩家A还没有躲到角落里。如果玩家B瞄准并开火，服务器将判断为击中。
	并通知玩家A被击中，即时它认为自己已经安全地躲在角落。
	对于游戏开发来说，这是一个权衡问题，需要根据游戏特性来决定是否使用这些技术。
```

---

# 第九章 可扩展性

```
扩大网络游戏的规模会引入小规模游戏中不存在的挑战
```

---

# 第十章 安全性

```
自从第一个网游出现，玩家已经设计出获得不公平优势的方法。
随着网络游戏越来越流行，抵制安全漏洞已经成为为所有玩家提供安全和有趣环境的重要部分。
```

---

# 第十一章 真实世界的引擎

```
对于大部分类型的网络游戏，小工作室使用现成的引擎在时间和成本上更合算。
这种情况下，网络工程师要写的代码处于比本书中大部分内容更高的逻辑层次。
```

```
虚幻引擎4
	使用Unreal的开发者一般不必关心底层的网络细节。
	相反，开发者关心的是更高层次的游戏代码，并确保其在网络环境下正常工作。

套接字和基本的网络体系
	为了给众多平台提供支持，Unreal必须从底层套接字抽象实现细节。
	ISocketSubsystem的接口类已经为Unreal的不同平台做了实现。
	Create函数返回创建的FSocket类的指针，然后就可以调用标准函数Send，Recv等
	UNetDriver类负责接收，过滤，处理和发送数据包。

游戏对象和拓扑
	Unreal使用一些相当特别的术语来表示该引擎中的关键游戏类。
	Actor几乎是所有游戏对象的基类。
	Pawn是可以被控制的Actor，这意味着Actor有一个指向Controller实例的指针。
	Controller也是Actor的子类，是因为Controller仍然是需要被更新的游戏对象。
	Controller应该是PlayerController或者AIControler取决于正在控制的Pawn。

	对于网络游戏，Unreal只支持客户端-服务器模型。
	服务器可以以两种不同的模式运行：专用服务器和监听服务器。
	在专用服务器dedicated server中，服务器作为与其他所有客户端分开的一个进程运行。
	通常，专用服务器完全运行在一个单独的机器上，尽管这不是必须的。
	在监听服务器listen server模式下，一个游戏实例既是服务器也是一个客户端。
	专用服务器和监听服务器的差异超出了本节范围。

Actor复制
	Unreal使用CS模型，需要有一种方法让服务器给所有客户端发送actor更新。
	可以恰当地成为actor复制actor replication
	Unreal做了一些不同的事情来减少需要在任何一个时间点被复制的actor的数量。
	Unreal尝试确定与任何一个客户端相关的actor集合。
	此外，如果有一个actor只会和一个特定客户端相关，那么可以在那个客户端上生成这个actor，
	而不需要在服务器上生成。
	一个使用第二种方法的例子是actor用于封装临时粒子的效果。
	此外，也可以使用不同的标识来进一步调整actor的相关性。
	例如，bAlwaysRelevant将大大增加actor是相关的可能性。

	相关性导致了下一个重要概念——角色 role
	在网络多人游戏中，会一次运行几个独立的游戏实例。
	每个实例可以查询每个actor的角色，这是为了确定谁具有该actor的控制权。
	根据查询角色的游戏实例不同，特定的actor的角色也可以不同，理解这一点很重要。

	在躲避球的多人游戏版本中，球是在服务器上生成的。
	那么，如果服务器查询球的角色，那么它将看到其具有权威角色，意味着服务器是球actor的最终权威。
	但是，其他客户端将看到模拟代理的角色，意思是它们只是简单模拟球，但不是球行为的权威。
		权威authority
			游戏实例是actor的权威
		模拟代理 simulated proxy
			在客户端上，这意味着服务器是actor的权威
			模拟代理意味着客户端可以模拟actor的某些方面，例如移动
		自治代理 autonomous proxy
			自治代理与模拟代理非常类似，尽管意味着它是直接从当前的游戏实例接收输入事件的代理，
			所以当代理在模拟时，应该考虑玩家的输入。
	这并不意味着在多人游戏中，服务器总是每个actor的权威。
	在本地粒子特效actor的例子中，让客户端生成actor是很有意义的，这里客户端是角色的权威，
	并且服务器甚至不知道粒子效果actor的存在。
	但是，每个以服务器作为角色权威的actor都将被复制给所有相关的客户端。
	在这些actor里面，可以指定哪些属性应该被复制，哪些属性不应该被复制。
	通过这种方法，可以仅仅复制对正确模拟actor关键的属性，以节省带宽。
	Unreal中的actor复制仅仅是从服务器到客户端，客户端没有办法创建一个actor，并将其赋值给服务器或其他客户端。
	除了复制属性，也可以有更先进的复制配置。
	例如，可以根据特定条件仅仅复制一个属性。
	也可以每当一个特定的属性从服务器复制过来时，就在客户端上执行自定义的函数。
	因为虚幻引擎4的游戏代码是C++的，所以该引擎使用了复杂的宏集合来跟踪所有不同的复制属性。
	所以，当在类的头文件中增加一个变量时，也可以通过宏在变量上标注合适的复制信息。
	Unreal蓝图Blueprint相当强大，大部分的多人游戏功能也可以通过蓝图访问。
	为了方便，Unreal已经实现了actor移动的客户端预测。
	具体而言，如果一个actor被设置bReplicateMovement标识，那么它将根据复制过来的速度信息，
	复制并预测模拟代理的移动。
	如果需要的话，也可以重写该方法为角色移动实现客户端预测。
	但是。默认的实现给大多数游戏提供了良好起点。

远程过程调用
	Unreal提供了相当强大的远程过程调用系统。
	有三种类型的RPC：服务器，客户端，多播

	服务器函数 server function
		是在客户端上调用并在服务器上执行的函数。
		一个重要的提醒是：服务器不会让任何客户端调用游戏世界中的所有actor上的服务器RPC。
		这很容易导致潜在的作弊等问题。
		相反，只有拥有这个actor的客户端可以在actor上成功地执行服务器RPC。
		需要注意的是，actor拥有者和具有权威角色的游戏实例不是一回事。
		事实上，拥有者是与所讨论的actor相关联的PlayerController。
		例如，如果PlayerController A控制PlayerPwan A，
		那么驾驭PlayerController A的客户端被认为是PlayerPawn A的拥有者。
		只有客户端A可以在PlayerPawn A上调用ThrowDodgeBall这一服务器RPC，
		客户端A试图在任何其他PlayerPawn上对ThrowDodgeBall的调用都将被忽略。
	客户端函数 client function
		与服务器函数相反。当服务器调用客户端函数时，这个过程调用被发送给拥有所讨论的actor客户端。
		例如，当服务器确定在躲避球游戏中，玩家C被消灭了，那么服务器在玩家C上调用一个客户端函数，
		这样玩家C的拥有者客户端会在屏幕上显示“被消灭”消息。
	多播函数 multicast function
		将会被发送到多个游戏实例。
		特别地，多播函数是在服务器上调用，并在服务器和所有客户端上执行的函数。
		多播函数用于通知每个客户端特定的事件
		例如：当服务器想让每一个客户端本地生成一个例子效果actor时，就是用多播函数。

	三种不同类型的RPC结合在一起具有很大的灵活性。
	还有一个要注意的是，Unreal提供了RPC是否可靠的选项。
	这意味着对于低优先级的事件可以将它们的RPC标记为不可靠的，这在发生数据包丢失时可以提升性能。
```

```
Unity
	已弃用
```

---

# 第十二章 玩家服务

```
如Steam，Xbox Live或者PlayStation Network
提供了许多功能，包括比赛安排，统计，成就，排行榜，云存储。
```

---

# 第十三章 云托管专用服务器

```
在云端运行游戏服务器的优缺点和必要方法。
```