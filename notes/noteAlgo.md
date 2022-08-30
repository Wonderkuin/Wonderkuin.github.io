# 记录一些算法

---

```python
#普通二次曲线
count = 0
time = 0.5
while(true):
    count += deltaTime
    r = 1 - count / time
    r = 1 - r * r
    start = start + (end - start) * r
    if count > time:
        start = end
        break
```

---

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

---

#### A* 寻路

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

---