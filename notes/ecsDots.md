### [<主页](https://www.wangdekui.com/)

#### 包安装
尽量用最新的 如果报错换版本  
Entities  
Mathematics  
Hybrid Renderer  
Jobs  
Burst  

#### Entity Debugger
场景不显示  

#### Tiny
Unity官方H5技术方案，使用了DOTS  

## 介绍
Data-Oriented Tech Stack 多线程 面向数据  
由  
Entity Component System 实体组件系统  
Job System 任务系统，多线程  
Burst Compiler Burst编译器，优化多线程  
组成  

为了改善<大量指针，全局变量，到处回收>造成的缓存不命中问题  
使用ECS紧密排列数据  

由于摩尔定律失效，单核性能不再增长  
使用Job System，Burst Compiler利用多核  

#### 独立性

不使用DOTS  
使用ECS  
使用ECS，使用Job System  
使用DOTS  

#### 转变

```c#
public class Game : MonoBehaviour {
    public int x;
    private void Update() {
        x++;
    }
}
```

```c#
public struct GameComponentData : IComponentData {
    public int x;
}
public class MyGameSystem : ComponentSystem {
    protected override void OnUpdate() {
        this.Entities.ForEach((ref GameComponentData data)=>{
            data.x++;
        });
    }
}
```

#### 高性能C#
Job System必须用高性能的数据结构  
High Performance C#

```c#
// 扩展数组需要新建
NativeArray<Vector3> nativeArray = new NativeArray<Vector3>(5, Allocator.Temp);
for (int i = 0; i < nativeArray.Length; i++)
    nativeArray[i] = Vector3.one * i;
nativeArray[2] = Vector3.one;
```

```c#
// 没有Remove，只有RemoveAtSwapBack，会打乱顺序
NativeList<Vector3> nativeList = new NativeList<Vector3>(5, Allocator.Temp);
nativeList.Add(Vector3.one);
nativeList.Add(Vector3.one);
nativeist.RemoveAtSwapBack(0);
```

```c#
// 其实是链表
NativeQueue<Vector3> queue = new NativeQueue<Vector3>(Allocator.Temp);
queue.Enqueue(Vector3.one);
queue.Enqueue(Vector3.one);
Vector3 a = queue.Dequeue();
```

```c#
// 没有iterator，不能foreach
NativeHashMap<int, Vector3> hashMap = new NativeHashMap<int, Vector3>(5, Allocator.Temp);
hashMap[0] = Vector3.one;
hashMap[1] = Vector3.one;
NativeArray<int> keys = hashMap.GetKeyArray(Allocator.Temp);
for (int i = 0; i < keys.Length; i++) {
    int key = keys[i];
    Vector3 value = hashMap[key];
}
```

其他  
NativeSlice  
NativeMultiHashMap  
NativeStream  

删除  
nativeArray.Dispose();  

```c#
void Update() {
    Profiler.BeginSample("NativeArray");
    NativeArray<Vector3> velocity = new NativeArray<Vector3>(1000, Allocator.Persistent);
    for (int i = 0; i < velocity.Length; i++)
        velocity[i] = Vector3.zero;
    Profiler.EndSample();
    velocity.Dispose();

    Profiler.BeginSample("Vector3[]");
    Vector3[] velocity1 = new Vector3[1000];
    for (int i = 0; i < velocity1.Length; i++)
        velocity[i] = Vector3.zero;
    Profiler.EndSample();
}
```
Allocator.Persistent 持久化  
Allocator.Temp 临时  
Allocator.TempJob  
Allocator.Invalid  
Allocator.None  
Allocator.AudioKernel  

#### Job System

```c#
// 看性能
[BurstCompile]
struct Jon : IJob {
    public NativeArray<float> values;
    public float offet;
    public void Execute() {
        for (int i = 0; i < values.Length; i++)
            values[i] = i / offset;
    }
}

void Update() {
    Profiler.BeginSample("Normal");
    float [] normal = new float[10000];
    for (int i = 0; i < normal.Length; i++)
        normal[i] /= 5f;
    Profiler.EndSample();

    Profiler.BeginSample("Job");
    NativeArray<float> values = new NativeArray<float>(10000, Allocator.TempJob);
    var job = new Job() {
        values = values,
        offset = 5f,
    };
    JobHandle jobHandle = job.Schedule();
    jobHandle.Complete();
    values.Dispose();
    Profiler.EndSample();
}
```

```c#
// 多线程并行
[BurstCompile]
struct Job : IJobParallelFor {
    [WriteOnly] public NativeArray<float> values;
    [ReadOnly] public float offset;
    public void Execute(int index) {
        values[index] /= offset;
    }
}
void Update()
{
    for (int i = 0; i < 1000; i++) {
        NativeArray<float> values = new NativeArray<float>(500, Allocator.TempJob);
        var job = new Job() {
            values = values,
            offset = 5f,
        };
        JobHandle jobHandle = job.Schedule(values.Length, 32);
        jobHandle.Complete();
        values.Dispose();
    }
}
```

并行写入 并行读取  
```c#
// AsParallelWriter
// AsParallelReader
public struct Job : IJobparallelFor {
    [ReadOnly] public NativeArray<int> array;
    [WriteOnly] public NativeQueue<int>.ParallelWriter queue;
    public void Excute(int index) {
        queue.Enqueue(array[index]);
    }
}
private void Start() {
    NativeArray<int> array = new NativeArray<int>(new int[]{0, 1, 2, 3}, Allocator.TempJob);
    NativeQueue<int> queue = new NativeQueue<int>(Allocator.TempJob);
    Job1 job = new Job1 {
        array = array,
        queue = queue.AdParalleWriter(),
    };
    job.Run(array.Length);
    array.Dispose();
    queue.Dispose();
}
```

#### Burst Compiler

导航菜单->Job->Burst->Open Inspector  

强制使用同步编译，牺牲浮点数精度提高效率  
```c#
[BurstCompile(CompileSynchronously = true)]
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast)]
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast, FloatPrecision = FloatPrecision.High)]
```

应用程序在手机安装时才进行AOT编译  
Project Settings  
Burst AOT Settings  

代码块中不能包含堆内存对象，如果关闭Burst编译，Job中就可以new 对象

```c#
class A {
    public int a;
}

struct Job : IJobParallelFor {
    [WriteOnly] public NativeArray<float> values;
    [ReadOnly] public float offset;
    public void Execute(int index) {
        value[index] = index / offset;
        A a = new A(){ a = 200 };
        Debug.Log(a.a);
    }
}
```

BurstDiscard  
```c#
// 使用Burst
[BurstCompile]
struct Job : IJobParallelFor {
    [WriteOnly] public NativeArray<float> values;
    [ReadOnly] public float offset;
    public void Execute(int index) {
        value[index] = index / offset;
        MethodToDiscard();
    }
}

[BurstDiscard]
static void MethodToDiscard() {
    A a = new A() { a = 200};
    Debug.Log(a.a);
}
```

#### Using Static

```c#
// 语法糖
// 只能用静态方法
// struct enum可以使用static
using static System.Console;
private static void func() {
    WriteLine("blackheart");
}
using static Mystruct;
using static System.ConsoleColor;
public struct Mystruct {
    public static void MystructStaticFunc() {

    }
}
private static void func2() {
    MystructStaticFunc();
    System.Console.ForegroundColor = Black;
}
```

#### Component

```c#
public struct PositionAndRotation : IComponentData {
    public float3 position;
    public quaternion rotation;
}

// 共享的Data
public struct RendererMesh : ISharedComponentData, IEquatable {
    public Mesh mesh;
    public Material material;
    public bool Equals(RendererMesh other) {
        return mesh == other.mesh && material == other.material;
    }
    public override int GetHashCode() {
        int hash = 0;
        if (!ReferenceEquals(mesh, null))
            hash ^= mesh.GetHashCode();
        if (!ReferenceEquals(material, null))
            hash ^= material.GetHashCode();
        return hash;
    }
}

void Start() {
    var entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
    entityManager.CreateEntity(typeof(PositionAndRotation), typeof(RendererMesh));
}
```

SystemStateComponentData 系统状态组件  
IBufferElementData 动态缓冲区  
块组件  entityManager.AddChunkComponentData(entity);

#### System

```c#
// 主线程运行
public struct PositionAndRotation : IComponentData {
    public float3 position;
    public quaterion rotation;
}
public class MySystem : ComponentSystem {
    protected override void OnUpdate() {
        Entities.ForEach((ref PositionAndRotation posAndRot) => {
            posAndRot.position += Time.time;
        });
    }
}
```

```c#
//多线程
public class MyJobSys : JobComponentSystem {
    [BurstCompile]
    struct MyJob : IJobForEach<PositionAndRotation> {//只关心这个组件
        public float time;
        public void Execute([WriteOnly] ref PositionAndRotation posAndRot) {
            posAndRot.position = time + posAndRot.position;
        }
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var job = new MyJob() { time = Time.time };
        return job.Schedule(this, inputDeps);
    }
}

//交给多线程
JobHandle handle = job.Schedule(this, inputDeps);
handle.Complete();
return handle;
```


## [<主页](https://www.wangdekui.com/)