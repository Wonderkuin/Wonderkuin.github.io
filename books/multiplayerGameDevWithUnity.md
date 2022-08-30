# Unity3D 网络游戏实战

---

# 第二章 坦克控制单元

```c#
// 坦克移动
void Update()
{
	float steer = 20;
	float x = Input.GetAxis("Horizontal");
	transform.Rotate(0, x * steer * Time.deltaTime, 0);

	float speed = 3f;
	float y = Input.GetAxis("Vertical");
	Vector3 s = y * transform.forward * speed * Time.deltaTime;
	transofrm.position += s;
}
```

```c#
//相机跟随
public class CamearaFollow : MonoBehaviro
{
	// 距离
	public flaot distance = 15;
	// 横向角度
	public float rot = 0;
	// 纵向角度
	private flaot roll = 30f * Mathf.PI * 2 / 360;
	// 目标物体
	private GameObject target;

	void Start()
	{
		// 找到坦克
		target = GameObject.Find("tank");
	}

	void LateUpdate()
	{
		// 一些判断
		if (target == null)
			return;
		if (Camera.main == null)
			return;
		// 目标的目标
		Vector3 targetPos = target.transform.position;
		//用三角函数计算相机位置
		Vector3 cameraPos;
		float d = distance * Mathf.Cos(roll);
		float height = distance * Mathf.Sin(roll);
		cameraPos.x = targetPos.x + d * Mathf.Cos(rot);
		cameraPos.z = targetPos.z + d * Mathf.Sin(rot);
		cameraPos.y = target.Pos.y + height;
		Camera.main.transform.position = cameraPos;
		//对准目标
		Camera.main.transform.LookAt(target.transform);
	}
}

// 对准目标
public void SetTarget(GameObjedct target)
{
	if (target.transform.FindChild("cameraPoint") != null)
		this.target = target.transform.FindChild("cameraPoint").gameObject;
	else
		this.target = target;
}

// 横向旋转
public float rotSpeed = 0.2f;
void Rotate()
{
	float w = Input.GetAxis("Mouse X") * rotSpeed;
	rot -= w;
}

// 纵向旋转
private flaot maxRoll = 70f * Mathf.PI * 2 / 360;
private flaot minRoll = -70f * Mathf.PI * 2 / 360;
private float rollSpeed = 0.2f;
void Roll()
{
	float w = Input.GetAxis("Mouse Y") * rollSpee * 0.5f;
	roll -= w;
	if (roll > maxRoll)
		roll = maxRoll;
	if (roll < minRoll)
		roll = minRoll;
}

// 调整距离
public flaot maxDistance = 22f;
public float minDistance = 5f;
public float zoomSpeed = 0.2f;
void Zoom()
{
	if (Input.GetAxis("Mouse ScrollWheel") > 0)
	{
		if (distance > minDistance)
			distance -= zoomSpeed;
	}
	else (Input.GetAxis("Mouse ScrollWheel" < 0)
	{
		if (distance < maxDistance)
			distance += zoomSpeed;
	}
}
```

```c#
// 炮塔旋转 向屏幕中心旋转
// 炮塔
public Transform turret;
// 炮塔旋转角度
public float turretRotSpeed = 0.5f;
// 炮塔目标角度
public float turretRotTarget = 0;

void Start()
{
	turret = transform.FindChild("turret");
}

void Update()
{
	turretRotTarget = Camera.main.transform.eulerAngles.y;
}

public void TurrentRotation()
{
	if (Camera.main == null)
		return;
	if (turret == null)
		return;
	
	// 归一化角度
	float angle = turret.eulerAngles.y - turretRotTarget;
	if (angle < 0)
		angle += 360;
	
	if (angle > turrentRotSpeed && angle < 180)
		turret.Rotate(0f, -turretRotSpeed, 0f);
	else if (angle > 180 && angle < 360 - turretRotSpeed)
		turret.Rotate(0f, turretRotSpeed, 0f);
}
```

```c#
// 炮管旋转
public void TurretRoll()
{
	if (Camera.main == null)
		return;
	if (turret == null)
		return;
	// 获取角度
	Vector3 worldEuler = gun.eulerAngles;
	Vector3 localEuler = gun.localEulerAngle;
	//世界坐标系角度计算
	worldEuler.x = turretRollTarget;
	gun.eulerAngles = world.Euler;
	//本地坐标系角度限制
	Vector3 euler = gun.localEulerAngles;
	if (euler.x > 180)
		euler.x -= 360;
	if (euler.x > maxRoll)
		euler.x = maxRoll;
	if (euler.x < minRoll)
		euler.x = minRoll;
	gun.localEulerAngles = new Vector3(euler.x, localEuler.y, localEuler.z);
}
```

```C#
// 坦克控制
// 使用 WheelCollider
// 
// 当 带有 Rigidbody 和 Collider 的游戏对象
// 碰撞 带有 Collider 的游戏对象 时
// OnCollisionEnter被调用

// 汽车的前轮和后轮分别悬挂在两条轴上
// 每条轴上的两个轮子的步调时一致的

using UntiyEngine;

[System.Serializable]
public class AxleInfo
{
	public WheelCollider leftWheel;
	public WheelCollider rightWheel;
	public bool motor;//是否有马力
	public bool steering;//是否有转向 前轴有，后轴没有
}

// 使用WheelCollider控制移动
// 轮轴
public List<AxleInfo> axleInfos;
// 马力 最大马力
private float moter = 0;
public float maxMoterTorque;
// 制动 最大制动
private float brakeTorque = 0;
public flaot maxBrakeTorque = 100;
// 转向角 最大转向角
private float steering = 0;
public float maxSteeringAngle;

// 玩家控制
public void PlayerCtrl()
{
	// 马力和转向角
	motor = maxMotorTorque * Input.GetAxis("Vertical");
	steering = maxSteeringAngle * Input.GetAxis("Horizontal");
	// 炮塔炮管角度
	turretRotTarget = Camera.main.transform.eulerAngles.y;
	turretRollTarget = Camera.main.transform.eulerAngles.x;

	// 刹车
	brakeTorque = 0;
	foreach (AxleInfo axleInfo in axleInfos)
	{
		if (axleInfo.leftWheel.rpm > 5 && motor < 0)//前进时，按下 下 键
			brakeTorque = maxBrakeTorque;
		else if (exleInfo.leftWheel.rpm < -5 && moter > 0)//后退时，按下 上 键
			brakeTorque = maxBrakeTorque;
	}
}

// 每帧执行一次
void Update()
{
	// 玩家控制操控
	PlayerCtrl();

	// 遍历车轴
	foreach (AxleInfo axleInfo in axleInfos)
	{
		// 转向
		if (axleInfo.steering)
		{
			axleInfo.leftWheel.steerAngle = steering;
			axleInfo.rightWHeel.steerAngle = steering;
		}
		// 马力
		if (axleInfo.motor)
		{
			axleInfo.leftWheel.brakeTorque = brakeTorque;
			axleInfo.rightWheel.brakeTorque = brakeTorque;
		}
	}
	// 炮塔炮管旋转
	TurretRotation();
	TurretRoll();
}
```

```c#
// 轮子旋转

public void WheelsRoatation(WheelCollider collider)
{
	if (wheels == null)
		return;
	// 获取旋转信息
	Vector3 position;
	Quaternion rotation;
	collider.GetWorldPose(out position, out rotation);
	// 旋转每个轮子
	foreach (Transform wheel in wheels)
	{
		wheel.rotation = rotation;
	}
}

void Update()
{
	foreach (AxleInfo axleInfo in axleInfos)
	{
		if (axleInfos[1] != null)
			WheelsRotation(axleInfos[1].leftWheel);
	}
}


// 履带滚动
// 改变贴图的偏移值 MainMaps.Offset.Y
private Transform tracks;
void Start()
{
	tracks = transform.FindChild("tracks");
}

public void TrackMove()
{
	if (tracks == null)
		return;

	float offset = 0;
	if (wheels.GetChild(0) != null)
		offset = wheels.GetChild(0).localEulerAngles.x / 90f;
	
	foreach (Transform track in tracks)
	{
		MeshRenderer mr = track.gameObject.GetComponent<MeshRenderer>();
		if (mr == null)
			continue;
		Material mtl = mr.material;
		mtl.mainTextureOffset = new Vector2(0, offset);
	}
}

void Update()
{
	WheelsRotation(axleInfos[1].leftWheel);
	TrackMove();
}
```

# 火炮与敌人

```c#
using UnityEngine;
using System.Collections;

public class Bullet : MonoBehaviour
{
	public flaot speed = 10f;
	public GameObject explode;
	public float maxLifeTime = 2f;
	public float instantiateTime = 0f;

	void Start()
	{
		instantiateTime = Time.time;
	}

	void Update()
	{
		// 前进
		transform.position += transform.forward * speed * Time.deltaTime;
		// 摧毁
		if (Time.time - instantiateTime > maxLeftTime)
			Destroy(gameObject);
	}

	// 碰撞
	void OnCollisionEnter(Collision collisionInfo)
	{
		// 爆炸效果
		Instantiate(explode, transform.position, transform.rotation);
		// 摧毁自身
		Destroy(gameObject);
	}
}
```

```c#
// 坦克开炮

// 炮弹预设
public GameObject bullet;
// 上一次开炮时间
public float lastShootTime = 0f;
// 开炮时间间隔
private float shootInterval = 0.5f;

public void Shoot()
{
	// 发射间隔
	if (Time.time - lastShootTime < shootInterval)
		return;
	//子弹
	if (bullet == null)
		return;
	// 发射
	Vector3 pos = gun.position + gun.forward * 5;
	Instantiate(bullet, pos, gun.rotation);
	lastShootTime = Time.time;
}
```

```c#
// 准星

// 计算目标角度
public void TargetSignPos()
{
	// 碰撞信息和碰撞点
	Vector3 hitPoint = Vector3.zero;
	RaycastHit raycastHit;
	// 屏幕中心位置
	Vector3 centerVec = new Vector3(Screen.width / 2, Screen.height / 2, 0);
	Ray ray = Camera.main.ScreenPointToRay(centerVec);
	// 射线检测，获取hitPoint
	if (Physics.Raycast(ray, out raycastHit, 400.0f))
	{
		hitPoint = raycastHit.point;
	}
	else
	{
		hitPoint = ray.GetPoint(400);
	}
	// 计算目标角度
	Vector3 dir = hitPoint - turret.position;
	Quaternion angle = Quaternion.LookRotation(dir);
	turretRotTarget = angle.eulerAngles.y;
	turretRollTarget = angle.eulerAngles.x;
	// 调试用，稍后删除
	Transform targetCube = GameObject.Find("TargetCube").transform;
	targetCube.position = hitPoint;
}
```

# AI

```c#
// NavMesh寻路
public void InitByNavMeshPath(Vector3 pos, Vector3 targetPos)
{
	// 重置
	waypoints = null;
	index = -1;
	//计算路径
	NavMeshPath navPath = new NavMeshPath();
	bool hasFoundPath = NavMesh.CalculatePath(pos, targetPos, NavMesh.AllAreas, navPath);
	if (!hasFoundPath)
		return;
	//生成路径
	int length = navPath.corners.Length;
	waypoints = new Vector3[length];
	for (int i = 0; i < length; i++)
		waypoints[i] = navPath.corners[i];
	index = 0;
	waypoint = waypoints[index];
	isFinish = false;
}
// NavMeshObstacle 动态寻路组件
```

# 代码分离的界面系统

```c#
//面板基类
using UnityEngine;
using System.Collections;

public class PanelBase : MonoBehaviour
{
	// 皮肤路径
	public string skinPath;
	// 皮肤
	public GameObject skin;
	// 层级
	public PanelLayer layer;
	// 面板参数
	public object[] args;

	#region 生命周期
	// 初始化
	public virtual void Init(param object[] args)
	{
		this.args = args;
	}
	// 开始面板前
	public virtual void OnShowing() {}
	// 开始面板后
	public virtual void OnShowed() {}
	// 帧更新
	public virtual void Update() {}
	// 关闭前
	public virtual void OnClosing() {}
	// 关闭后
	public virtual void OnClosed() {}
	#endregion

	#region 操作
	protected virtual void Cose()
	{
		string name = this.GetType().ToString();
		PanelMgr.instance.ClosePanel(name);
	}
	#endregion
}
```

```c#
// 面板管理器
using UnityEngine;
using System;
using System.Collections;
using System.Clooections.Generic;

public class PanelMgr : MonoBehavior
{
	// 单例
	public static PanelMgr instance;
	// 画板
	private GameObject canvas;
	// 面板
	public Dictionary<string, PanelBase> dict;
	// 层级
	private Dictionary<PanelLayer, Transform> layerDict;

	// 开始
	public void Awake()
	{
		instance = this;
		InitLayer();
		dict = new Dictionary<string, PanelBase>();
	}
	
	// 初始化层
	private void InitLayer()
	{
		// 画布
		canvas = GameObject.Find("Canvas");
		if (canvas == null)
			Debug.LogError("panelMgr.InitLayer fail, canvas is null");
		// 各个层级
		layerDict = new Dictionary<PanelLayer, Transform>();

		foreach (PanelLayer pl in Enum.GetValues(typeof(PanelLayer)))
		{
			string name = pl.ToString();
			Transform transform = canvas.transform.FindChild(name);
			layerDict.Add(pl, transform);
		}
	}

	// 分层类型
	public enum PanelLayer
	{
		// 面板
		Panel,
		// 提示
		Tips,
	}

	// 打开面板
	public void OpenPanel<T>(string skinPath, params object[] args) where T : PanelBase
	{
		// 已经打开
		string name = typeof(T).ToString();
		if (dict.ContainsKey(name))
			return;
		//面板脚本
		PanelBase panel = canvas.AddComponent<T>();
		panel.Init(args);
		dict.Add(name, panel);
		// 加载皮肤
		skinPath = (skinPath != "" ? skinPath : panel.skinPanel);
		GameObject skin = Resource.Load<GameObject>(skinPath);
		if (skin == null)
			Debug.LogError("panelMgr.OpenPanel fial, skin is null, skinPath = " + skinPath);
		panel.skin = (GameObject)Instantiate(skin);
		// 坐标
		Transform skinTrans = panel.skin.transform;
		PanelLayer layer = panel.layer;
		Transform parent = layerDict[layer];
		skinTrans.SetParent(parnet, false);
		// panel的生命周期
		panel.OnShowing();
		// anm
		panel.OnShowed();
	}
	
	// 关闭面板
	public void closePanel(string name)
	{
		PanelBase panel = (PanelBase)dict[name];
		if (panel == null)
			return;

		panel.OnClosing();
		dict.Remove(name);
		panel.OnClosed();
		GameObject.Destroy(panel.skin);
		Component.Destroy(panel);
	}
}
```

```c#
// 标题面板
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class TitlePanel : PanelBase
{
	private Button startBtn;
	private Button infoBtn;

	#regin 生命周期
	public override void Init(params object[] args)
	{
		base.Init(args);
		skinPath = "TitlePanel";
		layer = PanelLayer.Panel;
	}

	public override void OnShowing()
	{
		base.OnShowing();

		Transform skinTrans = skin.transform;
		startBtn = skinTrans.FindChild("StartBtn").GetComponent<Button>();
		infoBtn = sinTrans.FindChild("InfoBtn").GetComponent<Button>();
		startBtn.onClick.AddListener(OnStartClick);
		infoBtn.onClick.AddListener(OnInfoClick);
	}
	#endregion

	public void OnStartClick()
	{
		// 开始游戏
		Battle.instance.StartTwoCampBattle(2, 2);
		// 关闭
		Close();
	}

	public void OnInfoClick()
	{
		PanelMgr.instance.OpenPanel<InfoPanel>("");
	}
}
```

```c#
// 信息面板
using UnityEngine;
using System.Collections;
using UnityEngien.UI;

public class InfoPanel : PanelBase
{
	private Button closeBtn;

	#region 生命周期
	public override void Init(params object[] args)
	{
		base.Init(args);
		skinPath = "InfoPanel";
		layer = PanelLayer.Panel;
	}

	public override void OnShowing()
	{
		base.OnShowing();

		Transform skintrans = skin.transform;
		closeBtn = skinTrans.FindChild("CloseBtn").GetComponent<Button>();
		closeBtn.onClick.AddListener(OnCloseClick);
	}
	#endregion

	public void OnCloseClick()
	{
		Close();
	}
}
```

# 网络基础

```
System.Net
IPAddress "127.0.0.1"
PEndPoint "127.0.0.1:80"

TCP 建立连接 三次握手
TCP 断开连接 四次挥手


套接字 是支持TCP/IP协议的网络通信的基本操作单元
可以将套接字看作不同主机的进程双向通信的端点
它构成了单个主机内及整个网络间的编程界面

套接字存在于通信域中
通信域是为了处理一般的线程通过套接字通信而引进的一种抽象概念
套接字通常和同一个域中的套接字交换数据
数据交换也可能会穿越域的界限，但这时一定要执行某种解释程序
各种进程使用这个相同的域用Internet协议来进行相互之间的通信

1 开启一个连接之前 需要先完成 Socket 和 Bind 两个步骤
Socket是新建一个套接字 Bind是指定套接字的IP和端口
客户端在调用Connect时会由系统分配端口，因此可以省去Bind

2 服务端通过Listen开启监听，等待客户端接入

3 客户端通过Connect连接服务器，服务端通过Accept接受客户端连接
在connect-accept过程中，操作系统将会进行三次握手

4 客户端和服务端通过write和read发送和接受数据，操作系统将会完成TCP
数据的确认 重发等步骤

5 通过close关闭连接，操作系统将会进行四次挥手

Socket类
using System.Net.Sockets;
```

```c#
// 服务端程序
// 服务器遵照Socket通信的基本流程 先创建Socket，再调用Bind绑定IP地址和端口号
// 之后调用Listen等待客户端连接
// 最后在while循环中调用Accept接收客户端的连接，并回应消息
using Systen;
using System.Net;
using System.Net.Socket;

class MainClass
{
	public static void Main(string[] args)
	{
		Console.WriteLine("Hello World!");
		// Socket
		// 创建了一个套接字，参数 地址族 套接字类型 协议

		// 地址族 指明 使用的时 IPv4 PIv6 这里是IPv4 即InterNetwork IPv6是InterNetworkV6

		// SocketType 套接字类型 Dgram Raw Rdm Seqpacket Stream Unknown
		// Dgram 无连接 不可靠消息 Udp InterNetworkAddressFamily
		// Raw 可以使用Icmp Internet控制消息协议 Igmp Internet组管理协议
		// Rdm 无连接 面向消息 可靠  消息会依次到达，不重复。如果消息丢失，通知发送方 可以与多个对方主机通信
		// Seqpackage 提供排序字节流的面向连接且可靠的双向传输。与单个对方主机通信
		// Stream 可靠，双向，基于连接的字节流 不重复数据 不保留边界。与单个对方主机通信 TCP InterNetworkAddressFamily

		// ProtocolType 指明协议
		// Ggp 网关到网关  Icmp 网际消息控制协议  IcmpV6  Idp Internet数据报协议 Igmp 网际组管理协议
		// IP 网际协议 Internet 数据包交换协议 PARC 通用数据包协议 Raw 原始IP数据包协议 Tcp 传输控制协议
		// Udp 用户数据报协议 Unknown 未知 Unspecified 未指定

		Socket listenfd = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
		// Bind
		// listen.Bind(ipEp) 将 linstenfd套接字绑定IP和端口
		// 示例程序中绑定的是 localhost:1234
		IPAddress ipAdr = IPAddress.Parse("127.0.0.1");
		IPEndPoint ipEp = new IPEndPoint(ipAdr, 1234);
		listenfd.Bind(ipEp);
		// Listen
		// 服务端通过 listenfd.Listen(0) 开启监听 等待客户端连接
		// 参数 backlog 指定队列中 最多可容纳等待接受的连接数 0 表示不限制
		listenfd.Listen(0);
		Console.WriteLine("[服务器]启动成功");
		while (true)
		{
			// Accept
			// 开启监听后，listenfd.Accept()接收了客户端连接
			// 本例中使用的Socket方法都是阻塞方法，当没有客户端连接时，服务器程序会卡在
			// listenfd.Accept()处 而不会往下执行 直到Accept返回一个新客户端的Socket
			// 对于服务器来说 他有一个监听Socket 即listenfd用来Accept户端的连接 
			// 对于每个客户端来说还有一个专门的Socket 即connfd来处理客户端数据
			Socket connfd = linstenfd.Accept();
			Console.WriteLine("[服务器]Accept");
			// Recv
			// 客户端通过connfd.Receive接收客户端数据，Receive也是阻塞方法
			// 没有收到客户端数据时，程序将卡在Receive处，不会向下执行
			// Receive的返回值则指明接收到的数据长度，之后使用Endode将bytes数组
			// 转换成字符串显示在屏幕上
			byte[] readBuff = new byte[1024];
			int count = connfd.Receive(readBuff);
			string str = System.Text.Encoding.UTF8.GetStrign(readBuff, 0, count);
			Console.WriteLine("[服务器接收]" + str);
			// Send
			// 服务器通过connfd.Send发送数据，它接收一个byte[]类型的参数指明要发送的内容
			// Send的返回值指明发送数据的长度 例子中没有使用
			// 服务器程序用Encoding把字符串转换成byte[]数组，然后发送给客户端
			byte[] bytes = System.Text.Encoding.Default.GetBytes("serv echo " + str);
			connfd.Send(bytes);
		}
	}
}
```

```c#
// 客户端程序
// 创建Socket后，客户端通过Connect连接服务器，然后向服务器发送 Hello Unity
// 等待服务器回应，并把服务器回应的字符串显示出来
using System.Collections;
using System.Net;
using System.Net.Sockets;

public class net
{
	public void Connection()
	{
		// Socket
		Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
		// Connect
		string host = "127.0.0.1";
		int port = 1234;
		socket.Connect(host, port);
		Console.WriteLine("客户端地址" + socket.LocalEndPoint.ToString());
		// Send
		byte[] bytes = System.Text.Encoding.Default.GetBytes("Hello Socket");
		socket.Send(bytes);
		// Recv
		byte[] readBuff = new byte[1024];
		int count = socket.Receive(readBuff);
		string str = System.Text.Encoding.UTF8.GetString(readBuff, 0, count);
		DConsole.WriteLine("收到消息" + str);
		// Close
		socket.Close();
	}
}
```

```c#
// 异步Socket程序

// BeginAccept
listenfd.BeginAccept(AcceptCb, null);
private void AcceptCb(IAsyncResult ar)
{
	Socket socket = listenfd.EndAccept(ar);
	// 接收消息
}

// BeginReceive
public IAsyncResult BeginReceive {
	bytes[] buffer,//Byte类型的数组，存储接收的数据
	int offset,//buffer参数中存储数据的未知，从0计数
	int size,//最多接收的字符数量
	SocketFlags socketFlags,//SocketFlags值的按位组合，这里设置为0
	AsyncCallback callback,//回调函数，一个AsyncCallback委托
	object state//一个用户定义对象，其中包含接收操作的相关信息。
			//当操作完成时，此对象被传递进EndReceive委托
}

//Conn (state)
// 服务端要处理多个客户端，需要用一个数组来维护所有的客户端连接。
// 每个客户端都有自己的Socket和缓冲区
using System;
using System.Net;
using System.Net.Sockets;
using System.Collections;
using System.Collections.Generic;

public class Conn
{
	// 常量
	public const int BUFFER_SIZE = 1024;
	// Socket
	public Socket socket;
	// 是否使用
	public bool isUse = false;
	// Buff
	public byte[] readBuff = new byte[BUFFER_SIZE];
	publc int buffCount = 0;
	// 构造函数
	public Conn()
	{
		readBuff = new byte[BUFFER_SIZE];
	}
	// 初始化
	public void Init(Socket socket)
	{
		this.socket = socket;
		isUse = true;
		buffCount = 0;
	}
	// 缓冲区剩余的字节数
	public int BuffRemain()
	{
		return BUFFER_SIZE - buffCount;
	}
	// 获取客户端地址
	public string GetAdderss()
	{
		if (!isUse)
			return "无法获取地址";
		return socket.RemoteEndPoint.ToString();
	}
	// 关闭
	public void Close()
	{
		if (!isUse)
			return;
		Console.WriteLine("[断开连接] + GetAddress());
		socket.Close();
		isUse = false;
	}
}
```

```c#
// 服务端程序主题结构
public class Serv
{
	// 监听套接字
	public Socket listenfd;
	// 客户端连接
	public Conn[] conns;
	// 最大连接数量
	public int maxConn = 50;

	// 获取连接池索引，返回负数表示获取失败
	public int NewIndex()
	{
		if (conns == null)
			return -1;
		for (int i = 0; i < conns.Length; i++)
		{
			if (conns[i] == null)
			{
				conns[i] = new Conn();
				return i;
			}
			else if (conns[i].isUse == false)
			{
				return i;
			}
		}
		return -1;
	}

	// 开启服务器
	public void Start(string host, int port)
	{
		// 连接池
		conns = new Conn[maxConn];
		for (int i = 0; i < maxConn; i++)
		{
			conns[i] = new Conn();
		}
		// Socket
		listenfd = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
		// Bind
		IPAddress ipAdr = IPAddress.Parse(host);
		IPEndPoint ipEp = new IPEndPoint(ipAdr, port);
		listenfd.Bind(ipEp);
		// Listen
		listenfd.Listen(maxConn);
		// Accept
		listenfd.BeginAccept(AccetpCb, null);
		Console.WriteLine("[服务器]启动成功");
	}

	// AcceptCb回调
	// 给新的连接分配 conn
	// 异步接收客户端数据
	// 再次调用BeginAccept实现循环
	private void Acceptcb(IAsyncResult ar)
	{
		try
		{
			Socket socket = listenfd. (ar);
			int index = NewIndex();

			if (index < 0)
			{
				socket.Close();
				Console.Write("[警告]连接已满");
			}
			else
			{
				Conn conn = conns[index];
				conn.Init(socket);
				// 给新的连接分配 conn
				string adr = conn.GetAdress();
				Console.WriteLine("客户端连接[" + adr + "] conn 池Id: " + index);
				// 异步接收客户端数据
				conn.socket.BeginReceive(conn.readBuff, conn.buffCount, conn.BuffRemain(),
					SocketFlags.None, ReceiveCb, conn);
			}
			// 再次调用BeginAccept实现循环
			listenfd.BeginAccept(AcceptCb, null);
		}
		catch(Exception e)
		{
			Console.WriteLine("AcceptCb失败:" + e.Message);
		}
	}

	// ReceiveCb 接收回调
	// 接收并处理消息 因为是多人聊天 服务端收到消息后 要把它转发给所有人
	// 如果收到客户端关闭连接的信号 count == 0 则断开连接
	// 继续调用BeginReceive接收下一个数据
	private void ReceiveCb(IAsyncResult ar)
	{
		Conn conn = (Conn)ar.AsyncState;
		try
		{
			// 获取接收的字节数
			int count = conn.socket.EndReceive(ar);
			// 关闭信号
			if (count <= 0)
			{
				Console.WriteLine("收到[" + conn.GetAdress() + "]断开连接");
				conn.Close();
				return;
			}
			// 数据处理
			string str = System.Text.Encoding.UTF8.GetString(conn.readBuff, 0, count);
			Console.WriteLine("收到[" + conn.GetAdress() + "]数据" + str);
			str = conn.GetAdress() + ":" + str;
			byte[] bytes = System.Text.Encoding.Default.GetBytes(str);
			// 广播
			// 将消息发送给所有正在使用的连接
			for (int i = 0; i < conns.Length; i++)
			{
				if (conns[i] == null)
					continue;
				if (!conns[i].isUse)
					continue;
				Console.WriteLine("将消息转播给" + conns[i].GetAdress());
				conns[i].socket.Send(bytes);
			}
			// 继续接收
			conn.socket.BeginReceive(conn.readBuff, conn.buffCount, conn.BuffRemain()
						, SocketFlags.None, ReceiveCb, conn);
		}
		catch(Exception e)
		{
			Console.WriteLine("收到[" + conn.GetAdress() + "]断开连接");
			conn.Close();
		}
	}
}

// 开启服务端
Serv serv = new Serv();
serv.Start("127.0.0.1", 1234);

while(true)
{
	strign str = Console.ReadLine();
	switch(str)
	{
		case "quit":
			return;
	}
}
```

```c#
// 客户端程序
// 也使用 BeginReceive实现异步接收
// ReceiveCb不在主线程中， 需要主线程更新UI
using System;
using System.Collections;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;

public class net
{
	// Socket和缓冲区
	Socket socket;
	public byte[] readBuff = new byte[1024];

	// 连接
	public void Connetion()
	{
		// Socket
		socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Type);
		// Connect
		string host = "127.0.0.1";
		int port = 1234;
		socket.Connect(host, port);
		Console.WriteLine("客户端地址" + socket.LocalEndPoint.ToString());
		// Recv
		socket.BeginReceive(readBuff, 0, 1024, SocketFlags.None, ReceiveCb, null);
	}

	// 接收回调
	private void ReceiveCb(IAsyncResult ar)
	{
		try
		{
			// count 是接收数据的大小
			int count = socket.EndReceive(ar);
			// 数据处理
			string str = System.Text.Encoding.UTF8.GetString(readBuff, 0, count);
			Console.WriteLine("接收数据" + str);
			// 继续接收
			socket.BeginReceive(readBuff, 0, 1024, SocketFlags.None, ReceiveCb, null);
		}
		catch ( Exception e)
		{
			Console.WriteLine("连接已经断开");
			socket.Close();
		}
	}

	// 发送数据
	public void Send()
	{
		string str = "HAHAHA";
		byte[] bytes = System.Text.Encoding.Default.GetBytes(str);
		try
		{
			socket.Send(bytes);
		}
		catch()
		{}
	}
}
```

```c#
//类的序列化
[Serializable]
class Player
{
	public int coin = 0;
}

using System;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;
using System.IO;

Player player = new Player();
player.coin = 1;

// 序列化
IFormatter formatter = new BinaryFormatter();
Stream stream = new FileStream("data.bin", FileMode.Create, FileAccess.Write, FileShare.None);
formatter.Serialize(stream, player);
stream.Close();

// 反序列化
IFormatter formatter = new BinaryFormatter();
Stream stream = new FileStream("data.bin", FileMode.Open, FileAccess.Read, FileShare.Read);
Player player = (Player) formatter.Deserialize(stream);
stream.Close();
```

```c#
// 定时器
using System;
using system.Timers;

public static void Main(string[] args)
{
	Timer timer = new Timer();
	timer.AutoReset = ture;
	timer.Interval = 1000;
	timer.Elapsed  += new ElapsedEventHandler(Tick);
	timer.Start();
	// 不退出程序
	Console.Read();
}

public static void Tick(object sender, System.Timer.ElapsedEvnetArgs e)
{
	Console.WriteLine("每秒钟执行一次");
}
```

```c#
// 线程互斥
using System;
using System.Timers;
using System.Threading;

class MainClass
{
	static string str = "";

	public static void Main(string[] args)
	{
		Thread t1 = new Thread(Add1);
		t1.Start();
		Thread t2 = new Thread(Add2);
		t2.Start();
		// 等待一段时间
		Thread.Sleep(1000);
		Console.WriteLine(str);
	}

	// 线程1
	public static void Add1()
	{
		lock(str)
		{

		for (int i = 0; i < 20; i++)
		{
			Thread.Sleep(10);
			str += "A";
		}

		}
	}

	// 线程2
	public static void Add2()
	{
		lock(str)
		{

		for (int i = 0; i < 20; i++)
		{
			Thread.Sleep(10);
			str += "B";
		}

		}
	}
}
```

---

# BackUpCode

```c#
using System;

class Sys
{
    public static long GetTimeStamp()
    {
        TimeSpan ts = DateTime.UtcNow - new DateTime(1970, 1, 1, 0, 0, 0, 0);
        return Convert.ToInt64(ts.TotalSeconds);
    }
}
```

```c#
using System;
using System.Net;
using System.Net.Sockets;
using System.Collections;
using System.Collections.Generic;

// 每个客户端都有自己的Socket和缓冲区，Conn表示客户端的连接
public class Conn
{
    // 缓冲区大小
    public const int BUFFER_SIZE = 1024;

    //Socket
    public Socket socket;
    // 是否使用
    public bool isUse = false;

    //读缓冲区
    public byte[] readBuff = new byte[BUFFER_SIZE];
    //当前读缓冲区长度
    public int buffCount = 0;

    // 粘包分包
    public byte[] lenBytes = new byte[sizeof(UInt32)];
    public Int32 msgLength = 0;

    // 心跳时间
    public long lastTickTime = long.MinValue;

    // 初始化
    public void Init(Socket socket)
    {
        this.socket = socket;
        isUse = true;
        buffCount = 0;

        //心跳处理 稍后实现GetTimeStamp方法
        lastTickTime = Sys.GetTimeStamp();
    }

    // 缓冲区剩余的字节数
    public int BuffRemain()
    {
        return BUFFER_SIZE - buffCount;
    }

    // 获取客户端地址
    public string GetAdress()
    {
        if (!isUse)
            return "无法获取地址";
        return socket.RemoteEndPoint.ToString();
    }

    // 关闭
    public void Close()
    {
        if (!isUse)
            return;
        socket.Shutdown(SocketShutdown.Both);
        socket.Close();
        isUse = false;
    }
}
```

```c#
using System;
using System.Net;
using System.Net.Sockets;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

public class Serv
{
    // 监听套接字
    public Socket listenfd;
    // 客户端连接
    public Conn[] conns;
    // 最大连接数
    public int maxConn = 50;

    // 主定时器
    System.Timers.Timer timer = new System.Timers.Timer(1000);
    // 心跳时间
    public long heartBeatTime = 180;

    // 开启服务器
    public void Start(string host, int port)
    {
        // 连接池
        conns = new Conn[maxConn];
        for (int i = 0; i < maxConn; i++)
        {
            conns[i] = new Conn();
        }

        // 定时器 如果主线程阻塞 定时器将不起作用
        timer.Elapsed += new System.Timers.ElapsedEventHandler(HandleMainTimer);
        timer.AutoReset = false;
        timer.Enabled = true;

        //Socket
        listenfd = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        //Bind
        IPAddress ipAdr = IPAddress.Parse(host);
        IPEndPoint ipEd = new IPEndPoint(ipAdr, port);
        listenfd.Bind(ipEd);
        //Listen
        listenfd.Listen(maxConn);
        //Accept
        listenfd.BeginAccept(AcceptCb, null);
        Console.WriteLine("服务器启动成功");
    }

    // 主定时器
    private void HandleMainTimer(object sender, System.Timers.ElapsedEventArgs e)
    {
        // 处理心跳
        HeartBeat();
        timer.Start();
    }

    // 心跳
    private void HeartBeat()
    {
        long timeNow = Sys.GetTimeStamp();

        for (int i = 0; i < conns.Length; i++)
        {
            Conn conn = conns[i];
            if (conn == null) continue;
            if (!conn.isUse) continue;

            if (conn.lastTickTime < timeNow - heartBeatTime)
            {
                Console.WriteLine("心跳引起断开连接" + conn.GetAdress());
                lock (conn)
                    conn.Close();
            }
        }
    }

    // 获取连接池索引，返回负数表示获取失败
    public int NewIndex()
    {
        if (conns == null)
            return -1;
        for (int i = 0; i < conns.Length; i++)
        {
            if (conns[i] == null)
            {
                conns[i] = new Conn();
                return i;
            }
            else if (conns[i].isUse == false)
            {
                return i;
            }
        }
        return -1;
    }

    // 给新的连接分配conn
    // 异步接收客户端数据
    // 再次调用BeginAccept实现循环
    private void AcceptCb(IAsyncResult ar)
    {
        try
        {
            Socket socket = listenfd.EndAccept(ar);
            int index = NewIndex();

            if (index < 0)
            {
                socket.Close();
                Console.WriteLine("警告 连接以满");
            }
            else
            {
                Conn conn = conns[index];
                conn.Init(socket);
                string adr = conn.GetAdress();
                Console.WriteLine("客户端连接 " + adr + "池id " + index);
                conn.socket.BeginReceive(conn.readBuff, conn.buffCount, conn.BuffRemain(), SocketFlags.None, ReceiveCb, conn);
            }
            listenfd.BeginAccept(AcceptCb, null);
        }
        catch(Exception e)
        {
            Console.WriteLine("AcceptCb失败" + e.Message);
        }
    }

    // 接受并处理消息，因为是多人聊天，服务端收到消息后。要把它转发给所有人
    // 如果收到客户端关闭连接的信号 count==0 则断开连接
    // 继续调用BeginReceive接收下一个数据

    //处理粘包分包 使用conn中的buffCount表示缓冲区数据长度
    //框架中至少有异步回调 定时器 主线程 三个线程共同处理conn对象，不可避免地有竞争 需要加锁
    private void ReceiveCb(IAsyncResult ar)
    {
        Conn conn = (Conn)ar.AsyncState;
        lock(conn)
        {
            try
            {
                int count = conn.socket.EndReceive(ar);
                //关闭信号
                if (count <= 0)
                {
                    Console.WriteLine("收到 " + conn.GetAdress() + "断开连接");
                    conn.Close();
                    return;
                }

                // 处理粘包分包
                conn.buffCount += count;
                ProcessData(conn);

                //继续接收
                conn.socket.BeginReceive(conn.readBuff, conn.buffCount, conn.BuffRemain(), SocketFlags.None, ReceiveCb, conn);
            }
            catch (Exception e)
            {
                Console.WriteLine("收到 " + conn.GetAdress() + "断开连接");
                conn.Close();
            }
        }
    }

    // 因为尚未做消息分发的功能，下面的代码中我们只是解析消息字符串并调用send发回客户端
    // 稍后会处理消息分发
    private void ProcessData(Conn conn)
    {
        // 小于长度字节
        if (conn.buffCount < sizeof(Int32))
        {
            return;
        }
        //消息长度
        Array.Copy(conn.readBuff, conn.lenBytes, sizeof(Int32));
        conn.msgLength = BitConverter.ToInt32(conn.lenBytes, 0);
        if (conn.buffCount < conn.msgLength + sizeof(Int32))
        {
            return;
        }

        // 处理消息
        string str = System.Text.Encoding.UTF8.GetString(conn.readBuff, sizeof(Int32), conn.msgLength);

        if (str == "HeartBeat")
        {
            conn.lastTickTime = Sys.GetTimeStamp();
            Console.WriteLine("收到心跳" + conn.GetAdress());
        }
        else
        {
            Console.WriteLine("收到消息 " + conn.GetAdress() + str);
            Send(conn, str);
        }

        // 清除已处理的消息
        int count = conn.buffCount - conn.msgLength - sizeof(Int32);
        Array.Copy(conn.readBuff, sizeof(Int32) + conn.msgLength, conn.readBuff, 0, count);
        conn.buffCount = count;
        if (conn.buffCount > 0)
        {
            ProcessData(conn);
        }
    }

    private void Send(Conn conn, string str)
    {
        byte[] bytes = System.Text.Encoding.UTF8.GetBytes(str);
        byte[] length = BitConverter.GetBytes(bytes.Length);
        byte[] sendbuff = length.Concat(bytes).ToArray();
        try
        {
            conn.socket.BeginSend(sendbuff, 0, sendbuff.Length, SocketFlags.None, null, null);
        }
        catch(Exception e)
        {
            Console.WriteLine("发送消息 " + conn.GetAdress() + " : " + e.Message);
        }
    }

    public void Broadcast(string str)
    {
        byte[] bytes = System.Text.Encoding.UTF8.GetBytes(str);
        byte[] length = BitConverter.GetBytes(bytes.Length);
        byte[] sendbuff = length.Concat(bytes).ToArray();
        for (int i = 0; i < conns.Length; i++)
        {
            if (conns == null) continue;
            if (!conns[i].isUse) continue;
            var conn = conns[i];
            try
            {
                conn.socket.BeginSend(sendbuff, 0, sendbuff.Length, SocketFlags.None, null, null);
            }
            catch (Exception e)
            {
                Console.WriteLine("广播消息 " + conn.GetAdress() + " : " + e.Message);
            }
        }
    }

    public void Close()
    {
        for (int i = 0; i < conns.Length; i++)
        {
            Conn conn = conns[i];
            if (conn == null)
                continue;
            if (!conn.isUse)
                continue;
            lock(conn)
            {
                Console.WriteLine("服务器主动 " + conn.GetAdress() + "断开连接");
                conn.Close();
            }
        }
    }
}
```

```c#
using System;

class Program
{
    static void Main(string[] args)
    {
        Serv serv = new Serv();
        serv.Start("127.0.0.1", 1234);
        Console.ReadLine();
    }
}
```