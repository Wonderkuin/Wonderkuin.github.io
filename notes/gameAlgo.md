# 游戏有用的算法

### [<主页](https://www.wangdekui.com/)

#### Boids 类鸟群行为

[Sebastian Lague](https://github.com/SebLague/Boids)
[翻译](https://space.bilibili.com/2216109)

```c#
public float minVelocity = 2f;
public float maxVelocity = 5f;

public float nearDist = 30f;                //判定为附近的 boid 的最小范围值
public float collisionDist = 5f;            //判定为最近的 boid 的最小范围值(具有碰撞风险)

public float velocityMatchingAmt = 0.01f;   //与 附近的boid 的平均速度 乘数(影响新速度)
public float flockCenteringAmt = 0.15f;     //与 附近的boid 的平均三维间距 乘数(影响新速度)
public float collisionAvoidanceAmt = -0.5f; //与 最近的boid 的平均三维间距 乘数(影响新速度)

public float targetAvoiddanceDsit = 15f;     // 接近目标距离 最小范围
public float targetAtrractionAmt = 0.01f;    //当 目标位置距离 过大时，与其间距的 乘数(影响新速度)
public float targetAvoidanceAmt = 0.75f;     //当 目标位置距离 过小时，与其间距的 乘数(影响新速度)

public float velocityLerpAmt = 0.25f;        //线性插值法计算新速度的 乘数

public Vector3 TargetPos;                    //目标位置
public List<Boid> boids;                     //所有 boid

public class Boid
{
    public Vector3 velocity;        //当前速度
    public Vector3 newVelocity;     //下一帧中的速度
    public Vector3 newPosition;     //下一帧中的位置

    public List<Boid> neighbors = new List<Boid>();        //附近所有的 Boid 的表
    public List<Boid> collisionRisks = new List<Boid>();   //距离过近的所有 Boid 的表(具有碰撞风险，需要处理)
    public Boid closest;                //最近的 Boid

    private void Update()
    {
        //获取到 当前boid 附近所有的Boids 的表
        GetNeighbors(this, ref neighbors, ref collisionRisks);

        //使用当前位置和速度初始化新位置和新速度
        newVelocity = velocity;
        newPosition = this.transform.position;

        //加上 所有邻近Boid对象 的平均速度，主要是方向
        Vector3 neighborVel = GetAverageVelocity(neighbors);
        newVelocity += neighborVel * velocityMatchingAmt;
        
        // 凝聚向心性：当前boid 向 附近boids中心 移动
        Vector3 neighborCenterOffset = GetAveragePosition(neighbors) - this.transform.position;
        newVelocity += neighborCenterOffset * flockCenteringAmt;

        // 排斥性：避免撞到 附近的Boid
        Vector3 dist;
        if (collisionRisks.Count > 0)   //处理 最近的boid 表
        {
            //取得 最近的所有boid 的平均位置
            Vector3 collisionAveragePos = GetAveragePosition(collisionRisks);
            dist = collisionAveragePos - this.transform.position;
            //将 新速度 += 与最近boid的平均间距*flockCenteringAmt
            newVelocity += dist * collisionAvoidanceAmt;
        }

        // 若距离目标太远，则靠近；反之离开
        dist = TargetPos - this.transform.position;
        if (dist.magnitude > targetAvoiddanceDsit)
            newVelocity += dist.normalized * targetAtrractionAmt;
        else
            newVelocity += dist.normalized * -targetAvoidanceAmt;
    }

    private void LateUpdate()
    {
        // 线性插值法
        velocity = (1 - velocityLerpAmt) * velocity + velocityLerpAmt * newVelocity;

        // 速度范围
        if (velocity.magnitude > maxVelocity)
            velocity = velocity.normalized * maxVelocity;
        if (velocity.magnitude < minVelocity)
            velocity = velocity.normalized * minVelocity;

        // 新位置
        newPosition = this.transform.position + velocity * Time.deltaTime;

        // 方向
        this.transform.LookAt(newPosition);

        // 位移
        this.transform.position = newPosition;
    }
}

//todo 空间划分
public void GetNeighbors(Boid boi, ref List<Boid> neighbors, ref List<Boid> collisionRisks)
{
    neighbors.Clear();
    collisionRisks.Clear();

    float closesDist = float.MaxValue;              //最小间距

    //遍历所有的 boid
    foreach (Boid b in boids)
    {
        if (b == boi)                               //跳过自身
            continue;

        //三维间距
        var delta = b.transform.position - boi.transform.position;
        var dist = delta.magnitude;                 //实数间距

        if (dist < closesDist)
        {
            closesDist = dist;      //更新最小间距
            closest = b;            //更新最近的 boid
        }

        if (dist < nearDist)  //处在附近的 boid 范围
            neighbors.Add(b);

        if (dist < collisionDist) //有碰撞风险的 boid 范围
            collisionRisks.Add(b);
    }

    // 计算速度依赖于 附近的boid，所以要保证不为空
    if (neighbors.Count == 0)
        neighbors.Add(closest);
}
```

#### A* 寻路

```c#
public class Grid : MonoBehaviour
{
    public Transform StartPosition;     // 起点
    public LayerMask WallMask;          //
    public Vector2 vGridWorldSize;      // 地图大小
    public float fNodeRadius;           // 节点大小 半径
    public float fDistanceBetweenNodes; // 节点间距

    Node[,] NodeArray;                  // 节点数组
    public List<Node> FinalPath;        // 完成的路径

    float fNodeDiameter;                // 节点大小 直径
    int iGridSizeX, iGridSizeY;         // 网格的大小

    private void Start()
    {
        fNodeDiameter = fNodeRadius * 2;
        iGridSizeX = Mathf.RoundToInt(vGridWorldSize.x / fNodeDiameter);
        iGridSizeY = Mathf.RoundToInt(vGridWorldSize.y / fNodeDiameter);
        CreateGrid();
    }

    void CreateGrid()
    {
        NodeArray = new Node[iGridSizeX, iGridSizeY];
        // 网格左下角 真实位置
        Vector3 bottomLeft = transform.position - Vector3.right * vGridWorldSize.x / 2 - Vector3.forward * vGridWorldSize.y / 2;
        for (int x = 0; x < iGridSizeX; x++)
        {
            for (int y = 0; y < iGridSizeY; y++)
            {
                Vector3 worldPoint = bottomLeft + Vector3.right * (x * fNodeDiameter + fNodeRadius) + Vector3.forward * (y * fNodeDiameter + fNodeRadius);
                bool Wall = true;

                if (Physics.CheckSphere(worldPoint, fNodeRadius, WallMask))
                    Wall = false;

                // 创建新的节点
                NodeArray[x, y] = new Node(Wall, worldPoint, x, y);
            }
        }
    }

    // 获取相邻节点
    public List<Node> GetNeighboringNodes(Node a_NeighborNode)
    {
        List<Node> NeighborList = new List<Node>();
        int icheckX, icheckY;       // 检查下标，避免超出范围

        // 检查右侧
        icheckX = a_NeighborNode.iGridX + 1;
        icheckY = a_NeighborNode.iGridY;
        if (icheckX >= 0 && icheckX < iGridSizeX)
        {
            if (icheckY >= 0 && icheckY < iGridSizeY)
            {
                NeighborList.Add(NodeArray[icheckX, icheckY]);
            }
        }

        // 检查左侧
        icheckX = a_NeighborNode.iGridX - 1;
        icheckY = a_NeighborNode.iGridY;
        if (icheckX >= 0 && icheckX < iGridSizeX)
        {
            if (icheckY >= 0 && icheckY < iGridSizeY)
            {
                NeighborList.Add(NodeArray[icheckX, icheckY]);
            }
        }

        // 检查上边
        icheckX = a_NeighborNode.iGridX;
        icheckY = a_NeighborNode.iGridY + 1;
        if (icheckX >= 0 && icheckX < iGridSizeX)
        {
            if (icheckY >= 0 && icheckY < iGridSizeY)
            {
                NeighborList.Add(NodeArray[icheckX, icheckY]);
            }
        }

        // 检查下边
        icheckX = a_NeighborNode.iGridX;
        icheckY = a_NeighborNode.iGridY - 1;
        if (icheckX >= 0 && icheckX < iGridSizeX)
        {
            if (icheckY >= 0 && icheckY < iGridSizeY)
            {
                NeighborList.Add(NodeArray[icheckX, icheckY]);
            }
        }

        return NeighborList;
    }

    // 获取给定世界坐标位置的节点
    // 通过 加 vGridWorldSize.x / 2 再除以 vGridWorldSize.x
    // 得到右上角格子中的一个点，再通过四舍五入
    // 得到整点
    public Node NodeFromWorldPoint(Vector3 a_vWorldPos)
    {
        float ixPos = ((a_vWorldPos.x + vGridWorldSize.x / 2) / vGridWorldSize.x);
        float iyPos = ((a_vWorldPos.z + vGridWorldSize.y / 2) / vGridWorldSize.y);

        ixPos = Mathf.Clamp01(ixPos);
        iyPos = Mathf.Clamp01(iyPos);

        int ix = Mathf.RoundToInt((iGridSizeX - 1) * ixPos);
        int iy = Mathf.RoundToInt((iGridSizeY - 1) * iyPos);

        return NodeArray[ix, iy];
    }


    // 编辑器绘制
    private void OnDrawGizmos()
    {
        Gizmos.DrawWireCube(transform.position, new Vector3(vGridWorldSize.x, 1, vGridWorldSize.y));

        if (NodeArray != null)
        {
            foreach (Node n in NodeArray)
            {
                if (n.bIsWall)
                    Gizmos.color = Color.white;
                else
                    Gizmos.color = Color.yellow;

                if (FinalPath != null)
                {
                    if (FinalPath.Contains(n))
                    {
                        Gizmos.color = Color.red;
                    }
                }

                Gizmos.DrawCube(n.vPosition, Vector3.one * (fNodeDiameter - fDistanceBetweenNodes));
            }
        }
    }
}

public class Node
{
    public int iGridX;              // 格子下标 x
    public int iGridY;              // 格子下标 y

    public bool bIsWall;            // 是否是障碍物
    public Vector3 vPosition;       // 世界位置

    public Node ParentNode;         // 存储先前来自哪个节点，以便可以追踪最短路径

    public int igCost;              // 移动到下一节点的 Cost
    public int ihCost;              // 该节点到目标的距离

    public int FCost { get { return igCost + ihCost; } }

    public Node(bool a_bIsWall, Vector3 a_vPos, int a_igridX, int a_igridY)
    {
        bIsWall = a_bIsWall;
        vPosition = a_vPos;
        iGridX = a_igridX;
        iGridY = a_igridY;
    }
}

public class Pathfinding : MonoBehaviour
{
    Grid GridReference;             // Grid类

    public Vector3 startPosition;
    public Vector3 targetPosition;

    private void Update()
    {
        // Start节点 Target节点
        Node StartNode = GridReference.NodeFromWorldPoint(startPosition);
        Node TargetNode = GridReference.NodeFromWorldPoint(targetPosition);

        List<Node> OpenList = new List<Node>();
        HashSet<Node> ClosedList = new HashSet<Node>();

        OpenList.Add(StartNode);

        while(OpenList.Count > 0)
        {
            Node CurrentNode = OpenList[0];
            for(int i = 1; i < OpenList.Count; i++) // i == 1
            {
                // 如果 FCost 小于等于当前节点，
                if (OpenList[i].FCost < CurrentNode.FCost || OpenList[i].FCost == CurrentNode.FCost && OpenList[i].ihCost < CurrentNode.ihCost)
                    CurrentNode = OpenList[i];//将当前节点设置为该对象
            }

            OpenList.Remove(CurrentNode);
            ClosedList.Add(CurrentNode);

            // 如果找到了
            if (CurrentNode == TargetNode)
                GetFinalPath(StartNode, TargetNode);

            // 遍历当前节点的每个邻居
            foreach (Node NeighborNode in GridReference.GetNeighboringNodes(CurrentNode))
            {
                // 如果是墙 或者是 Close节点，跳过
                if (!NeighborNode.bIsWall || ClosedList.Contains(NeighborNode))
                    continue;

                // 邻居节点的 FCost
                int MoveCost = CurrentNode.igCost + GetManhattenDistance(CurrentNode, NeighborNode);

                // 如果 FCost 小于 GCost，或者邻居节点不是 Open节点
                if (MoveCost < NeighborNode.igCost || !OpenList.Contains(NeighborNode))
                {
                    NeighborNode.igCost = MoveCost;
                    NeighborNode.ihCost = GetManhattenDistance(NeighborNode, TargetNode); // 设置 Hcost
                    NeighborNode.ParentNode = CurrentNode; // 设置父级节点

                    // 添加到 Open节点
                    if(!OpenList.Contains(NeighborNode))
                        OpenList.Add(NeighborNode);
                }
            }

        }
    }

    void GetFinalPath(Node a_StartingNode, Node a_EndNode)
    {
        List<Node> FinalPath = new List<Node>();
        Node CurrentNode = a_EndNode;
        while(CurrentNode != a_StartingNode)
        {
            FinalPath.Add(CurrentNode);
            CurrentNode = CurrentNode.ParentNode;
        }

        FinalPath.Reverse(); // 反转路径，调换顺序

        GridReference.FinalPath = FinalPath;

    }

    int GetManhattenDistance(Node a_nodeA, Node a_nodeB)
    {
        int ix = Mathf.Abs(a_nodeA.iGridX - a_nodeB.iGridX);//x1-x2
        int iy = Mathf.Abs(a_nodeA.iGridY - a_nodeB.iGridY);//y1-y2
        return ix + iy;
    }
}
```

[Wikipedia的伪代码实例](https://en.wikipedia.org/wiki/A*_search_algorithm#Pseudocode)

```c#
// 建立节点的关联
private void ConnectNeighbors (Node node)
{
    int nodeX = (int)node.transform.position.x;
    int nodeZ = (int)node.transform.position.z;
    for (var z = nodeZ - 1; z <= nodeZ + 1; z++) {
        for (var x = nodeX - 1; x <= nodeX + 1; x++) {
            Connect (node, x, z);
        }
    }
}

private void Connect (Node node, int otherX, int otherZ)
{
    if (otherX < 0 || otherX >= gridWidth || otherZ < 0 || otherZ >= gridHeight)
        return;

    Node otherNode = nodes [otherZ, otherX].GetComponent<Node> ();

    if (node.Type == Node.NodeType.Wall || otherNode.Type == Node.NodeType.Wall)
        return;

    if (node == otherNode)
        return;
    
    node.AddConnection (otherNode);
    otherNode.AddConnection (node);
}

// 节点
public class Node
{
	public Dictionary<Node,float> Connections = new Dictionary<Node, float> ();

    // 将连接添加到具有给定边长的另一个节点
	public void AddConnection (Node node, float edgeLength)
	{
		if (!connections.ContainsKey (node))
			connections.Add (node, edgeLength);
	}

    // 将连接添加到另一个节点 节点的距离用作边缘长度
	public void AddConnection (Node node)
	{
		AddConnection (node, Vector3.Distance (node.transform.position, transform.position));
	}
}

public static class AStar
{
    private class DefaultDict
    {
        Dictionary<Node,float> container;

        public DefaultDict () {
            container = new Dictionary<Node,float> ();
        }

        public void Set (Node node, float value) {
            container [node] = value;
        }

        public float Get (Node node) {
            if (container.ContainsKey (node)) {
                return container [node];
            } else {
                return float.MaxValue;
            }
        }
    }

    public static bool FindPath (Node start, Node goal)
    {
        var closedSet = new HashSet<Node> ();
        var openSet = new HashSet<Node> ();
        openSet.Add (start);

        var gScore = new DefaultDict ();
        gScore.Set (start, 0);

        var fScore = new DefaultDict ();
        fScore.Set (start, HeuristicEstimate (start, goal));

        var cameFrom = new Dictionary<Node, Node> ();

        Node current = start;
        while (openSet.Count > 0) {
            current = FindLowestFScore (openSet, current, fScore);
            if (current.Type != Node.NodeType.Start && current.Type != Node.NodeType.Goal) {
                current.Type = Node.NodeType.Explored;
            }
            if (current == goal) {
                ReconstructPath (cameFrom, current);
                return true;
            }
            openSet.Remove (current);
            closedSet.Add (current);

            foreach (var neighbor in current.Connections.Keys) {
                if (closedSet.Contains (neighbor)) {
                    continue;
                }

                if (!openSet.Contains (neighbor)) {
                    openSet.Add (neighbor);
                }

                var tentativeGScore = gScore.Get (current) + current.Connections [neighbor];
                if (tentativeGScore	>= gScore.Get (neighbor)) {
                    continue;
                }

                cameFrom [neighbor] = current;
                gScore.Set (neighbor, tentativeGScore);
                fScore.Set (neighbor, gScore.Get (neighbor) + HeuristicEstimate (neighbor, goal));
            }
        }

        return false;
    }


    // 查找F分数最低的节点。
    private static Node FindLowestFScore (HashSet<Node> openSet, Node current, DefaultDict fScore)
    {
        var lowest = float.MaxValue;
        Node selectedNode = current;

        foreach (var node in openSet) {
            if (fScore.Get (node) < lowest) {
                selectedNode = node;
                lowest = fScore.Get (node);
            }
        }

        return selectedNode;
    }


    // 使用欧几里得距离作为启发式估计
    private static float HeuristicEstimate (Node from, Node to)
    {
        return Vector3.Distance (from.transform.position, to.transform.position);
    }


    // 重建路径
    private static void ReconstructPath (Dictionary<Node,Node> cameFrom, Node current)
    {
        List<Node> totalPath = new List<Node> ();
        totalPath.Add (current);

        while (cameFrom.ContainsKey (current)) {
            current = cameFrom [current];
            totalPath.Add (current);
            if (current.Type != Node.NodeType.Start && current.Type != Node.NodeType.Goal) {
                current.Type = Node.NodeType.Chosen;
            }
        }
    }
}
```

### 图形学基础

#### DDA算法 画线

```c++
void DrawLine(const Point2& p1, const Point2& p2, COLORREF color)
{
    //浮点数除以0得到无穷大，正常数字除以无穷大等于0，所以这个运算是安全的
	double k = (p1.Y - p2.Y) / (p1.X - p2.X);
	if (ABS(k) < 1)
	{
		double dy = k;
		int x = 0;
		double y = 0.0;
		int ex = 0;
		if (p1.X < p2.X) //把起点的x,y值赋给sx,sy
		{
			x = (int)p1.X;
			y = p1.Y;
			ex = (int)p2.X;
		}
		else
		{
			x = (int)p2.X;
			y = p2.Y;
			ex = (int)p1.X;
		}
		for (; x < ex; x++)
		{
			graphicsdevice.SetPixel(x, (int)y, color);
			y += dy;
		}
	}
	else // 斜率太大，x 变化 1，y变化很大，此时线条就会断裂
	{
		double dx = 1 / k;
		double x = 0.0;
		int y = 0;
		int ey = 0;
		if (p1.Y < p2.Y)
		{
			x = p1.X;
			y = (int)p1.Y;
			ey = (int)p2.Y;
		}
		else
		{
			x = p2.X;
			y = (int)p2.Y;
			ey = (int)p1.Y;
		}
		for (; y < ey; y++)
		{
			graphicsdevice.SetPixel((int)x, y, color);
			x += dx;
		}
	}
}
```

#### 多边形扫描线填充算法 简化 绘制三角形

先将三角形的三个顶点按照 Y 值从小到大排序为𝑃1、𝑃2、𝑃3  
然后在𝑃1𝑃3这条边上面找到一点𝑃′，使得𝑃′的 Y 值等于𝑃2的 Y值  
然后再使用𝑃2、𝑃′的 X 值对𝑃2、𝑃′进行排序，使得𝑃2.𝑥 < 𝑃′. 𝑥  
这样任意一个三角形都能被上述方法分割成两个类似的三角形  
在排序之后△ 𝑃1𝑃2𝑃′和△ 𝑃2𝑃′𝑃3的绘制方法如下  

```
             0 𝑃1
           0 0
      𝑃2 0   0 𝑃′
          0  0
           0 0
         𝑃3 0

//下半边三角形的绘制
dx_left=1/𝑃1𝑃2斜率
dx_right=1/𝑃1𝑃′斜率
sx=𝑃1. 𝑥//扫描线起点 X 值
ex=𝑃1.𝑥//扫描线终点 X 值
for(从𝑃1. 𝑦到𝑃2. 𝑦的每个 y)
{
    //sx 到 ex 相当于扫描线的一部分
    for(从 sx 到 ex 的每个 x)
    {
    绘制(x,y)
    }
    sx=sx+dx_left
    ex=ex+dx_right
}

//上半边三角形的绘制
dx_left =1/𝑃2𝑃3斜率
dx_right =1/𝑃′𝑃3斜率
sx=𝑃2. 𝑥//扫描线起点 X 值
ex=𝑃′.𝑥//扫描线终点 X 值
for(从𝑃2. 𝑦到𝑃3. 𝑦的每个 y)
{
    //sx 到 ex 相当于扫描线的一部分
    for(从 sx 到 ex 的每个 x)
    {
    绘制(x,y)
    }
    sx=sx+dx1
    ex=ex+dx2
}
```

```
注意 P1、P2、P3 顺序
三角形的顶点坐标 P1、P2、P3
我们可以用用两条边作为向量得到 vector(p1, p2) vector(p1, p3)
向量叉乘结果的模长即为三角形面积的两倍

因为叉乘的两个向量是三维向量
所以我们构造一个坐标系
以原先屏幕的 x、y 轴作为坐标轴
垂直于屏幕向外作为 z 轴(右手坐标系)
这样原先的坐标点(x,y)就会变成(x,y,0)

叉乘结果为： G / 2
G = 𝑃1. 𝑥 ∗ 𝑃2.𝑦 + 𝑃2.𝑥 ∗ 𝑃3.𝑦 + 𝑃3.𝑥 ∗ 𝑃1. 𝑦 − 𝑃1.𝑥 ∗ 𝑃3. 𝑦 − 𝑃2. 𝑥 ∗ 𝑃1. 𝑦 − 𝑃3. 𝑥 ∗ 𝑃2.𝑦
结果可以是负数，面积也叫有向面积，三角形剔除需要用到

如果 G == 0，不需要绘制三角形
则 abs(G) < 1e-5 ，不需要绘制三角形

可能绘制超过三角形范围的像素
在某个边斜率特别小的时候，计算出的dx特别大
比如：
P1 4.25, 1.99999
P2 3.50, 2.00001
P3 5.05, 2.00001

P1P2, dx = -37500
P1P3, dx = 40000

因此需要限制sx的范围
```

```c++
void DrawTriangle2D(const Point2& a, const Point2& b, const Point2& c, COLORREF color)
{
	// 面积为0，放弃绘制
	if (abs(a.X * b.Y + b.X * c.Y + c.X * a.Y - a.X * c.Y - b.X * a.Y - c.X * b.Y) < 1e-15)
		return;

	// 对三个点进行排序 冒泡排序，三个数要比三次
	const Point2* pa = &a;
	const Point2* pb = &b;
	const Point2* pc = &c;
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point2* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	if (pb->Y > pc->Y) //使得p3的y值不小于p2的y值
	{
		const Point2* tmp = pb;
		pb = pc;
		pc = tmp;
	}
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point2* tmp = pa;
		pa = pb;
		pb = tmp;
	}
    //得到P'，再次排序
	Point2 p = Point2((pc->X - pa->X) * ((pb->Y - pa->Y) / (pc->Y - pa->Y)) + pa->X, pb->Y); 
	const Point2* _p = &p;
	if (pb->X > _p->X)
	{
		const Point2* tmp = pb;
		pb = _p;
		_p = tmp;
	}

	const Point2& P1 = *pa;
	const Point2& P2 = *pb;
	const Point2& P3 = *pc;
	const Point2& P_ = *_p;
	double DXleft[2] = { (P1.X - P2.X) / (P1.Y - P2.Y), (P2.X - P3.X) / (P2.Y - P3.Y) };
	double DXright[2] = { (P1.X - P_.X) / (P1.Y - P_.Y), (P_.X - P3.X) / (P_.Y - P3.Y) };
	double Start_x[2] = { P1.X, P2.X };
	double End_x[2] = { P1.X, P_.X };
	double Start_y[2] = { P1.Y, P2.Y };
	double End_y[2] = { P2.Y, P3.Y };
	double edge_left_ex[] = { P2.X, P3.X }; //记录每条边结束的x值
	double edge_right_ex[] = { P_.X, P3.X };
	for (int i = 0; i < 2; i++)
	{
		double dx_left = DXleft[i];
		double dx_right = DXright[i];
		double sx = Start_x[i]; //扫描线起点X值
		double ex = End_x[i];	//扫描线终点X值
		double sy = Start_y[i]; //三角形起始扫描线
		double ey = End_y[i];	//三角形结束扫描线
		for (int y = (int)sy; y < ey; y++)
		{
			if (dx_left < 0) //sx随着y增大而减小
			{
				sx = _max(sx, edge_left_ex[i]); //将sx限定为不小于终点x
			}
			else if (dx_left > 0) //sx随着y增大而增大
			{
				sx = _min(sx, edge_left_ex[i]); //将sx限定为不大于终点x
			}
			if (dx_right < 0) //ex随着y增大而减小
			{
				ex = _max(ex, edge_right_ex[i]); //将ex限定为不小于终点x
			}
			else if (dx_right > 0) //ex随着y增大而增大
			{
				ex = _min(ex, edge_right_ex[i]); //将ex限定为不大于终点x
			}
			//sx到ex相当于扫描线的一部分
			for (int x = (int)sx; x <= ex; x++)
			{
				graphicsdevice.SetPixel(x, y, color);
			}
			sx = sx + dx_left;
			ex = ex + dx_right;
		}
	}
}
```

#### 重心坐标插值

```
一维
Vp = W1 * V1 + W2 * V2
W1 + W2 = 1
长10的无质量杆，左端质量6，右端质量14，重心在左端起，位置7处
则：
W1 = 6 / (6 + 14) = 0.3
W2 = 14 / (6 + 14) = 0.7
V1 = 0
V2 = 10
Vp = 0.3 * 0 + 0.7 * 10 = 0.7
```

```
三角形重心
点 A
点 B
点 C
边 a 为 A对边
边 b 为 B对边
边 c 为 C对边
质量 Ma 为 A点质量
质量 Mb 为 B点质量
质量 Mc 为 C点质量
面积 Sa 为 a和P围住面积
面积 Sb 为 b和P围住面积
面积 Sc 为 c和P围住面积
重心 P， 重心坐标遵从线性插值，Px = (Ax + Bx + Cx) / 3
面积 S = Sa + Sb + Sc
质量 M = Ma + Mb + Mc

则有:
Sa / S = Ma / M
Sb / S = Mb / M
Sc / S = Mc / M

面积可以是负值，重心可以在三角形外
```

```
重心坐标插值在栅格化程序的优化

计算三角形面积花费的运算量比较大
而重心坐标插值的结果在直线上是线性变化的
所以我们可以将重心坐标插值按照扫描线的填充顺序进行插值
```

```
伪代码

---------------
新增属性，用作灰度

/*
在顶点排序求𝑃′的时候使用线性插值计算𝑃′的属性值
*/
DXleft=[1/𝑃1𝑃2斜率, 1/𝑃2𝑃3斜率]//存放左边两条边的斜率倒数
DXright=[1/𝑃1𝑃′斜率, 1/𝑃′𝑃3斜率]//存放右边两条边的斜率倒数
DVyleft=[𝑃1𝑃2边属性值在 Y 方向的变化量, 𝑃2𝑃3边属性值在 Y 方向的变化量]---------------
DVyright=[𝑃1𝑃′边属性值在 Y 方向的变化量, 𝑃′𝑃3边属性值在 Y 方向的变化量]---------------
Start_x=[𝑃1.𝑥, 𝑃2. 𝑥]
Start_v=[𝑃1.𝑣, 𝑃2.𝑣]---------------
End_x=[𝑃1.𝑥, 𝑃′. 𝑥]
End_v=[𝑃1. 𝑣, 𝑃′. 𝑣]---------------
Start_y=[𝑃1. 𝑦, 𝑃2. 𝑦]
End_y=[𝑃2. 𝑦, 𝑃3. 𝑦]
for(int i=0;i<2;i++)
{
    dx_left= DXleft[i]
    dx_right= DXright[i]
    dv_left= DVyleft[i]---------------
    dv_right = DVyright [i]---------------
    sx=Start_x[i]//扫描线起点 X 值
    sv=Start_v[i]//扫描线起点属性值---------------
    ex=End_x[i]//扫描线终点 X 值
    ev= End_v[i]//扫描线终点属性值---------------
    sy=Start_y[i]//三角形起始扫描线
    ey=End_y[i]//三角形结束扫描线
    for(从 sy 到 ey 的每个 y)
    {
        //sx 到 ex 相当于扫描线的一部分
        v=sv---------------
        dvx=(ev-sv)/(ex-sx)//计算属性值 v 在 x 方向上的增量---------------
        for(从 sx 到 ex 的每个 x)
        {
            绘制(x,y,v)
            v=v+dvx---------------
        }
        sx=sx+dx_left
        sv= sv+dv_left---------------
        ex=ex+dx_right
        ev=ev+dv_right---------------
    }
}
```

```c++
void SingleInterpolationIn2D(const Point2& a, const Point2& b, const Point2& c)
{
	// 面积为0, 放弃绘制
	if (abs(a.X * b.Y + b.X * c.Y + c.X * a.Y - a.X * c.Y - b.X * a.Y - c.X * b.Y) < 1e-15)
		return;

	// 冒泡排序，三个数要比三次
	const Point2* pa = &a;
	const Point2* pb = &b;
	const Point2* pc = &c;
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point2* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	if (pb->Y > pc->Y) //使得p3的y值不小于p2的y值
	{
		const Point2* tmp = pb;
		pb = pc;
		pc = tmp;
	}
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point2* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	Point2 p = Point2((pc->X - pa->X) * ((pb->Y - pa->Y) / (pc->Y - pa->Y)) + pa->X, pb->Y); //得到P'
	p.V = (pc->V - pa->V) * ((pb->Y - pa->Y) / (pc->Y - pa->Y)) + pa->V;					 //计算P点属性值
	const Point2* _p = &p;
	if (pb->X > _p->X)
	{
		const Point2* tmp = pb;
		pb = _p;
		_p = tmp;
	}

	const Point2& P1 = *pa;
	const Point2& P2 = *pb;
	const Point2& P3 = *pc;
	const Point2& P_ = *_p;
	double DXleft[2] = { (P1.X - P2.X) / (P1.Y - P2.Y), (P2.X - P3.X) / (P2.Y - P3.Y) };
	double DXright[2] = { (P1.X - P_.X) / (P1.Y - P_.Y), (P_.X - P3.X) / (P_.Y - P3.Y) };
	double DVyleft[2] = { (P1.V - P2.V) / (P1.Y - P2.Y), (P2.V - P3.V) / (P2.Y - P3.Y) };
	double DVyright[2] = { (P1.V - P_.V) / (P1.Y - P_.Y), (P_.V - P3.V) / (P_.Y - P3.Y) };
	double Start_x[2] = { P1.X, P2.X };
	double Start_v[2] = { P1.V, P2.V };
	double End_x[2] = { P1.X, P_.X };
	double End_v[2] = { P1.V, P_.V };
	double Start_y[2] = { P1.Y, P2.Y };
	double End_y[2] = { P2.Y, P3.Y };
	double edge_left_ex[] = { P2.X, P3.X }; //记录每条边结束的x值
	double edge_right_ex[] = { P_.X, P3.X };
	for (int i = 0; i < 2; i++)
	{
		double dx_left = DXleft[i];
		double dx_right = DXright[i];
		double dvy_left = DVyleft[i];
		double dvy_right = DVyright[i];
		double sx = Start_x[i]; //扫描线起点X值
		double sv = Start_v[i]; //扫描线起点属性值
		double ex = End_x[i];	//扫描线终点X值
		double ev = End_v[i];	//扫描线终点属性值
		double sy = Start_y[i]; //三角形起始扫描线
		double ey = End_y[i];	//三角形结束扫描线
		for (int y = (int)sy; y < ey; y++)
		{
			//sx到ex相当于扫描线的一部分
			double v = sv;
			double dvx = (ev - sv) / (ex - sx); //计算属性值v在x方向上的增量
			for (int x = (int)sx; x <= ex; x++)
			{
				graphicsdevice.SetPixel(x, y, RGB((int)v, (int)v, (int)v));
				v = v + dvx;
			}
			sx = sx + dx_left;
			sv = sv + dvy_left;
			ex = ex + dx_right;
			ev = ev + dvy_right;
		}
	}
}
```

#### 纹理

```c++
// 通常我们将纹理的左下角定为纹理的原点
// uv 直角坐标系，等于 xy 直角坐标系
// uv 的取值都是[0,1]
// u=1 时 x=width
// v=1 时 y=height
// 可以通过纹理坐标取得该点的颜色值
// 这个颜色称之为纹素。
// 我们可以通过如下的公式将 uv 坐标转换成纹理原始图片的 xy 坐标：
// 𝑥 = 𝑤𝑖𝑑𝑡ℎ ∗ 𝑢
// 𝑦 = ℎ𝑒𝑖𝑔ℎ𝑡 ∗ 𝑣

class Texture
{
private:
	COLORREF img[2][2] = {
	{RGB(255,0,0),RGB(0,255,0)},
	{RGB(0,0,255),RGB(255,123,172)}
	};
	double width = 2, heigth = 2;
public:
	COLORREF texture2D(double u, double v)
	{
        //因为u表示横轴，在我们这定义里面应该是最低维的数据，所以应该是img[v][u]表示纹素坐标
		return img[(int)(v * width)][(int)(u * width)];
	}
};

// 片段着色器
Texture texture;
COLORREF FS(std::vector<double>& values)
{
	return texture.texture2D(values[0], values[1]);
};
```

#### 齐次坐标 透视投影

```
一维
    x
额外增加一个维度
    (x, w)
解释
    (x) = (wx, w)
    2 = 2 / 1 = 4 / 2 = 6 / 3
    (wx, w)叫做 (x) 的 齐次坐标

齐次坐标是用一个N + 1维的坐标，来表示一个N维坐标

齐次坐标可以方便的使用矩阵进行坐标变换
W分量可以记录额外信息
比如：投影运算的时候，记录原始点的深度信息
让我们可以忍受多写一个分量带来的麻烦

齐次坐标可以区分 点 和 向量
(x, y, 0) 表示向量
因为
    x' = x / w = x / 0 = ∞
    y' = y / w = y / 0 = ∞
表示 x y 是无穷远点，并且增长率不一样

如果将坐标用于矩阵运算
平移还对向量无效
缩放和旋转却可以作用

和向量不能平移，但可以缩放旋转的结果一致，所以使用齐次坐标
```

```
二维透视投影
near 近平面
x' = x * ( n / y )

(x, y) = (50 , 100)
近平面距离观察点 10
，投影在近平面上的 x'为， y'自然都是 10
5 =  50 * ( 10 / 100 )


三维透视投影
near平面平行于 XY 平面，在z轴上
同理

x' = x * ( n / z )
y' = y * ( n / z )

坐标 (x, y, z) 透视投影后的坐标为 (xn/z, yn/z, n)
z分量暂时无意义，所以表示为
(xn/z, yn/z, any)

而且，空间中的直线，经过透视投影变换后，在平面上仍是一条直线
所以，只需要计算线段的两个端点的投影


用齐次坐标表示透视投影

(xn/z, yn/z, any)
(w*xn/z, w*yn/z, w*any, w)
如果令w=z，则
(xn, yn, any, z)
把投影公式修改为：
    x' = xn
    y' = yn
    z' = any
    w' = z

通过这种方法，可以用一个齐次坐标表示投影，还能附带额外信息
这里，w分量携带了顶点的原始信息，进行 透视校正插值 和 裁剪 的时候很有用
如果用齐次坐标表达投影，经过透视变换，不仅进行了投影，还把三维坐标变成了四维坐标
```

#### 透视校正插值

```c++
// 空间三角形的绘制
// 先把三角形变换成 2D 的，再绘制

Point4 Perspective(Point4& p, double n)
{
	auto pn = p.Normalize();//先将齐次坐标变成三维坐标
	return Point4(pn.X * n, pn.Y * n, pn.Z * n, pn.Z);//执行透视变换
}

int main()
{
	GraphicsDevice gd(640, 480);
	GraphicsLibrary gl(gd);
	Point4 a(0, 0, 100, 1);
	Point4 b(320, 480, 100, 1);
	Point4 c(640, 0, 100, 1);
	auto ta = Perspective(a, 80);
	auto tb = Perspective(b, 80);
	auto tc = Perspective(c, 80);
	ta.ValueArray = { 0,0 };
	tb.ValueArray = { 0.5,1 };
	tc.ValueArray = { 1,0 };
	gl.DrawTriangle(ta, tb, tc, 2, FS);
	getchar();
	return 0;
}

// 和 MultipleInterpolationArrayIn2D 几乎一致
void GraphicsLibrary::DrawTriangle(const Point4& sa, const Point4& sb, const Point4& sc, 
    int ValueLength, COLORREF(*FragmentShader)(std::vector<double>& values))
{
	//将齐次坐标规范化
	Point4 a = sa.Normalize();
	Point4 b = sb.Normalize();
	Point4 c = sc.Normalize();
	a.ValueArray = sa.ValueArray;
	b.ValueArray = sb.ValueArray;
	c.ValueArray = sc.ValueArray;

	//判断面积是否为0
	if (abs(a.X * b.Y + b.X * c.Y + c.X * a.Y - a.X * c.Y - b.X * a.Y - c.X * b.Y) < 1e-15)
	{
		return; //放弃本三角形的绘制
	}
	//按照文中的方法进行排序
	const Point4* pa = &a;
	const Point4* pb = &b;
	const Point4* pc = &c;
	//冒泡排序，三个数要比三次
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point4* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	if (pb->Y > pc->Y) //使得p3的y值不小于p2的y值
	{
		const Point4* tmp = pb;
		pb = pc;
		pc = tmp;
	}
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point4* tmp = pa;
		pa = pb;
		pb = tmp;
	}

	Point4 p = Point4((pc->X - pa->X) * ((pb->Y - pa->Y) / (pc->Y - pa->Y)) + pa->X, pb->Y, 0, 1); //得到P'
	p.ValueArray = std::vector<double>(ValueLength);											   //创建容器
	for (int i = 0; i < ValueLength; i++)														   //计算出属性集合中每个属性的值
	{
		p.ValueArray[i] = (pc->ValueArray[i] - pa->ValueArray[i]) * ((pb->Y - pa->Y) / (pc->Y - pa->Y)) + pa->ValueArray[i];
	}
	const Point4* _p = &p;
	if (pb->X > _p->X)
	{
		const Point4* tmp = pb;
		pb = _p;
		_p = tmp;
	}
	//排序完毕

	const Point4& P1 = *pa;
	const Point4& P2 = *pb;
	const Point4& P3 = *pc;
	const Point4& P_ = *_p;
	double DXleft[2] = { (P1.X - P2.X) / (P1.Y - P2.Y), (P2.X - P3.X) / (P2.Y - P3.Y) };
	double DXright[2] = { (P1.X - P_.X) / (P1.Y - P_.Y), (P_.X - P3.X) / (P_.Y - P3.Y) };

	std::vector<double> dvy_left1(ValueLength);
	std::vector<double> dvy_left2(ValueLength);
	std::vector<double> dvy_right1(ValueLength);
	std::vector<double> dvy_right2(ValueLength);
	for (int i = 0; i < ValueLength; i++) //计算出属性集合中每个属性的值
	{
		dvy_left1[i] = (P1.ValueArray[i] - P2.ValueArray[i]) / (P1.Y - P2.Y);
		dvy_left2[i] = (P2.ValueArray[i] - P3.ValueArray[i]) / (P2.Y - P3.Y);
		dvy_right1[i] = (P1.ValueArray[i] - P_.ValueArray[i]) / (P1.Y - P_.Y);
		dvy_right2[i] = (P_.ValueArray[i] - P3.ValueArray[i]) / (P_.Y - P3.Y);
	}
	std::vector<double> DVylefts[2] = { dvy_left1, dvy_left2 };
	std::vector<double> DVyrights[2] = { dvy_right1, dvy_right2 };

	double Start_x[2] = { P1.X, P2.X };
	double End_x[2] = { P1.X, P_.X };
	double Start_y[2] = { P1.Y, P2.Y };
	double End_y[2] = { P2.Y, P3.Y };

	std::vector<double> Start_vs[2] = { P1.ValueArray, P2.ValueArray };
	std::vector<double> End_vs[2] = { P1.ValueArray, P_.ValueArray };

	for (int i = 0; i < 2; i++)
	{
		double dx_left = DXleft[i];
		double dx_right = DXright[i];
		std::vector<double> dvy_lefts = DVylefts[i];
		std::vector<double> dvy_rights = DVyrights[i];
		double sx = Start_x[i];				   //扫描线起点X值
		std::vector<double> svs = Start_vs[i]; //扫描线起点属性值
		double ex = End_x[i];				   //扫描线终点X值
		std::vector<double> evs = End_vs[i];   //扫描线终点属性值
		double sy = Start_y[i];				   //三角形起始扫描线
		double ey = End_y[i];				   //三角形结束扫描线

		double edge_left_ex[] = { P2.X, P3.X }; //记录每条边结束的x值
		double edge_right_ex[] = { P_.X, P3.X };
		for (int y = (int)sy; y < ey; y++)
		{
			if (dx_left < 0) //sx随着y增大而减小
			{
				sx = _max(sx, edge_left_ex[i]); //将sx限定为不小于终点x
			}
			else if (dx_left > 0) //sx随着y增大而增大
			{
				sx = _min(sx, edge_left_ex[i]); //将sx限定为不大于终点x
			}
			if (dx_right < 0) //ex随着y增大而减小
			{
				ex = _max(ex, edge_right_ex[i]); //将ex限定为不小于终点x
			}
			else if (dx_right > 0) //ex随着y增大而增大
			{
				ex = _min(ex, edge_right_ex[i]); //将ex限定为不大于终点x
			}
			std::vector<double> vs = svs;
			std::vector<double> dvxs(ValueLength); //计算每一个属性值v在x方向上的增量
			for (int i = 0; i < ValueLength; i++) //计算每一个属性值v在x方向上的增量
			{
				dvxs[i] = (evs[i] - svs[i]) / (ex - sx);
			}
			//sx到ex相当于扫描线的一部分
			for (int x = (int)sx; x <= ex; x++)
			{
				graphicsdevice.SetPixel(x, y, FragmentShader(vs));
				for (int i = 0; i < ValueLength; i++)
				{
					vs[i] = vs[i] + dvxs[i];
				}
			}
			sx = sx + dx_left;

			ex = ex + dx_right;
			for (int i = 0; i < ValueLength; i++)
			{
				svs[i] = svs[i] + dvy_lefts[i];
				evs[i] = evs[i] + dvy_rights[i];
			}
		}
	}
}
```

#### 绘制两个三角形 拼接成一个正方形

```c++
// 画两个三角形 改变纹理坐标就行 
int main()
{
	GraphicsDevice gd(640, 480);
	GraphicsLibrary gl(gd);

	Point4 a1(0, 0, 100, 1);
	Point4 b1(480, 0, 100, 1);
	Point4 c1(0, 480, 100, 1);
	auto ta1 = Perspective(a1, 80);
	auto tb1 = Perspective(b1, 80);
	auto tc1 = Perspective(c1, 80);
	ta1.ValueArray = { 0,0 };
	tb1.ValueArray = { 1,0 };
	tc1.ValueArray = { 0,1 };
	gl.DrawTriangle(ta1, tb1, tc1, 2, FS);//绘制第一个三角形


	Point4 a2(480, 0, 100, 1);
	Point4 b2(0, 480, 100, 1);
	Point4 c2(480, 480, 100, 1);
	auto ta2 = Perspective(a2, 80);
	auto tb2 = Perspective(b2, 80);
	auto tc2 = Perspective(c2, 80);
	ta2.ValueArray = { 1,0 };
	tb2.ValueArray = { 0,1 };
	tc2.ValueArray = { 1,1 };
	gl.DrawTriangle(ta2, tb2, tc2, 2, FS);//绘制第二个三角形

	getchar();
	return 0;
}
```

#### 仿射变换导致的错误

```
上面两个小结中，对纹理属性的插值方法是错误的
只是刚好 被投影的平面 和 near平面 平行
这种情况下， 在屏幕空间进行线性插值刚好得到正确结果

如果两个三角形和near平面有一点点夹角，立马就能发现错误


透视校正插值

原始空间：
    没有经过投影变换的坐标系
屏幕空间:
    屏幕上的坐标系

不管是3D还是2D投影，投影到屏幕空间之后都会丢失一个维度


二维情况下的透视投影：
原始空间：
    A P B
    AB = L
    AP = l
    Wb = l / L
屏幕空间:
    A' P' B'
    A'B' = L'
    A'P' = l'
    Wb' = l' / L'

我们之前的程序是在 A'B' 上面进行线性插值
很容易发现， l / L != l' / L' ， 即 Wb != Wb'
Wb才是真正需要计算的结果，Wb'是个错误的结果，导致了我们的纹理出现错乱

设
在真实空间中，对于P点的
    Wb = t
投影到屏幕上的P'点
    Wb'=t'
我们可以得到下面方程：
    Xp' = n xp / yp = n[ (1-t) xa + t xb ] / [ (1-t)ya + tyb ]
    Xp' = (1-t')xa' + t' xb'
    Xa' = n xa / ya
    Xb' = n xb / yb

联立上面四个方程得:
    n[ (1-t) xa + t xb ] / [ (1-t)ya + tyb ] = (1-t')n xa / ya + t' n xb / yb

因为t是原始空间的权值，对于线性插值是有效的，t'是屏幕空间的权值
我们希望解出一个t=f(t')的函数，这样就可以在屏幕空间计算出原始空间的权值
    t = ya t' / [ yb + (ya - yb)t' ]

使用齐次坐标计算投影时用w分量记录原始顶点的原始深度值 ya和yb可以写成wa和wb
上述方程可以写为：
    t = wa t' / [ wb + (wa - wb)t' ]

如果在原始空间中的直线上有一个线性变化的属性值V
    (Vp - Va) / (Vb - Va)
=   (xp - xa) / (xb - xa)
=   (yp - ya) / (yb - ya)
=   (wp - wa) / (wb - wa)
=   t

通过上式，我们可以计算出 Vp, xp, yp, wp

计算Vp：
    (Vp - Va) / (Vb - Va) = wa t' / [ wb + (wa - wb)t' ]
得到：
    Vp = (wb va + wa vb t' - wb va t') / (wb + wa t' - wb t')

计算wp:
    wp = wa wb / (wb + wa t' - wb t')
更直观的表达式：
    1 / wp = 1 / wa + (1/wb - 1/wa) t'

大部分的资料介绍透视校正插值时，都会给出下面两个公式:
    1 / wp = 1 / wa + (1/wb - 1/wa) t'   (1)
    vp / wp = va / wa + (vb/wb - va/wa)t'  (2)
先用 (1) 计算出 wp 的值，再用 (2) 计算出 vp 的值，从而得到正确的插值结果
```

```c++
Point4 Point4::Normalize_special() const {
	return Point4(this->X / this->W, this->Y / this->W, this->Z / this->W, this->W);
}

void DrawTriangleSpecial(const Point4& sa, const Point4& sb, const Point4& sc, int ValueLength, COLORREF(*FragmentShader)(std::vector<double>& values))
{
	//将齐次坐标规范化
	Point4 a = sa.Normalize_special();
	Point4 b = sb.Normalize_special();
	Point4 c = sc.Normalize_special();
	a.ValueArray = sa.ValueArray;
	b.ValueArray = sb.ValueArray;
	c.ValueArray = sc.ValueArray;

	//判断面积是否为0 放弃本三角形的绘制
	if (abs(a.X * b.Y + b.X * c.Y + c.X * a.Y - a.X * c.Y - b.X * a.Y - c.X * b.Y) < 1e-15) {
		return;
	}
    
	//按照文中的方法进行排序
	const Point4* pa = &a;
	const Point4* pb = &b;
	const Point4* pc = &c;
	//冒泡排序，三个数要比三次
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point4* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	if (pb->Y > pc->Y) //使得p3的y值不小于p2的y值
	{
		const Point4* tmp = pb;
		pb = pc;
		pc = tmp;
	}
	if (pa->Y > pb->Y) //使得p2的y值不小于p1的y值
	{
		const Point4* tmp = pa;
		pa = pb;
		pb = tmp;
	}
	double p_t = (pb->Y - pa->Y) / (pc->Y - pa->Y);						 //得到屏幕空间点P'的t'值(c'点的权值)
	double p_omega = (pa->W * pc->W) / (pc->W + (pa->W - pc->W) * p_t);	 //计算P'点的ω值
	Point4 p = Point4(pa->X + (pc->X - pa->X) * p_t, pb->Y, 0, p_omega); //得到P'
	p.ValueArray = std::vector<double>(ValueLength);					 //创建容器
	for (int i = 0; i < ValueLength; i++)								 //计算出属性集合中每个属性的值
	{
		p.ValueArray[i] = (pc->W * pa->ValueArray[i] + (pa->W * pc->ValueArray[i] - pc->W * pa->ValueArray[i]) * p_t) / (pc->W + (pa->W - pc->W) * p_t);
	}
	const Point4* _p = &p;
	if (pb->X > _p->X) //让_p永远在右侧
	{
		const Point4* tmp = pb;
		pb = _p;
		_p = tmp;
	}
	//排序完毕

	const Point4& P1 = *pa;
	const Point4& P2 = *pb;
	const Point4& P3 = *pc;
	const Point4& P_ = *_p;
	double DXleft[2] = { (P1.X - P2.X) / (P1.Y - P2.Y), (P2.X - P3.X) / (P2.Y - P3.Y) }; //计算x增量，斜率
	double DXright[2] = { (P1.X - P_.X) / (P1.Y - P_.Y), (P_.X - P3.X) / (P_.Y - P3.Y) };

	//记录每条边结束的x值
	double edge_left_ex[] = { P2.X, P3.X };
	double edge_right_ex[] = { P_.X, P3.X };

	//计算t的增量
	double dt_left[] = { 1 / (P2.Y - P1.Y), 1 / (P3.Y - P2.Y) };
	double dt_right[] = { 1 / (P_.Y - P1.Y), 1 / (P3.Y - P_.Y) };

	//用于计算单条扫描线的起始和结束x值
	double Start_x[2] = { P1.X, P2.X };
	double End_x[2] = { P1.X, P_.X };
	double Start_y[2] = { P1.Y, P2.Y };
	double End_y[2] = { P2.Y, P3.Y };

	//保存各条边起始和结束的值
	double left_start_w[2] = { P1.W, P2.W };
	double left_end_w[2] = { P2.W, P3.W };
	double right_start_w[2] = { P1.W, P_.W };
	double right_end_w[2] = { P_.W, P3.W };
	const std::vector<double>* left_start_value[2] = { &P1.ValueArray, &P2.ValueArray };
	const std::vector<double>* left_end_value[2] = { &P2.ValueArray, &P3.ValueArray };
	const std::vector<double>* right_start_value[2] = { &P1.ValueArray, &P_.ValueArray };
	const std::vector<double>* right_end_value[2] = { &P_.ValueArray, &P3.ValueArray };

	std::vector<double> svs(ValueLength); //扫描线起点属性值集合
	std::vector<double> evs(ValueLength); //扫描线终点属性值集合
	std::vector<double> vs(ValueLength);  //当前像素属性值集合
	for (int i = 0; i < 2; i++)			  //i=0和1时分别表示下半三角形和上半三角形
	{
		double dx_left = DXleft[i];
		double dx_right = DXright[i];

		double sx = Start_x[i]; //扫描线起点X值
		double ex = End_x[i];	//扫描线终点X值

		double sy = Start_y[i]; //三角形起始扫描线
		double ey = End_y[i];	//三角形结束扫描线

		double t_left = 0;
		double t_right = 0;

		for (int y = (int)sy; y < ey; y++)
		{
			if (dx_left < 0) //sx随着y增大而减小
			{
				sx = _max(sx, edge_left_ex[i]); //将sx限定为不小于终点x
			}
			else if (dx_left > 0) //sx随着y增大而增大
			{
				sx = _min(sx, edge_left_ex[i]); //将sx限定为不大于终点x
			}
			if (dx_right < 0) //ex随着y增大而减小
			{
				ex = _max(ex, edge_right_ex[i]); //将ex限定为不小于终点x
			}
			else if (dx_right > 0) //ex随着y增大而增大
			{
				ex = _min(ex, edge_right_ex[i]); //将ex限定为不大于终点x
			}

			double lwa = left_start_w[i];
			double lwb = left_end_w[i];

			double rwa = right_start_w[i];
			double rwb = right_end_w[i];
			//本行扫描线起始和结束的ω值
			double sw = (lwa * lwb) / (lwb + (lwa - lwb) * t_left);
			double ew = (rwa * rwb) / (rwb + (rwa - rwb) * t_right);

			for (int vindex = 0; vindex < ValueLength; vindex++) //计算每条扫描线的起始和结束属性值
			{
				double lva = (*left_start_value[i])[vindex];
				double lvb = (*left_end_value[i])[vindex];
				svs[vindex] = (lwb * lva + (lwa * lvb - lwb * lva) * t_left) / (lwb + (lwa - lwb) * t_left);

				double rva = (*right_start_value[i])[vindex];
				double rvb = (*right_end_value[i])[vindex];
				evs[vindex] = (rwb * rva + (rwa * rvb - rwb * rva) * t_right) / (rwb + (rwa - rwb) * t_right);
			}

			double dt = 1 / (ex - sx); //t在扫描线上面的变化率
			double t = 0;
			//sx到ex相当于扫描线的一部分
			for (int x = (int)sx; x <= ex; x++)
			{
				for (int vindex = 0; vindex < ValueLength; vindex++) //计算每个顶点的属性值
				{
					double wa = sw;
					double wb = ew;
					double va = svs[vindex];
					double vb = evs[vindex];
					vs[vindex] = (wb * va + (wa * vb - wb * va) * t) / (wb + (wa - wb) * t);
				}
				graphicsdevice.SetPixel(x, y, FragmentShader(vs));
				t += dt;
			}
			sx = sx + dx_left;
			ex = ex + dx_right;

			t_left += dt_left[i];
			t_right += dt_right[i];
		}
	}
}

```

## [<主页](https://www.wangdekui.com/)
