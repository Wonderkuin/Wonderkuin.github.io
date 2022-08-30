# C# and Shader Tutorials

---

# C#与Unity基础

---

## 游戏物体与脚本

```c#
//钟表模拟
const float degreesPerHour = 30f, degreesPerMinute = 6f, degreesPerSecond = 6f;

//不平滑移动
DateTime time = DateTime.Now;
hourTransform.localRotation = Quaternion.Euler(0f, time.Hour * degreesPerHour, 0f);
minutesTransform.localRotation = Quaternion.Euler(0f, time.Minute * degreesPerMinute, 0f);
secondsTransform.localRotation = Quaternion.Euler(0f, time.Second * degreesPerSecond, 0f);

//平滑移动
TimeSpan time = DateTime.Now.TimeOfDay;
hoursTransform.localRotation = Quaternion.Euler(0f, time.TotalHours * degreesPerHour, 0f);
minutesTransform.localRotation = Quaternion.Euler(0f, time.TotalMinutes * degreesPerMinute, 0f);
secondsTransform.localRotation = Quaternion.Euler(0f, time.TotalSeconds * degreesPerSecond, 0f);
```

---

## 组成图形

```c++
//显示数学函数曲线

//自定义Shader
Shader "Custom/ColoredPoint" {
  Properties {
      _Color ("Color", Color) = (1,1,1,1)
      _MainTex ("Albedo (RGB)", 2D) = "white" {}
      _Glossiness ("Smoothness", Range(0,1)) = 0.5
      _Metallic ("Metallic", Range(0,1)) = 0.0
   }
  SubShader {
      Tags { "RenderType"="Opaque" }
      LOD 200

      CGPROGRAM
      #pragma surface surf Standard fullforwardshadows
      #pragma target 3.0
      sampler2D _MainTex;
      struct Input {
         float2 uv_MainTex;
      };
      half _Glossiness;
      half _Metallic;
      fixed4 _Color;
      UNITY_INSTANCING_CBUFFER_START(Props)
      UNITY_INSTANCING_CBUFFER_END
      void surf (Input IN, inout SurfaceOutputStandard o) {
         fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
         o.Albedo = c.rgb;
         o.Metallic = _Metallic;
         o.Smoothness = _Glossiness;
         o.Alpha = c.a;
      }
      ENDCG
   }
  FallBack "Diffuse"
}
```

```c++
//一个材质通过漫反射 diffuse reflectivity而表现出的颜色被称为
//albedo 反射色
//Albedo是一个拉丁文单词，含义是 白度 whiteness
//它描述了材质可以漫反射多少红绿蓝三种颜色通道 color channel的色彩

//Alpha用来衡量不透明度 opacity

//修改Shader为不透明黑色
Shader "Custom/ColoredPoint" {
  Properties {
      _Glossiness ("Smoothness", Range(0,1)) = 0.5
      _Metallic ("Metallic", Range(0,1)) = 0.0
   }
  SubShader {
      Tags { "RenderType"="Opaque" }
      LOD 200

      CGPROGRAM
      #pragma surface surf Standard fullforwardshadows
      #pragma target 3.0
      struct Input {
	float3 worldPos;
      };
      half _Glossiness;
      half _Metallic;
      UNITY_INSTANCING_CBUFFER_START(Props)
      UNITY_INSTANCING_CBUFFER_END
      void surf (Input IN, inout SurfaceOutputStandard o) {
        //o.Albedo = c.rgb;

	// 通过世界位置改变颜色
	// 不需要设置 绿色 蓝色 色值，因为它们在 surf被调用前，已经被设置为0
	o.Albedo.r = IN.worldPos.x;
	// x坐标是负数的对象，变成了黑色，因为颜色色值不能是负数，最小是0
	// rgb都是0就是黑色 为了让r通道 可以随着 -1 到 1的坐标区间变化
	// 我们需要在代码中将x坐标减半再加上0.5，使 r的值从 -1 1 变成 0 2
	o.Albedo.r = In.worldPos.x * 0.5 + 0.5


	// y来改变绿色值
	o.Albedo.g = IN.worldPos.y * 0.5 + 0.5
	// rg 一起
	o.Albedo.rg = IN.worldPos.xy * 0.5 + 0.5

         o.Metallic = _Metallic;
         o.Smoothness = _Glossiness;
        //o.Alpha = c.a;
	// 不透明
         o.Alpha = 1;
      }
      ENDCG
     }
  FallBack "Diffuse"
}
```

```c#
//显示正弦曲线
Vector3 position = point.localPosition;
position.y = Mathf.Sin(Mathf.PI * position.x);
point.localPosition = position;

//使图形动起来
position.y = Mathf.Sin(Mathf.PI * (position.x + Time.time));
```

---

## 数学曲面

```c#
//改变正弦曲线
x = position.x;
t = Time.deltaTime;
float y = Mathf.Sin(Mathf.PI * (x + t));
y += Mathf.Sin(2f * Mathf.PI * (x + t)) / 2f;
y *= 2f / 3f;

//数学函数为
//f(x,t) = sin(pi(x+t)) + [ sin(2pi(x+t))] / 2
//正弦函数取值范围是 -1 1
//这个函数范围是 -1.5 1.5
//因此结果要限制回到 -1 1

//大弦套小弦，使小弦运动，只要把小弦的周期改变就行了
y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;

//这种数学计算函数是 自包含的 self-contained
//可以声明为 static
//静态方法不与类的实例关联，程序编译并运行后，将不会去跟踪实例，
//这意味着静态方法调用速度更快，不过这种速度差别大多数情况不值一提
```

```c#
//委托
public delegate flaot GraphFunction(flaot x, float t);
GraphFunction f;
f = SineFunction;
GraphFunction[] functions = { SineFunction, MultiSineFunction };

public enum GraphFunctionName {
   Sine,
   MultiSine
}

GraphFunctionName function = GraphFunctionName.Sine;
GraphFunction f = functions[(int)function];
```

```c#
//增加一个维度
//空间维度
//o.Albedo.rg = IN.worldPos.xy * 0.5 + 0.5;
o.Albedo.rgb = IN.worldPos.xyz * 0.5 + 0.5;

//public delegate float GraphFunction (float x , float t);
public delegate float GraphFunction(float x, float z, float t);

//static float SineFunction (float x, float t)
static float SineFunciton (float x, float z, float t) {
   return Mathf.Sin(Mathf.PI * (x + t));
}

//static float MultiSineFunction (float x, float t) {
static flaot MultiSineFunction (float x, float z, float t){
   float y = Mathf.Sin(Mathf.PI (x + t));
   y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
   y *= 2f / 3f;
   return y;
}

//positon.y = f(position.x, t);
position.y = f(positon.x, position.z, t);
```

```c#
//构成网格图形
//points = new Transform[resolution];
points = new Transform[resolution * resolution];

//for (int i = 0; i < points.Length; i++)
for (int i = 0, x = 0; i < points.Length; i++, x++)
{
   //新增的if判断语句 如果x与resolution相等 x重置为0
   if (x == resolution)
      x = 0;
   Transform point = Instantiate(pointPrefab);
   //position.x = (i + 0.5f) * step - 1f;
   // 使用x代替i进行cube的横坐标计算
   position.x = (x + 0.5f) * step - 1f;
   position.localPosition = position;
   point.localScale = scale;
   point.SetParent(transform, false);
   points[i] = point;
}

// 下一步 要将每行曲线都沿着z轴进行位置偏移
//for (int i = 0, x = 0; i < points.Length; i++, x++)
for (int i = 0, x = 0, z = 0; i < points.Length; i++, x++)
{
   if (x == resolution)
   {
      x = 0;
      z++;
   }
   // 用z变量计算和设置z轴坐标
   position.z = (z + 0.5f) * step - 1f;
}
// 至此，一条线变成了一个平面
// 可以调整光源角度 将光源阴影去掉
```

```c#
// 双重循环
for (int i = 0, z = 0; z < resolution; z++)
{
   position.z = (z + 0.5f) * step - 1f;
   for (int x = 0; x < resolution; x++, i++)
   {
      Transform point = Instantiate(pointPrefab);
      position.x = (x + 0.5f) * step - 1f;
      point.localPosition = position;
      point.localScale = scale;
      point.SetParent(transform, false);
      points[i] = point;
   }
}
```

```c#
// 新的模拟
const float pi = Mathf.PI;

//f(x, z, t) = sin(pi (x+z+t))
static float Sine2DFunction(float x, float z, float t) {
   return Mathf.Sin(pi * (x + z + t));
}

//f(x, z, t) = [sin(pi(x + t)) + sin(pi(z + t))] * 0.5
static float Sine2DFunction(float x, flaot z, float t) {
   flaot y = Mathf.Sin(pi * (x + t));
   y = Mathf.Sin(pi * (z + t));
   y *= 0.5f;
   return y;
}

//M = sin(pi ( x + z + t/2))
//Sx = sin(pi(x + t))
//Sz = sin(2pi(z + 2t))
//f(x, z, t) = 4M + Sx + Sz / 2
static float MultiSine2DFunction(float x, float z, float t)
{
   float y = 4f * Mathf.Sin(pi * (x + z + t * 0.5f));
   y += Mathf.Sin(pi * (x + t));
   y += Mathf.Sin(2f * pi * (z + 2f * t)) * 0.5f;
   y *= 1f / 5.5f;
   return y;
}
```

```c#
//产生涟漪
// 想让涟漪向所有维度分散传播
// 需要创建一个基于原点距离而变化的正弦波动
// 这个距离可以使用勾股定理计算
static float Ripple(float x, float z, float t)
{
   float d = Mathf.Sqrt(x * x + z * z);
   return y;
}

// 为了创建出涟漪图形，需要使用函数f(x,z,t)=sin(pi D)
// D代表原点距离
//float y = d;
float y = Mathf.Sin(pi * d);

// 只有一次波动不够，将波动频率增加四倍
//float y = Mathf.Sin(pi * d);
float y = Mathf.Sin(4f * pi * d);

// 现在波动有点剧烈，需要降低幅度
// 使用 1/10D作为振幅
float y = Mathf.Sin(4f * pi * d);
y /= 1f + 10f * d;

// 将正弦加入时间参数
//float y = Mathf.Sin(4f * pi * d);
float y = Mathf.Sin(pi * (4f * d - t));
```

```c#
// 立体图形
// 通过xz计算y，没有两个点可以具有相同的xz又有不同的y
// 这意味着图形表面的曲率 curvature 是有限制的
// 图形不能出现大于直角的表面弯曲，为了打破这个限制
// 就需要函数方法不止返回y坐标，还需要返回x和z坐标

// 三维函数
// 如果函数可以返回三维位置而不是只有一个维度的位置，我们就可以用这个
// 函数创造任意形状的表面。f(u,v)=[u+v, uv, u/v]
public delegate float GraphFunction(float u, float v, float t);

//f(u,v,t) = [u, sin(pi(u+t)), v]
static Vector3 SineFunction(float x, flaot z, float t)
{
   float y = Mathf.Sin(pi * (x + t));
   return new Vector3(x, y, z);
}

static Vector3 Sine2DFunction(float x, float z, float t)
{
   float y = Mathf.Sin(pi * (x + t));
   y += Mathf.Sin(pi * (z + t));
   y *= 0.5f;
   return new Vector3(x, y, z);
}

static Vector3 MultiSineFunction(float x, float z, float t)
{
   float y = Mathf.Sin(pi * (x + t));
   y += Mathf.Sin(2f * pi * (x + 2f * t)) / 2f;
   y *= 2f / 3f;
   return new Vector3(x, y, z);
}

static Vector3 MultiSine2DFunction(float x, float z, float t)
{
   float y = 4f * Mathf.Sin(pi * (x + z + t / 2f));
   y += Mathf.Sin(pi * (x + t));
   y += Mathf.Sin(2f * pi * (z + 2f * t)) * 0.5f;
   y *= 1f / 5.5f;
   return new Vector3(x, y, z);
}

static Vector3 Ripple(float x, float z, float t)
{
   float d = Mathf.Sqrt(x * x + z * z);
   float y = Mathf.Sin(pi * (4f * d - t));
   y /= 1f + 10f * d;
   return new Vector3(x, y, z);
}

// uv带序号，点在平面上位于哪行哪列
void Update()
{
   float t = Time.time;
   GraphFunction f = functions[(int)function];
   float step = 2f / resolution;
   for (int i = 0, z = 0; z < resolution; z++)
   {
      float v = (z + 0.5f) * step - 1f;
      for (int x = 0; x < resolution; x++, i++)
      {
         float u = (x + 0.5f) * step - 1f;
         points[i].localPosition = f(u, v, t);
      }
   }
}

void Awake()
{
   float step = 2f / resolution;
   Vector3 scale = Vector3.one * step;
   points = new Transform[resolution * resolution];
   for (int i = 0; i < points.Length; i++)
   {
      Transform point = Instantiate(pointPrefab);
      point.localScale = scale;
      point.SetParent(transform, false);
      points[i] = point;
   }
}
```

```c#
// 生成圆柱
static Vector3 Cylinder(float u , float v, float t) {
   return Vector3.zero;
}

//圆柱体可以看成一个被拉伸了的圆形，所以首先生成一个圆形
//圆形上的所有点可以通过 [sinA cosA]的位置进行定义
//随着角度A从0变化到2pi，就能得到圆上所有坐标
static Vector3 Cylinder(float u , float v, float t) {
   Vector3 p;
   p.x = Mathf.Sin(pi * u);
   p.y = 0f;
   p.z = Mathf.Cos(pi * u);
   return p;
}
//将y设置为u 得到一条曲线
//将y设置为v 得到很多圆形堆叠起来的圆柱体

// 半径通过缩放正弦和余弦的幅度来调整
// f(u,v) = [R sin(pi * u), v, R cos(pi * u)]
float r = 1f;//半径
p.x = r * Mathf.Sin(pi * u);
p.y = v;
p.z = r * Mathf.Cos(pi * u);

// 使用不同振幅
// R = 1 + sin(6 pi r) / 5
float r = 1f + Mathf.Sin(6f * pi * u) * 0.2f;
// 圆柱体变成了六角星一样的形状

// 让半径和v有关
float r = 1f + Mathf.Sin(6f * pi * v) * 0.2f;
// 圆柱体的边被扭曲

// 加入时间
float r = 0.8f + Mathf.Sin(pi * (6f * u + 2f * v + t)) * 0.2f;
// 旋转六角星
```

```c#
// 生成球面
// 可以将圆柱两侧的圆逐渐减小半径到0 获得球
// 半径计算公式 R = cos ( pi * v / 2)
static Vector3 Sphere(float u, float v, float t)
{
   Vector3 p;
   flaot r = Mathf.Cos(pi * 0.5f * v);
   p.x = r * Mathf.Sin(pi * u);
   p.y = v;
   p.z = r * Mathf.Cos(pi * u);
   return p;
}
// 几乎是一个球，但两边太尖了
//半径的波动变化还不是一个圆形，这是因为圆需要正弦和余弦共同构成
//将y坐标赋值为 R = sin( pi * v / 2)
p.y = Mathf.Sin(pi * 0.5f * v);

//我们得到了一个球，这就是UV-Sphere UV球面
//此时球面上的点分布不均匀

// 为了能够控制半径，我们需要调整函数公式，使用
// f(u,v) = [Ssin(pi u), Rsin(pi v /2), Scos(pi u)]
// S = Rcos(pi v / 2) R是半径
// 通过改变R的值可以让球面的半径发生动态变化，让我们使用u和v的
// 正弦曲线设置半径
// R = 4/5 + sin(pi(6u + t))/10 + sin(pi(4v + t))/10
float r = 0.8f + Mathf.Sin(pi * (6f * u + t)) * 0.1f;
r += Mathf.Sin(pi * (4f * v + t)) * 0.1f;
float s = r * Mathf.Cos(pi * 0.5f * v);
p.x = s * Mathf.Sin(pi * u);
p.y = r * Mathf.Sin(pi * 0.5f * v);
p.z = s * Mathf.Cos(pi * u);
// 有点复杂 没理解
```

```c#
// 生成环面
static Vector3 Torus(float u , float v, float t) {
   Vector3 p;
   // 把球面拉开，变成环面
   float s = Mathf.Cos(pi * 0.5f * v) + 0.5f;
   p.x = s * Mathf.Sin(pi * u);
   p.y = Mathf.Sin(pi * 0.5f * v);
   p.z = s * Mathf.Cos(pi * u);
   return p;
}

// 上面只是半成品的环面
// 需要把 pi * v / 2 都替换成 pi * v
float s = Mathf.Cos(pi * v) + 0.5f;
p.x = s * Mathf.Sin(pi * u);
p.y = Mathf.Sin(pi * v);
p.z = s * Mathf.Cos(pi * u);

// 只把球面拉开了0.5 因此很紧
// 这叫做 自交形状 self-intersecting shape
// 当前的形状也被称为 纺锤环面 spindle torus
// 拉开一个单位远，会得到一个没有自交的环面
// 但环面中间依然没有空洞，这称为 角环面
// 球形被拉开多远直接影响环面的样子，准确的说，这个拉开的距离
// 定义了环面的整体半径，我们将其称为R1
// f(u,v) = [ Ssin(pi u) sin(pi u) Scos(pi u)]
// S = cos(pi u) + R1
float r1 = 1f;
float s = Mathf.Cos(pi * v) + r1;

// R1大于1时，会在中心产生空洞，这时的环面叫做 环状环面
// 此时，环绕着环面中心的圆形管道的半径记作R2
// f(u,v) = [Ssin(pi u) R2sin(pi v) Scos(pi u)]
// S = R2cos(pi v) + R1
// 在R1为1时，将R2设置为0.5
float r1 = 1f;
float r2 = 0.5f;
float s = r2 * Mathf.Cos(pi * v) + r1;
p.x = s * Mathf.Sin(pi * u);
p.y = r2 * Mathf.Sin(pi * v);
p.z = s * Mathf.Cos(pi * u);

// 现在，我们有了两个半径可以设置 从而改变环形
// 一个不难但是很有趣的做饭是用u输入的波动影响R1
// 用v输入的波动影响R2，让这两个半径同时动态变化
// 同时确保环形的尺寸在-1到1之间
float r1 = 0.65f + Mathf.Sin(pi * (6f * u + t)) * 0.1f;
float r2 = 0.2f + Mathf.Sin(pi * (4f * v + t)) * 0.05f;
// 旋转波动的六角星形状
```

---

## 构造分型

```c#
// 分形 Fractal 神秘又美丽

using UnityEngine;
using System.Collections;
public class Fractal : MonoBehaviour {
   public Mesh mesh;
   public Material material;

   public int maxDepth;
   public int depth;

   public float childScale;

   private void Start() {
      gameObejct.AddComponent<MeshFilter>.mesh = mesh;
      gameObejct.AddComponent<MeshRenderer>.material = material;

      GetComponent<MeshRenderer>().material.color = Color.Lerp(Color.white, Color.yellow, (float)depth / maxDepth);

      if (depth < maxDepth)
      {
         StartCoroutine(CreateChildren());
      }
   }

   private IEnumerator CreateChildren() {
      yield return new WaitForSeconds(0.5f);
      new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Vector3.up, Quaternion.identity);
      yield return new WaitForSeconds(0.5f);
      new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Vector3.right, Quaternion.Euler(0f, 0f, -90f));
      yield return new WaitForSeconds(0.5f);
      new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Vector3.left, Quaternion.Euler(0f, 0f, 90f));
   }

   private void Initialize(Fractal parent, Vector3 direction, Quaternion orientation)
   {
      mesh = parent.mesh;
      material = parent.material;
      maxDepth = parent.maxDepth;
      depth = parent.depth + 1;

      childScale = parent.childScale;
      transform.parent = parent.transform;
      transform.localScale = Vector3.one * childScale;

      transform.localPosition = direction * (0.5f + 0.5f * childScale);

      transfrom.localRotation = orientation;
   }
}

// 动态合并批次 Material[] 需要从父对象传递到子对象，不要一直new
// 为什么不使用 static Material[] 因为数组长度不固定

private void InitializeMaterials() {
   materials = new Material[maxDepth + 1];
   for (int i = 0; i <= maxDepth; i++) {
      float t = i / (maxDepth - 1f);
      //将参数2次方计算，以便让每个材质之间颜色差距增大
      t *= t;
      materials[i] = new Material(material);
      materials[i].color = Color.Lerp(Color.white, Color.yellow, t);
   }
   // 设置最后一层是洋红色
   materials[maxDepth].color = Color.magenta;
}
```

---

## 每秒帧数

```c#
// 建立原子核
using UnityEngine;
[RequireComponent(typeof(Rigidbody))]
public class Nucleon : MonoBehaviour {
   public float attractionForce; //代表引力的字段
   Rigidbody body; //代表刚体的字段
   void Awake() {
      body = GetComponent<Rigidbody>();
   }
   void FixedUpdate() {
      body.AddForce(transform.localPosition * -attractonForce);
   }
}

// 生成原子核
using UnityEngine;
public class NucleonSpawner : MonoBehaviour {
   public float timeBetweenSpawns; //生成时间间隔
   public float spawnDistance; //生成的位置距离
   public Nucleon[] nucleonPrefabs;

   float timeSinceLastSpawn; //上一次产生核体后经过的时间

   void FixedUpdate() {
      timeSinceLastSpawn += Time.deltaTime;
      if (timeSinceLastSpawn >= timeBetweenSpawns) {
         timeSinceLastSpawn -= timeBetweenSpawns;
         SpawnNucleon();
      }
   }

   void SpawnNucleon() {
      Nucleon prefab = nucleonPrefab[Random.Range(0, nucleonPrefabs.Length)];
      Nucleon spawn = Instantiate<Nucleon>(prefab);
      spawn.transform.localPosition = Random.onUnitSphere * spawnDistance;
   }
}

//对于简单场景, 关闭垂直同步后会提升很高的帧率, 远超100. 这会给硬件带来不必要的压力. 
//你可以通过代码, 设置Application.targetFrameRate属性来强制规定一个最大帧率从而避免发生这种情况. 
//需要注意的是, 如果通过代码设置了帧率最大值, 即使退出运行模式后该帧率限制也会被保存在编辑组数据中. 
//将帧率最大值设置为-1则表示取消限制.
//(注意, 该代码设置的帧率限制在编辑器状态下是不生效的, 它只会在打包后的程序中运行才会生效, 本例中不需要设置这个代码, 不影响教程的进行)
//(还有点要特别注意, 如果你已经在代码中使用Application.targetFrameRate设置了一个具体的帧数限制, 
//此时会自动生效垂直同步机制, 所以你在Profiler中会依然看到垂直同步所占用的CPU资源比例, 
//你需要去修改你的代码, 将Application.targetFrameRate设置为-1来, 就可以解决这个问题)
```

```c#
//FPScounter.cs
using UnityEngine;
public class FPSCounter : MonoBehaviour {
   // 这种简写无法被Unity序列化
   public int FPS { get; private set;}
}

// 等价于
private int fps;
public int FPS {
   get { return fps; }
   private set { fps = value; }
}

void Update() {
   //FPS = (int)(1f / Time.deltaTime);
   FPS = (int)(1f / Time.unscaledDeltaTime);
}


// 字符串优化
//新建字符串数组, 存储00到99的数字字符串
static string[] stringsFrom00To99 = {
 	"00", "01", "02", "03", "04", "05", "06", "07", "08", "09",
 	"10", "11", "12", "13", "14", "15", "16", "17", "18", "19",
 	"20", "21", "22", "23", "24", "25", "26", "27", "28", "29",
 	"30", "31", "32", "33", "34", "35", "36", "37", "38", "39",
 	"40", "41", "42", "43", "44", "45", "46", "47", "48", "49",
 	"50", "51", "52", "53", "54", "55", "56", "57", "58", "59",
 	"60", "61", "62", "63", "64", "65", "66", "67", "68", "69",
 	"70", "71", "72", "73", "74", "75", "76", "77", "78", "79",
 	"80", "81", "82", "83", "84", "85", "86", "87", "88", "89",
 	"90", "91", "92", "93", "94", "95", "96", "97", "98", "99"
};

void Update () {
    //fpsLabel.text = Mathf.Clamp(fpsCounter.FPS, 0, 99).ToString();
    //计算出来的帧率正好对应其字符串数组中同样内容元素的索引
 	fpsLabel.text = stringsFrom00To99[Mathf.Clamp(fpsCounter.FPS, 0, 99)];
}


// 平均每秒帧数
int[] fpsBuffer;
int fpsBufferIndex;

void InitializeBuffer() {
   if (frameRange <= 0)
      frameRange = 1;
   fpsBuffer = new int[frameRange];
   fpsBufferIndex = 0;
}

void Update() {
   if (fpsBuffer == null || fpsBuffer.Length != frameRange) {
      InitializeBuffer();
   }
   UpdateBuffer();
   CalculateFPS();
}

void UpdateBuffer() {
   fpsBuffer[fpsBufferIndex++] = (int)(1f / Time.unscaledDeltaTime);
   if (fpsBufferIndex >= frameRange)
      fpsBufferIndex = 0;
}

void CalculateFPS() {
   int sum = 0;
   for (int i = 0; i < frameRange; i++) {
      sum += fpsBuffer[i];
   }
   AverageFPS = sum / frameRange;
}
```

---

# 移动控制

## 滑动球体

```c#
//MovingSphere.cs
using UnityEngine;
public class MovingSphere : MonoBehaviour {
   [SerizlizeField, Range(0f, 100f)]
   float maxSpeed = 10f;

   void Update() {
      Vector2 playerInput;
      //playerInput.x = 0f;
      playerInput.x = Input.GetAxis("Horizontal");
      //playerInput.y = 0f;
      playerInput.y = Input.GetAxis("Vertical");

      // 归一化输入向量
      //playerInput.Normalize();

      // 约束输入
      playerInput = Vector2.ClampMagnitude(playerInput, 1f);

      // 相对运动
      //Vector3 displacement = new Vector3(playerInput.x, 0f, playerInput.y);
      // 不受帧率影响
      Vector3 velocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
      Vector3 displacement = velocity * Time.deltaTime;
      //transform.localPositon = new Vector3(playerInput.x, 0.5f, playerInput.y);
      transform.localPosition += displacement;
   }
}

// 直接控制速度，速度都是瞬时变化，现实中速度是平滑变化的，需要使用加速度
public class MovingSphere : MonoBehaviour {
   Vector3 velocity;
   void Update() {
      Vector3 acceleration = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
      velocity += acceleration * Time.deltaTime;
   }
}

//控制加速度之后，降低了对于球体的控制性，多数游戏中，需要更直接的控制速度
//我们可以使用加速度控制速度，知道速度达到所需要的值
public class MovingSphere : MonoBehaviour {
   [SerializeField, Range(0f, 100f)]
   float maxAcceleration = 10f;
   void Update() {
      //Vector3 acceleration = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
      //velocity += acceleration * Time.deltaTime;
      //代表我们要达到的目标速度
      Vector3 desiredVelocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
      //在desiredVelocity下面加入一行代码来获取本次update的运动速度改变值
      float maxSpeedChange = maxAcceleration * Time.deltaTime;
      //处理x轴上的速度
      /*
      if (velocity.x < desiredVelocity.x) {
         velocity.x = Mathf.Min(velocity.x + maxSpeedChange, desireVelocity.x);
      }
      else if (velocity.x > desiredVelocity.x) {
         velocity.x = Mathf.Max(velocity.x - maxSpeedChaneg, desireVelocity.x);
      }
      */
      velocity.x = Mathf.MoveTowards(velocity.x, desiredVelocity.x, maxSpeedChange);
      velocity.z = Mathf.MoveTowards(velocity.z, desiredVelocity.z, maxSpeedChange);

      Vector3 displacement = velocity * Time.deltaTime;
      transform.localPositon += displacement;
   }
}


// 约束位置
public class MovingSphere : MonoBehaviour {
   [SerializeField]
   Rect allowedArea = new Rect(-5f, -5f, 10f, 10f);
   void Update() {
      Vector3 newPosition = transform.localPosition + displacement;

      if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z)) {
         newPosition = transform.localPosition;
      }

      transform.localPosition = newPosition;
   }
}


//精确定位
//我们现在可以通过 只限制会超出边缘的运动方向上的坐标 来取代 完全不进行移动，修复现在的问题
if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z)) {
   newPosition.x = Mathf.Clamp(newPosition.x, allowedArea.xMin, allowedArea.xMax);
   newPosition.z = Mathf.Clamp(newPosition.z, allowedArea.yMin, allowedArea.yMax);
}


//消除速度
//球体停靠在边缘，接触边缘后 离开会有延迟，因为速度还在，要清理指向边缘方向的速度
//if (!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z)) {
//   newPosition.x = Mathf.Clamp(newPosition.x, allowedArea.xMin, allowedArea.xMax);
//   newPosition.z = Mathf.Clamp(newPosition.z, allowedArea.yMin, allowedArea.yMax);
//}
if (newPosition.x < allowArea.xMin) {
   newPosition.x = allowArea.xMin;
   velocity.x = 0f;
}
else if (newPosition.x > allowArea.xMax) {
   newPosition.x = allowArea.xMax;
   velocity.x = 0f;
}
if (newPosition.z < allowArea.yMin) {
   newPosition.z = allowArea.yMin;
   velocity.z = 0f;
}
else if (newPosition.z > allowedArea.yMax) {
   newPosition.z = allowArea.yMax;
   velocity.z = 0f;
}


//反弹
if (newPosition.x < allowArea.xMin) {
   newPosition.x = allowArea.xMin;
   //velocity.x = 0f;
   velocity.x = -velocity.x;
}
else if (newPosition.x > allowArea.xMax) {
   newPosition.x = allowArea.xMax;
   //velocity.x = 0f;
   velocity.x = -velocity.x;
}
if (newPosition.z < allowArea.yMin) {
   newPosition.z = allowArea.yMin;
   //velocity.z = 0f;
   velocity.z = -velocity.z;
}
else if (newPosition.z > allowedArea.yMax) {
   newPosition.z = allowArea.yMax;
   velocity.z = 0f;
   velocity.z = -velocity.z;
}


//弹性削减
float bounciness = 0.5f;
velocity.x = -velocity.x * bounciness;
// 同理
```

---

## 模拟物理

```c#
Rigidbody body;
void Awake() {
   body = GetComponent<Rigidbody>();
}
void Update() {
   velocity = body.velocity

   velocity.x = Mathf.MoveTowards(velocity.x, desiredVelocity.x, maxSpeedChange);
   velocity.z = Mathf.MoveTowards(velocity.z, desiredVelocity.z, maxSpeedChange);

   body.velocity = velocity;
}
// 去掉滑动摩擦力
// 锁定三个轴的旋转，使小球完全滑动而不是滚动

//可以在FixedUpdate方法中使用Time.deltaTime吗?
//可以.
//当FixedUpdate方法被调用时, TIme.deltaTime的值与Time.fixedDeltaTime的值相等
```

```c#
//跳跃
bool desiredJump;
void Update() {
   desiredJump |= Input.GetButtonDown("Jump");
}
void FixedUpdate() {
   if (desiredJump) {
      desiredJump = false;
      Jump();
   }
   body.velocity = velocity;
}
void Jump() {
   velocity.y += 5f;
}


//跳跃高度
//我们可以直接控制跳跃速度从而限制其跳跃高度，但是这样做并不能直观地感受到
//限制了多少跳跃高度，更方便的一种方法是直接控制跳跃高度本身
[SerizlizeField, Range(0f, 10f)]
float jumpHeight = 2f;
//跳跃需要克服重力，所以垂直方向需要多少速度取决于重力，计算公式为
// Vy = sqrt(-2gh)
void Jump() {
   velocity.y += Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
}

// 达到目标高度所需的速度公式是怎么算出来的
//自由落体 h = gtt/2
//速度等于加速度乘以时间 v = gt  => t = v/g
//带入得 h = vv/2g => v = sqrt(2gh)


//在地面才允许跳跃
bool onGround;

//void OnCollisionEnter() {
void OnCollisionStay() {
   onGround = true;
}

void OnCollisionExit() {
   onGround = false;
}

// 每次物理模拟都会先调用所有的FixedUpdate，之后才调用碰撞相关的方法
void FixedUpdate() {
   onGround = false;
}


//不因接触墙壁而允许跳跃
// 检查接触点的法线向量
void OnCollisionEnter(Collision collision) {
   //onGround = true;
   EvaluateCollision(collision);
}
void OnCollisionStay(Collision collision) {
   //onGround = false;
   EvaluateCollision(collision);
}
void EvaluateCollision(Collision collision) {
   for (int i = 0; i < collision.contactCount; i++) {
      Vector3 normal = collision.GetContact(i).normal;
      //如果接触点的法线y值大于等于0.9，则认为球体接触的是地面
      onGround |= normal.y >= 0.9f;
   }
}


//空中跳跃
[SerializeField, Range(0,5)]
int maxAirJumps = 0;
int jumpPhase; //跳跃阶段

void FixedUpdate() {
   //velocity = body.velocity;
   UpdateState();
}
void UpdateState() {
   velocity = body.velocity;
   //如果球体位于地面，则设置jumpPhase为0，表示一次空中连跳也没跳过
   if (onGound) {
      jumpPhase = 0;
   }
}

void Jump() {
   //为什么是小于，而不是小于等于？
   //每次重新接触地面的时候都会将jumpPhase设置为0，后面会解释为什么
   if (onGround || jumpPhase < maxAirJump) {
      //每次跳跃时jumpPhase加1
      jumpPhase += 1;
      velocity.y += Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
   }
}


//限制向上速度
//快速连续空中跳跃会给予球体比较大的向上速度
//从而比一次跳跃的更高
//我们接下来要控制跳跃速度不会超过单次跳跃可以达到的速度
//第一步需要在jump方法中将跳跃速度的计算结果存储到一个变量jumpSpeed中，单独计算跳跃速度
void Jump() {
   if (onGround || jumpPhase < maxAirJump) {
      jumpPhase += 1;
      //velocity.y += Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
      //上面的代码被拆分为下面两部分代码
      float jumpSpeed = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);

      // 如果球体已经有了一个向上的速度，那么在将其新的跳跃速度加给球体前
      // 需要先减去球体当前的向上速度，这样球体就不会超过设置的向上速度了
      if (velocity.y > 0f) {
         //但是如果我们的向上速度已经比计算出来的跳跃速度还大
         //我们不希望会导致球体在跳跃过程中被二段跳减速
         // 其实这个方法多此一举
         //jumpSpeed = jumpSpeed - velocity.y;
         jumpSpeed = Mathf.Max(jumpSpeed - velocity.y, 0f);
      }

      velocity.y += jumpSpeed;
   }
}


//空中移动
// 让球体在空中时难以操控，添加一个最大空中加速度，将球体在空中时的操控性变得可配置
[SerializeField, Range(0f, 100f)]
float maxAirAcceleration = 1f;
// 现在有了最大加速度和最大空中加速度
float acceleration = onGround ? maxAcceleration : maxAirAcceleration;
float maxSpeedChange = acceleration * Time.deltaTime;
```

## 表面接触

```c#
// 斜坡

//地面的最大倾斜角度
[SerializeField, Range(0, 90)]
float maxGroundAngle = 25f;

//点积
//A·B = ||A|| ||B|| cos(theta)
float minGroundDotProduct;
void OnValidate() {
   minGroundDotProduct = Mathf.Cos(maxGroundAngle * Mathf.Deg2Rad);
}
void Awake() {
   body = GetComponent<Rigidbody>();
   OnValidate();
}

//判断是否在地面
// onGround |= normal.y >= 0.9f;
onGround |= normal.y >= minGroundDotProduct;

//存储法线
Vector3 contactNormal;
void EvaluateCollision(Collision collision) {
   for (int i = 0; i < collision.contactCount; i++) {
      Vector3 normal = collision.GetContact(i).normal;
      //onGround |= normal.y >= minGroundDotProduct;
      if (normal.y >= minGournDotProduct) {
         onGround = true;
         contactNormal = normal;
      }
   }
}

//不接触地面的情况下空中跳跃，这时用向上方向作为法线
void UpdateState() {
   velocity = body.velocity;
   if (onGround) {
      jumpPhase = 0;
   }
   else {
      contactNormal = Vector3.up;
   }
}

// 跳跃 使用法线
void jump() {
   if (onGround || jumpPhase < maxAirJumps) {
      jumpPhase += 1;
      //velocity.y += Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
      //上面的代码被拆分为下面两部分代码
      float jumpSpeed = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);

      // if (velocity.y > 0f) {
      //    jumpSpeed = Mathf.Max(jumpSpeed - velocity.y, 0f);
      // }
      // 新增变量alignedSpeed存储计算出来的法线方向的运动分量
      float alignedSpeed = Vector3.Dot(velocity, contactNormal);
      // 新增if判断，如果法线方向运动分量不为0，对本次的跳跃速度值进行调整
      if (alignedSpeed > 0f)
         jumpSpeed = Mathf.Max(jumpSpeed - alignedSpeed, 0f);

      //velocity.y += jumpSpeed;
      velocity += contactNormal * jumpSpeed;
   }
}

// 沿着斜坡移动
// 球沿着斜坡下落，加速度如果够大，球将离开斜面
// 为了避免球离开斜面掉落，要计算斜面的速度，而不是使用在地面上的速度

// 斜面速度
// Vector3.ProjectOnPlane会额外归一化
// 这个不额外归一化 因为法线就是单位长度的
Vector3 ProjectOnContactPlane(Vector3 vector) {
   return vector - contactNormal * Vector3.Dot(vector, contactNormal);
}

//调整运动速度 根据在接触平面向右和向前的向量投影来确定投影X轴和Z轴
void AdjustVelocity() {
   Vector3 xAxis = ProjectOnContactPlane(Vector3.right).normalized;
   Vector3 zAxis = ProjectOnContactPlane(Vector3.forward).normalized;
   flaot currentX = Vector3.Dot(velocity, xAxis);
   flaot currentZ = Vector3.Dot(velocity, zAxis);
   float acceleration = onGround ? maxAcceleration : maxAirAcceleration;
   float maxSpeedChange = acceleraton * Time.deltaTime;
   float newX = Mathf.MoveTowards(currentX, desiredVelocity.x, maxSpeedChange);
   float newZ = Mathf.MoveTowards(currentZ, desiredVelocity.z, maxSpeedChange);
   velocity += xAxis * (newX - currentX) + zAsix * (newZ - currentZ);
}

void FixedUpdate() {
   UpdateState();
   //float acceleration = onGround ? maxAcceleration : maxAirAcceleration;
   //float maxSpeedChange = acceleration * Time.deltaTime;
   //velocity.x = Mathf.MoveTowards(velocity.x, desiredVelocity.x, maxSpeedChange);
   //velocity.z = Mathf.MoveTowards(velocity.z, desiredVelocity.z, maxSpeedChange);
   //调用AdjustVelocity方法来代替上面删掉的四行代码
   AdjuseVelocity();

   if (desiredJump) {
      desiredJump = false;
      Jump();
   }
   body.velocity = velocity;
   onGround = false;
}


// 多地面法线
// 需要将法线算成平均值
void FixedUpdate() {
   body.velocity = velocity;
   ClearState();
}

//计数地面接触
void ClearState() {
   //onGround = false;
   groundContactCount = 0;
   contactNormal = Vector3.zero;
}

// 累积法线 取代之前重写法线的代码
void EvaluateCollision(Collision collision) {
   for (int i = 0; i < collision.contactCount; i++) {
      Vector3 normal = collision.GetContact(i).normal;
      if (normal.y >= minGroundDotProduct) {
         // onGround = true;
         groundContactCount += 1;
         contactNormal += normal;
      }
   }
}

void UpdateState() {
   velocity = body.velocity;
   if (onGround) {
      jumpPhase = 0;
      //增加if判断决定是否将接触法线归一化
      if (groundContactCount > 1) {
         contactNormal.Normalize();
      }
   }
   else {
      contactNormal = Vector3.up;
   }
}
```


```c#
// 表面接触
// 防止物体离开地面飞起来，判断离开地面后向下施加力

// 最后一次接触地面后物理更新
int stepSinceLastGrounded;

void UpdateState() {
   stepSinceLastGrounded += 1;
   velocity = body.velocity;
   // if (onGround) {
   if (onGround || SnapToGround()) {
      //如果球落地，将字段置为0
      stepSinceLastGrounded = 0;
      jumpPhase = 0;
      if (groundContactCount > 1) {
         contactNormal.Normallize();
      }
   }
   else {
      contactNormal = Vector3.up;
   }
}

// 不用防止整形溢出吗？
// 不用，这个值不会很大

// 贴合
// 使球保持贴合在地面上
bool SnapToGround() {
   if (stepSinceLastGrounded > 1) {
      return false;
   }
   // 发射射线
   // 使用射线判断在球体位置正下方击中了什么东西
   // if (!Physics.Raycast(body.position, Vector3.down)) {
   if (!Physics.Raycast(body.position, Vector3.down, out RaycastHit hit)) {
      return false;
   }
   // 如果击中表面法线小于设定的临界点乘数，表示表面的倾斜角度过大，不被视作地面
   if (hit.normal.y < minGroundDotProduct) {
      return false;
   }
   // return false;
   groundContactCount = 1;
   contactNormal = hit.normal;
   // 现在我们希望空中的球体应该贴合在地面上，下一步调整速度，让球体与地面对齐
   // 需要保持球体当前的速度，不使用ProjectOnContactPlane方法，直接计算球体在地面的速度分量

   // 取到当前速度的大小
   float speed = velocity.magnitude;
   // 当前速度与射线击中的表面的法线进行点积，也就是得到它们长度的积再乘以它们夹角余弦
   float dot = Vector3.Dot(velocity, hit.normal);
   // 此时我们的球体依然没有贴合地面，不过重力会将球体向下拉向地面
   // 所以，应该只在 点积 不为负的时候才调整速度
   if (dot > 0f) {
      // 计算当前运动速度在下方地面平面上的投影分量
      velocity = (velocity - hit.normal * dot).normalized * speed;
   }
   return true;
}


// 最大贴合速度
// 高于某个速度，就可以弹跳
if (speed > maxSnapSpeed) {
   reutrn false;
}

// 探测距离
// 只在一定范围内检测地面
if (!Physics.Raycast(body.position, Vector3.down, out RaycastHit hit, probeDistance)) {
   return false;
}

// 忽略代理
// 忽略射线对其他球体的检测，使用Layer
[SerizalizeField]
LayerMask probeMask = -1;

if (!Physics.Raycast(body.position, Vector3.down, out RaycastHit hit, probeDistance, probeMask)) {
   return false;
}


// 楼梯
// 将楼梯的阶梯形状用作渲染，物理只使用斜面

// 细节Layer和楼梯Layer
// 球体运动使用斜面，但是物体下落使用阶梯
// 通过不同的层级，使用不同的地面

// 最大楼梯角度
[SerizalizeField, Range(0, 90)]
maxStairsAngel = 50f;
float minStairsDotProduct;

void OnValidate() {
   minGroundDotProduct = Mathf.Cos(maxGroundAngle * Mathf.Deg2Rad);
   minStairsDotProduct = Mathf.Cos(maxStairsAngle * Mathf.Deg2Rad);
}

// 返回给定layer的最小点积
float GetMinDot(int layer) {
   return (stairsMask & (1 << layer)) == 0 ? minGroundDotProduct : minStarisDotProduct;
}


// 陡峭接触面 Steep Contacts
// 除了地面与楼梯的接触外，还存在一些其他的接触面。
// 地面或者楼梯接触面用于球体的运动，但是有时我们只接触到了墙壁，或者球体可以被卡在墙壁的裂缝里
// 如果我们设置了空中加速度，那么不会被卡死，不过我们还可以做额外的工作来实现更好的效果

// 检测陡峭接触面
// 如果判断法线分量Y后，发现表示地面或者楼梯，那么继续判断是否时陡峭表面
// 完全垂直的墙壁与水平面点积是0，不过我们放宽限制，判断-0.01
if (normal.y >= minDot) {
   groundContactCount += 1;
   contactNormal += normal;
}
else if (normal.y > -0.01f) {
   steepContactCount += 1;
   steepNormal += normal;
}


// 缝隙
// 缝隙的存在会造成问题，因为一旦球体在空中没有二次跳跃，卡在缝隙里，就出不来了
// 如果球体的maxAirAcceleration很小，甚至为0，那么当球体卡在两个陡峭平面间
// 球体无法跳跃，就无法移动。
// 如果接触到一个以上陡峭表面，我们可以通过向接触点反方向推动球体来进行移动

// 如果存在多个陡峭接触面，那么需要将steepNormal归一化再比较
// 如果接触法线Y分量大于等于minGroundDotProduct，则返回true
// 否则返回false，这种情况下需要判断minStairsDotProduct
bool CheckSteepContacts() {
   if (steepContactCount > 1) {
      steepNormal.Normalize();
      if (steepNormal.y >= minGroundDotProduct) {
         groundContactCount = 1;
         contactNormal = steepNormal;
         return true;
      }
   }
   return false;
}

if (OnGround || SnapToGround() || CheckSteepContacts()) {
   // Can Move
}


// 墙壁跳跃
// 跳跃方向使用steepNormal确定
void Jump() {
   // 新增变量
   Vector3 jumpDirection;

   if (OnGround) {
      jumpDirection = contactNormal;
   }
   else if (OnSteep) {
      jumpDirection = steepNormal;
   }
   else if (jumpPhase < maxAirJumps) {
      jumpDirection = contactNormal;
   }
   else {
      return;
   }

   // 删除第一个判断
   // if (OnGound || jumpPhase < maxAirJumps) {
   jumpPhase += 1;
   float jumpSpeed = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);

   //float alignedSpeed = Vector3.Dot(velocity, contactNormal);
   //使用jumpDirection来代替代码中的contactNormal
   float alignedSpeed = Vector3.Dot(velocity, jumpDirection);
   if (alignedSpeed > 0f) {
      jumpSpeed = Math.Max(jumpSpeed - alignedSpeed, 0f);
   }

   //velocity += contactNormal * jumpSpeed;
   //使用jumpDirection来代替代码中的contactNormal
   velocity += jumpDirection * jumpSpeed;
   //}
}


// 空中跳跃
// 代码的逻辑有点问题：按下空格后，先执行jump方法，将jumpPhase加1
// 下一次物理更新时，先执行UpdateState方法，此时由于groundContactCount在上一帧的OnCollsionStay方法中被处理为1
// 也就是此时依然认为球体与地面有接触，则会在UpdateState方法再次将jumpPhase设置为0
// 因此我们应该输入跳跃指令后至少经过一次物理更新后，再在UpdateState中重置jumpPhase，从而避免错误的落地判断

stepSinceLastGrounded = 0;
// 经过一次以上的物理更新才重置jumpPhase
if (stepSincelastJump > 1) {
   jumpPhase = 0;
}

// 为了保持空中二段跳依旧有效，我们要修改之前在Jump方法中检查jumpPhase的判断语句
// else if (jumpPhase < maxAirJumps) {
else if (jumpPhase <= maxAirJumps) {
   jumpDirection = contactNormal;
}

// 这种做法会导致球体在没有跳跃的情况下直接下落到表面后，能在空中额外跳跃一次。
// 为了防止这个问题，我们需要在这时设置jumpPhase至少为1
else if (jumpPhase <= maxAirJumps) {
   if (jumpPhase == 0) {
      jumpPhase = 1;
   }
   jumpDirection = contactNormal;
}

// 严格来说，只有maxAirJumps大于0时，这段逻辑才有意义 继续上面的修改
// else if (jumpPhase <= maxAirJumps) {
else if (maxAirJumps > 0 && jumpPhase <= maxAirJumps) {
   if (jumpPhase == 0) {
      jumpPhase = 1;
   }
   jumpDirection = contactNormal;
}

// 最后，在发生从墙壁表面的跳跃时，将jumpPhase重置为0，这样就可以通过墙壁跳跃来进行一系列空中二段跳
else if (OnSteep) {
   jumpDirection = steepNormal;
   jumpPhase = 0;
}


// 偏上跳跃
// 现在从垂直的墙壁起跳后，不会增加垂直向上的速度，所以球体会落下来
// 我们想让球体能踩着墙壁起跳

jumpDirection = (jumpDirection + Vector3.up).normalized;

float alignedSpeed = Vector3.Dot(velocity, jumpDirection);
```

## 环游摄像机

```c#
// Cinemachine 内置的自由摄像机功能

// 保持相对位置
using UnityEngine;
[RequireComponent(typeof(Camera))]
public class OrbitCamera : MonoBehaviour {
   // 用来指定摄像机要跟踪的Transform
   [SerializeField]
   Transform focus = default;
   // 设置摄像机与跟踪目标之间的距离
   [SerializeField, Range(1f, 20f)]
   float distance = 5f;

   void LateUpdate() {
      // 摄像机要跟随的位置点
      Vector3 focusPoint = focus.position;
      // 摄像机的观察方向，就是自身的z轴正方向，也就是transform.forward
      Vector3 lookDirection = transform.forward;
      // 摄像机的位置设置为从focus位置开始，向负的lookDirection方向，前进distance距离
      // 这里使用了localPosition，因为摄像机此时没有父物体
      transform.localPosition = focusPoint - lookDirection * distance;
   }
}

// 物理 刚体 插值
// 此时，球体运动不平滑，相机也不平滑
// 将球体的刚体组件插值打开 Interpolate:Interpolate

// 灵敏度
[SerializeField, Min(0f)]
float responsiveness = 5f;

// 从LateUpdate中拿出来
Vector3 focusPoint;
void Awake() {
   focusPoint = focus.position;
}
void LateUpdate() {
   UpdateFocusPoint();
   Vector3 lookDirection = transform.forward;
   transform.localPosition = focusPoint - lookDirection * distance;
}

void UpdateFocusPoint() {
   Vector3 p = foucs.position;
   if (responsiveness > 0f) {
      // 如果灵敏度大于0， 插值
      // focusPoint = Vector3.Lerp(focusPoint, p, responsiveness * Time.deltaTime);
      // 为了避免Time.deltaTime受到时间缩放系数影响
      // focusPoint = Vector3.Lerp(focusPoint, p, responsiveness * Time.unscaledDeltaTime);

      // 需要保障球体发生较大位置变化时不会在视野消失
      // 发生极小位置变化时可以忽略不计
      // 我们可以使用当前跟踪位置及球体位置间距离的平方作为系数来进行插值，从而实现这种效果
      float distanceSqr = (focusPoint - p).sqrMagnitude;
      //使用当前球体位置p于与上次的跟随位置focusPoint之间的距离的平方作为系数,
      //这样可以让距离小于1时得到更小跟随点坐标, 距离大于1时得到更大的跟随点坐标
      focusPoint = Vector3.Lerp(focusPoint, p, distanceSqr * responsiveness * Time.unscaledDeltaTime);
   }
   else {
      focusPoint = p;
   }
}

// 受到帧率影响
// 更高的帧率将会比更低的帧率受到更大的抑制效果
// 如果帧率不太稳定，可以使用下面方法
focusPoint = Vector3.Lerp(focusPoint, p, distanceSqr * Mathf.Pow(responsiveness, Time.unscaledDeltaTime));


// 环绕球体
// 调整摄像机朝向，在环绕球体的轨道上运动来观察球体
// 我们可以手动控制这个轨道，也可以让摄像机自动随着它的跟随点自动旋转

// 朝向角度
// 摄像机的朝向可以使用两个角度进行描述，x角度定义了摄像机的垂直朝向
// 0度表示水平，90度表示垂直向下
// y角度定义了摄像机的水平朝向
// 0度表示看向世界空间的z轴正方向

// 保存摄像机的朝向角度
Vector2 orbitAngles = new Vector2(45f, 0f);

// 在LateUpdate中，我们要通过Quaternion.Euler方法来构造一个代表摄像机朝向的四元数
// 之后可以使用得到的四元数与Vector3.forward向量相乘，代替transform.forward
// 计算出摄像机的新朝向，然后通过transform.SetPositionAndRotation方法一次性的设置好摄像机的位置和朝向
void LateUpdate() {
   UpdateFocusPoint();

   // 根据oribitAngles得到代表摄像机朝向变化的四元数
   Quaternion lookRotation = Quaternion.Euler(orbitAngles);
   //Vector3 lookDiraction = transform.forward;
   // 根据朝向变化计算摄像机的新朝向
   Vector3 lookDirection = lookRotation * Vector3.forward;
   //transform.localPosition = focusPoint - lookDirection * distance;
   // 新增lookPosition变量存储计算出的摄像机新位置
   Vector3 lookPosition = focusPoint - lookDirection * distance;
   // 一次性设置摄像机的位置和朝向角度
   transform.SetPositionAndRotation(lookPosition, lookRotation);
}

// 控制轨道
// 为了可以手动控制摄像机环绕球体，首先需要添加一个rotationSpeed
// 表示角速度，单位：角度/秒
[SerializeField, Range(1f, 360f)]
float rotationSpeed = 90f;

// 自定义输入Vertical Camera和Horizontal Camera
// 新增方法ManualRotation，用来获取输入指令
// 根据用户输入设置orbitAngles
void ManualRotation() {
   //使用input向量存储输入数据
   Vector2 input = new Vector2(Input.GetAxis("Vertical Camera"), Input.GetAxis("Horizontal Camera"));
   //常量e
   const float e = 0.001f;

   if (input.x < -e || input.x > e || input.y < -e || input.y > e) {
      //如果在任意方向上的输入值大于e，则计算新的orbitAngles的值
      orbitAngles += rotationSpeed * Time.unscaledDeltaTime * input;
   }
}

//在LateUpdate方法中调用
void LateUpdate() {
   UpdateFocusPoint();
   ManualRotation();
}

// 注意，球体控制运动方向是相对世界空间的，与摄像机此时的朝向无关
// 所以如果将摄像机水平旋转180度，球体的运动控制表现会变得和输入完全相反
// 这样很难下达正确指令


// 约束角度
// 摄像机在水平方向上环绕球体不会出现明显问题
// 但是在垂直方向上环绕超过90度，画面就上下颠倒了

// 增加两个字段限制相机在垂直方向上环绕运动的角度范围
[SerizalieField, Range(-89f, 89f)]
float minVerticalAngle = -30f, maxVerticalAngle = 60f;

// 最大旋转角度不应该低于最小旋转角度
void OnValidate() {
   if (maxVerticalAngle < minVerticalAngle) {
      // 如果最大角度小于最小角度，设置相等
      maxVerticalAngle = minVerticalAngle;
   }
}

// 新增ConstraninAngles方法，将orbitAngle限制在两个角度之间
void ConstrainAngles() {
   orbitAngles.x = Mathf.Clamp(orbitAngles.x, minVerticalAngle, maxVerticalAngle);
   if (orbitAngles.y < 0f) {
      // 如果y轴旋转角度是负数，加上360
      orbitAngle.y += 360f;
   }
   else if (orbitAngles.y > 360f) {
      //如果y轴旋转角度大于360度，减去
      orbitAngles.y -= 360f;
   }
}

// 为什么不使用循环语句来不断调整y轴旋转角度，使其最终处于360度内？
// 如果轨道角度是任意变化的，那么确实需要多次循环处理
// 然而，我们只会通过输入控制，连续的，进行较小的角度变化，
// 所以在这里不需要循环处理，因为摄像机不会出现一次更新就改变了360度以上的情况

// 我们只需要在角度被改变时才对其进行约束
// void ManualRotation() {
bool ManualRotation() {
   //使用input向量存储输入数据
   Vector2 input = new Vector2(Input.GetAxis("Vertical Camera"), Input.GetAxis("Horizontal Camera"));
   //常量e
   const float e = 0.001f;

   if (input.x < -e || input.x > e || input.y < -e || input.y > e) {
      //如果在任意方向上的输入值大于e，则计算新的orbitAngles的值
      orbitAngles += rotationSpeed * Time.unscaledDeltaTime * input;

      return true;
   }

   return false;
}

void LateUpdate() {
   UpdateFocusPoint();
   //ManualRotation();
   //Quaternion lookRotation = Quaternion.Euler(orbitAngles);
   // 优化
   Quaternion lookRotation;
   if (ManualRotaton()) {
      // 如果返回true，说明角度改变了
      ConstrainAngles();
      lookRotation = Quaternion.Euler(oribitAngles);
   }
   else {
      // 没用变化，保持和现在一致
      lookRotation = transform.localRotation;
   }
}

// 还需要在Awake中保障初始值
void Awake() {
   focusPoint = focus.position;
   // 保障摄像机在最开始时的旋转与orbitAngles相等
   transform.localRotation = Quaternion.Euler(orbitAngles);
}


// 自动对准

// 设置用户在水平旋转相机多少秒后才可以进行相机自动对准
[SerializeField, Min(0f)]
float alignDelay = 5f;

// 存储最后一次改变相机X轴角度的时间点
float lastManualRotationTime;
// 在ManualRotation返回true之前
lastManualRotationTime = Time.unscaledTime;

// 判断是否可以进行自动对准
bool AutomaticRotation() {
   if (Time.unscaledTime - lastManualRotationTime < alignDelay) {
      // 时间还没超过延迟时间，不能自动对准
      return false;
   }
   return true;
}

// 在LateUpdate中，判断发生了手动旋转或者自动旋转时，约束旋转角度并且重新计算相机的四元数
if (ManualRotation() || AutomaticRotation()) {
   ConstrainAngles();
   lookRotation = Quaternion.Euler(orbitAngles);
}


// 强制对准球体前进方向
// 让相机看向跟随点最后一次前进方向

// 新增字段，存储上一个相机跟随位置点
Vector3 previousFocusPoint;

void UpdateFocusPoint() {
   // 设置上一个跟随位置点
   previousFocusPoint = focusPoint;

   //Vector3 p = focus.positon;
   //不再需要p
   focusPoint = focus.position;

   if (responsiveness > 0f) {
      //float distanceSqr = (focusPoint - p).sqrMagnitude;
      //不再使用p来计算distanceSqr
      float distanceSqr = (foucusPoint - previousFocusPoint).sqrMagnitude;
      //focusPoint = Vector3.Lerp(focusPoint, p, distanceSqr * Mathf.Pow(responsiveness, Time.unscaledDeltaTime);
      //不再使用变量p
      focusPoint = Vector3.Lerp(previousFocusPoint, focusPoint, distanceSqr * responsiveness * Time.unscaledDeltaTime);
   }
}

//然后需要计算当前帧，球体移动的方向
bool AutomaticRotation() {
   if (Time.unscaledTime - lastmanualRotationTime < alignDelay) {
      return false;
   }
   //计算从上一个跟随点到当前跟随位置点的运动向量movement
   Vector2 movement = new Vector2(focusPoint.x - previousFocusPoint.x, focusPoint.z - previsousFocusPonit.z);
   // 计算球体运动向量大小的平方
   float movememtDeltaSqr = movement.sqrMagnitude;
   if (movementDeltaSqr < 0.000001f) {
      return false;
   }

   // 计算球体的前进方向角度
   float headingAngle = GetAngle(movement / Mathf.Sqrt(movementDeltaSqr));
   // 使用计算出的球体前进方向角度为水平方向上的摄像机轨道旋转角度赋值
   orbitAngles.y = headingAngle;

   return true;
}

// 如果球体确实在移动，那么我们需要计算出当前运动方向的水平旋转角度
static float GetAngle(Vector2 direction) {
   // Acos将参数视为余弦值，返回该余弦对应的弧度
   float angle = Mathf.Acos(direction.y) * Mathf.Rad2Deg;
   // return angle;
   // 该角度可能是顺时针也可能是逆时针
   // 如果向量的x分量小于0，说明结果是逆时针角度
   return direction.x < 0f ? 360 - angle : angle;
}


// 平滑过渡对准过程
float headingAngle = GetAngle(movement / Mathf.Sqrt(movementDeltaSqr));
//使用与手动操作旋转一样的角速度rotationSpeed得到每次自动旋转需要更新的水平角度变化
float rotationChange = rotationSpeed * Time.unscaledDeltaTime;
//orbitAngle.y = headingAngle;
//使用MoveTowardsAngle方法与计算出的rotationChange来平滑的进行摄像机自动旋转
orbitAngles.y = AMthf.MoveTowardsAngle(orbitAngles.y, headingAngle, rotationChange);

// 现在对准过程效果更好了，但是却始终使用最大旋转速度进行对准
// 更加自然的方式应该让旋转速度因需要旋转的角度大小而进行缩放
// 我们将让缩放系数在需要旋转的角度大于某个角度时达到最大值
[SerializeField, Range(0f, 90f)]
//当摄像机当前朝向与球体前进方向夹角大于等于
float alignSmoothRange = 45f;

// 然后我们需要在AutomaticRotation中得到当前角度和目标角度之间的差
// 可以用Mathf.DeltaAngle来得到这个角度
// 然后我们取得这个角度的绝对值，如果绝对值小于alignSmoothRange
// 那么就按照该角度相对alignSmoothRnage的比例来缩放旋转角度

// 计算当前摄像机水平角度与球体前进方向角度间的最小角度差的绝对值
// Mathf.DeltaAngle方法将得到两个角度值参数直接的最小相差的角度, 比如10度与20度最小相差10度, 370度与20度同样相差10度
float deltaAbs = (Mathf.DeltaAngle(orbitAngles.y, headingAngle));

float rotationChange = rotationSpeed * Time.unscaledDeltaTime;
if (deltaAbs < alignSmoothRange) {
   //如果deltaAbs小于alignSmoothRange，则最终旋转速度要乘以deltaAbs / alignSmoothRange 也就是降低了实际的旋转速度
   rotationChange *= deltaAbs / alignSmoothRange;
}
orbitAngles.y = Mathf.MoveTowardsAngle(orbitAngles.y, headingAngle, rotationChange);

// 我们可以用类似方法处理球体朝着摄像机所在位置接近时摄像机旋转速度的变化
// 防止相机进行大角度跟踪时，突然发生快速旋转
if (deltaAbs < alignSmoothRange) {
      rotationChange *= deltaAbs / alignSmoothRange;
} else if (180f - deltaAbs < alignSmoothRange) {
   rotationChange *= (180f - deltaAbs) / alignSmoothRange;
}

// 最后，我们可以通过时间增量和移动增量中的最小值 与 rotationSpeed相乘来进一步抑制微小移动时的镜头旋转
float rotationChange = rotationSpee * Mathf.Min(Time.unscaledDeltaTime, movementDeltaSqr);



// 相对摄像机移动

// 输入坐标空间
// 不一定非要是世界坐标空间
[SerializeField]
Transform playerInputSpace = default;

// 如果希望使用摄像机自身的Transform作为输入的坐标空间
// 就需要将这个自定义的输入坐标空间转换为世界坐标空间
// 可以在MovingSphere脚本中的Update方法中，使用Transform.TransformDirection方法
if (playerInputSpace) {
   //如果playerInputSpace被分配了一个代表自定义坐标空间的transform，那么运动方向就是该transform
   //的本地空间下的方向，使用这个方向向量来计算最终的球体运动速度
   desiredVelocity = playerInputSpace.TransformDirection(playerInput.x, 0f, playerInput.y) * maxSpeed;
}
else {
   desiredVelocity = new Vector3(playerInput.x, 0f, playerInput.y) * maxSpeed;
}

// 方向归一化
if (playerInputSpace) {
   //得到色号相机本地空间，前方向，在世界坐标空间下的向量
   Vector3 forward = playerInputSpace.forward;
   //舍弃前方向的y分量
   forward.y = 0f;
   //将前方向归一化
   forward.Normalize();
   //得到摄像机本地空间 右方向 在世界坐标空间下的向量
   Vector3 right = playerInputSpace.right;
   //舍弃右方向的y分量
   right.y = 0f;
   right.Normalize();
   //球体的目标运动速度方向就等于摄像机的前方向和右方向与输入数据的乘积之和，再乘以maxSpeed得到最终速度
   desiredVelocity = (forward * playerInput.y + right * playerInput.x) * maxSpeed;
}

// 摄像机碰撞
// 穿透阻挡它的物体，遮挡视线


// 减小观察距离
// 在摄像机和它的跟随位置点之间出现其他东西时，将摄像机沿着它的观察方向向前推
Vector3 lookDirection = lookRotation * Vector3.forward;
//定义变量存储计算出的摄像机最终距离
float lookDistance;
if (Physics.Raycast(focusPoint, -lookDirection, out RaycastHit hit, distance)) {
   lookDistance = hit.distance;
}
else {
   lookDistance = distance;
}
//摄像机的位置使用lookDistance代替之前的distance来计算
Vector3 lookPosition = focusPoint - lookDirection * lookDistance;


// 防止近裁面穿透物体
// 发射一条射线不足以完全解决问题，即使在摄像机的位置和跟踪位置点之间存在通畅的连线
// 摄像机的近裁面矩形仍然可以部分穿透场景内的物体
// 解决这个问题的办法是在世界空间内投射与摄像机近裁面矩形相匹配的box

//1 使用该字段存储摄像机引用
Camera regularCamera;
void Awake() {
   regularCamera = GetComponent<Camera>();

   focusPoint = focus.position;
   transform.localRotation = Quaternion.Euler(orbitAngles);
}
// 2 投射
// 盒状投射需要一个包含一半盒子尺寸的向量，也就是盒子一半的宽度，高度，深度
Vector3 CameraHalfExtends {
   get {
      //盒子尺寸的一半
      Vector3 halfExtends;
      halfExtands.y = regularCamera.nearClipPlane * Mathf.Tan(0.5f * Mathf.Deg2Rad * regularCamera.fieldOfView);
      //计算高度
      halfExtends.x = halfExtends.y * regularCamera.aspect;
      //深度为0
      halfExtends.z = 0f;
      return healfExtends;
   }
}

// 用box
if (Physics.BoxCast(focusPoint, CameraHalfExtends, -lookDirection, out RaycastHit hit, lookRotation, distance)) {
   lookDistance = hit.distance;
}

// 我们应该使用摄像机面到近裁面的距离作为box射线的长度，只需使用distance减去摄像机的近裁面距离设置就可以得到
if (Physics.BoxCast(focusPoint, CameraHalfExtends, -lookDirection, out RaycastHit hit, lookRotation, distance - regularCamera.nearClipPlane)) {
   //如果box射线在跟随位置点与摄像机近裁面之间检测到物体，则设置摄像机近裁面处于射线击中的距离处
   lookDistance = hit.distance + regularCamera.nearClipPlane;
}

// mask
[SerializeField]
LayerMask obstructionMask = -1;
```