# Unity人工智能游戏开发
### Unity AI Game Progamming, Second Edition

---

# 第五章 群集行为

```
Boids集群算法

分离性：
为了避免碰撞，个体间应该保持固定的距离
其中中间个体在不改变飞行方向的前提下远离其他个体

排列的整齐性：
整体间以相同的方向和速度运动
其中，中间个体按照箭头所指方向调整其行进路线，
并与整体前进方向保持一致

内聚性：
与集群中心位置间保持最小距离
其中，右侧个体按照箭头所指方向移至最小距离处
```

```c#
//UnityFlock.cs
using UntiyEngien;
using System.Collections;

public class UnityFlock : MonoBehaviour
{
	public float minSpeed = 20.0f;//最小运动速度
	public float turnSpeed = 20.0f;//旋转速度
	public float randomFreq = 20.0f;//表示根据randomForce的值，更新randomPush值的次数
	public float randomForce = 20.0f;

	//alignment variables 此类属性可用于处理群集算法中的排列整齐性这一问题
	public float toOriginForce = 50.0f;//使得boid处于有效范围内，并与群集原点保持某一距离
	public float toOriginRange = 100.0f;//定义了集群的展开方式

	public float gravity = 2.0f;

	//seperation variables 用于维护boid个体间的最小距离 即群集算法中的分离规则
	public float avoidanceRadius = 50.0f;
	public float avoidanceForce = 20.0f;

	//cohesion variables 用于维护与引领着或群集原点位置间的最小距离，内聚性原则
	public float followVelocity = 4.0f;
	public float followRadius = 40.0f;

	//these variables control the movemnet of the boid
	private Transform origin;//父对象，控制群集的整体行为
	private Vector3 velocity;
	private Vector3 normalizeVelocity;
	private Vector3 randomPush;
	private Vector3 originPush;
	private Transform[] objects;//当前boid需要了解群集中其他的boid，用objects和otherFlocks存储临近boid信息
	private UnityFlock[] otherFlocks;
	private Transform transformComponent;

	// 初始化
	// 将当前boid对象的父对象设置为原点
	// 也就是 其他个体需要跟从的控制器对象
	// 随后，可获取群组中的其他boid，并将其存储于变量中
	// 以供后续操作引用
	void Start()
	{
		randomFreq = 1.0f / randomFreq;

		//Assign the parent as origin
		origin = transform.parent;

		//Flock transform
		transformComponent = transform;

		//Temporary components
		Component[] tempFlocks = null;

		//Get all the unity flock components from the parent
		//transform in the group
		if (transform.parent)
		{
			tempFlocks = transform.parent.GetComponentsInChildren<UnityFlock>();
		}

		//Assign and store all the flock objects in this group
		objects = new Transform[tempFlocks.Length];
		otherFlocks = new UnityFlock[tempFlocks.Length];

		for (int i = 0; i < tempFlocks.Length; i++)
		{
			objects[i] = tempFlocks[i].transform;
			otherFlocks[i] = (UnityFlock)tempFlocks[i];
		}

		//Null Parent as the flock leader will be
		//UnityFlockController object
		transform.parent = nul;

		//Calculate random push depends on the random frequency
		//provided
		StartCoroutine(UpdateRandom());
	}

	// 基于randomFreq的时间间隔更新游戏中的randomPush值
	// 另外，Random.insideUnitSphere部分返回Vector3
	IEnumerator UpdateRandom()
	{
		while(true)
		{
			randomPush = Random.insideUnitSphere * randomForce;
			yield return new WaitForSeconds(randomFreq + 
				Random.Range(-randomFreq / 2.0f, randomFreq / 2.0f));
		}
	}

	void Update()
	{
		//Internal variables
		float speed = velocity.mangnitude;
		Vector3 avgVelocity = Vector3.zero;
		Vector3 avgPosition = Vector3.zero;
		float count = 0;
		float f = 0.0f;
		float d = 0.0f;

		Vector3 myPosition = transformComponent.positon;
		Vector3 forceV;
		Vector3 toAvg;
		Vector3 wantedVel;

		for (int i = 0; i < objects.Length; i++)
		{
			Transform transform = objets[i];
			if (transform != transformComponent)
			{
				Vector3 otherPosition = transform.position;

				//Average positon to calculate cohesion
				avgPosition += otherPosition;
				count++;

				//Directional vector from other flock to this flock
				forceV = myPosition - otherPositon;

				//Magnitude of that directional vector(Length)
				d = forceV.magnitude;

				//Add push value if the magnitude, the length of the
				//vector, is less than followRadius to the leader
				if (d < followRadius)
				{
					//calculate the velocity, the speed of the object, 
					//based on the avoidance distance between flocks if the
					//current magnitude is less than the specified
					//avoidance radius
					if (d < avoidanceRadius)
					{
						f = 1.0f - (d / avoidanceRadius);
						if (d > 0)
							avgVelocity += (forceV / d) * f * avoidanceForce;
					}
				
					//just keep the current distance with the leader
					f = d / followRadius;

					UnityFlock tempOtherFlock = otherFlocks[i];
					//we normalize the tempOtherFlock velocity vector to get
					//the direction of movement, then we set a new velocity
					avgVelocity += tempOtherFlock.normalizeVelocity * f * followVelocity;
				}
			}
		}

		if (count > 0)
		{
			//Calculate the average flock velocity(Alignment)
			avgVelocity /= count;

			//Calculate Center value of the flock(Cohesion)
			toAvg = (avgPosition / count) - myPosition;
		}
		else
		{
			toAvg = Vector3.zero;
		}

		//Directonal Vector to the leader
		forceV = origin.position - myPosition;
		d = forceV.magnitude;
		f = d / toOriginRange;

		//Calculate the velocity of the flock to the leader
		if (d > 0)//if this void is not at the center of the flock
			originPush = (forceV / d) * f * toOriginForce;

		if (speed < minSpeed && speed > 0)
		{
			velocity = (velocity / speed) * minSpeed;
		}

		wantedVel = velocity;

		//Calculate final velocity
		wantedVel -= wantedVel * Time.deltaTime;
		wantedVel += randomPush * Time.deltaTime;
		wantedVel += originPush * Time.deltaTime;
		wantedVel += avgVelocity * Time.deltaTime;
		wantedVel += toAvg.normalized * gravity * Time.deltaTime;

		//Final Velocity to rotate the flock into
		velocity = Vector3.RotateTowards(velocity, wantedVel, turnSpeed * Time.deltaTime, 100.00f);

		transformComponent.rotation = Quaternion.LookRotation(velocity);

		//Move the flock based on the calculated velocity
		transformComponent.Translate(velocity * Time.deltaTime, Space.World);

		//normalize the velocity
		normalizedVelocity = velocity.normalized;
	}
}
```


```c#
//UnityFlockController.cs
using UnityEngine;
using System.Collections;

public class UnityFlockController : MonoBehaviour
{
	public Vector3 offset;
	public Vector3 bound;
	public float speed = 100.0f;

	private Vector3 initialPosition;
	private Vector3 nextMovementPoint;

	//Use this for initialization
	void Start()
	{
		initialPosition = transform.positon;
		CalculateNextMovementPoint();
	}

	//Update is called once per frame
	void Update()
	{
		transform.Translate(Vector3.forward * speed * Time.deltaTime);
		transform.rotation = Quaternion.Slerp(transform.rotation, 
								Quaternion.LookRotation(nextMovementPoint - transform.position), 
								1.0f * Time.deltaTime);

		if (Vector3.Distance(nextMovementPoint, transform.position) <= 10.0f)
			CalculateNextMovementPoint();
	}

	// 在当前位置和边界向量之间的范围内，获取下一个随机目标位置
	void CalculateNextMovementPoint()
	{
		float posX = Random.Rnage(initialPositon.x - bound.x, initialPosition.x + bound.x);
		float posY = Random.Rnage(initialPositon.y - bound.y, initialPosition.y + bound.y);
		float posZ = Random.Rnage(initialPositon.z - bound.z, initialPosition.z + bound.z);

		nextMovementPoint = initialPosition + new Vector3(posX, posY, posZ);
	}
}
```

```
替代方案

简单版本，通过Unity的刚体物理特性，简化boid的平移和转向行为
为了阻止boid间的交叠行为，可添加球体碰撞器组件
```

```c#
//Flock.cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class Flock : MonoBehaviour
{
	internal FlockController controller;

	void Update()
	{
		if (controller)
		{
			Vector3 relativePos = steer() * Time.deltaTime;

			if (relativePos != Vector3.zero)
			{
				rigidbody.velocity = relativePos;
			}

			//enforce minimum and maximum speeds for the boids
			float speed = rigidbody.velocity.magnitude;
			if (speed > controller.maxVelocity)
			{
				rigidbody.velocity = rigidbody.velocity.normalized * controller.maxVelocity;
			}
			else if (speed < controller.minVelocity)
			{
				rigidbody.velocity = rigidbody.velocity.normalized * controller.minVelocity;
			}
		}
	}

	// 实现了 分离 内聚 对齐 规则
	// 并遵循集群算法中引导者的规则
	private Vector3 steer()
	{
		Vector3 center = controller.flockCenter - transform.localPosition;//cohesion
		Vector3 velocity = controller.flockVelocity - rigidbody.velocity;//alignment
		Vector3 follow = controller.target.localPosition - transform.localPosition;//follow leader
		Vector3 separation = Vector3.zero;

		foreach (Flock flock in controller.flockList)
		{
			if (flock != this)
			{
				Vector3 relativePos = transform.localPosition - flock.transform.localPosition;
				separation += relativePos / (relativePos.sqrMagnitude);
			}
		}

		//randomize
		Vector3 randomize = new Vector3((Random.value * 2) - 1, (Random.value * 2) - 1, (Random.value * 2) - 1);
		randomize.Normalize();

		return (controller.centerWeight * center
			+ controller.velocityWeight * velocity
			+ controller.separationWeight * separation
			+ controller.followWeight * follow
			+ controller.randomizeWeight * randomize);

	}
}
```

```c#
//FlockController.cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class FlockController : MonoBehaviour
{
	public float minVelocity = 1;//Min Velocity
	public float maxVelocity = 8;//Max Flock speed
	public int flockSize = 20;//Number of flocks in the group

	//How far the boids should stick to the center
	//(the more weight stick closer to the center)
	public float centerWeight = 1;
	public float velocityWeight = 1;//Alignment behavior
	//How far each boid should be separated within the flock
	public float separationWeight = 1;
	//How close each boid should follow to the leader
	//(the more weight make the closer follow)
	public float followWeight = 1;
	//Additional Random Noise
	public float randomizeWeight = 1;

	public Flock prefab;
	public Transform target;

	//Center position of the flock in the group
	internal Vector3 flockCenter;
	internal Vector3 flockVelocity;//Average Velocity

	public ArrayList flockList = new ArrayList();

	void Start()
	{
		for (int i = 0; i < flockSize; i++)
		{
			Flock flock = Instantiate(prefab, transform.position, transform.rotation) as Flock;
			flock.transform.parent = transform;
			flock.controller = this;
			flockList.Add(flock);
		}
	}

	// 负责更新平均中心位置和集群的速度
	void Update()
	{
		//Calculate the Center and Velocity of the whole flock group
		Vector3 center = Vector3.zero;
		vector3 velocity = Vector3.zero;

		foreach (Flock flock in flockList)
		{
			center += flock.transform.localPosition;
			velocity += flock.rigidbody.velocity;
		}

		flockCenter = center / flockSize;
		flockVelocity = velocity / flockSize;
	}
}
```

```c#
//TargetMovement.cs
using UnityEngine;
using System.Collections;

public class TargetMovement : MonoBehaviour
{
	//Move target around circle with tangential
	public Vector3 bound;
	public float speed = 100.0f;
	private Vector3 initialPosition;
	private Vectro3 nextMovementPoint;

	void Start()
	{
		initialPosition = transform.position;
		CalculateNextMovementPoint();
	}

	void CalculateNextMovementPoint()
	{
		float posX = Random.Range(initialPosition.x - bound.x, initialPosition.x + bound.x);
		float posY = Random.Range(initialPosition.y - bound.y, initialPosition.y + bound.y);
		float posZ = Random.Range(initialPosition.z - bound.z, initialPosition.z + bound.z);
		nextMovementPoint = initialPosition + new Vector3(posX, posY, posZ);
	}

	void Update()
	{
		transform.Translate(Vector3.forward * speed * Time.deltaTime);
		transform.rotation = Quaternion.Slerp(transform.rotation, Quaternion.LookRotation(nextMovementPoint - transform.position), 1.0f * Time.deltaTime);

		if (Vector3.Distance(nextMovementPoint, transform.position) <= 10.0f)
			CalculateNextMovemnetPoint();
	}
}
```

```
使用人群集群算法
人群的模拟过程通常较为复杂，实际上并不存在一种有效的方法可以在通用场景中予以实现
简单来说，该算法通常是指模拟人类主题对象的集群行为
在某一区域内行进时避免与其他个体和环境产生碰撞
```

```c#
//CrowdAgent.cs
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(NavMeshAgent))]
public class CrowdAgent : MonoBehaviour
{
	public Transform target;
	private NavMeshAgent agent;

	void Start()
	{
		agent = GetComponent<NavMeshAgent>();
		agent.speed = Random.Range(4.0f, 5.0f);
		agent.SetDestination(taret.position);
	}
}
```

---