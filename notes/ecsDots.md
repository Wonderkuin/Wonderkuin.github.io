### [<主页](https://www.wangdekui.com/)

[指南第二部分](#index_second)
[文档与博客](#index_documents)

### 宏
不用托管的IComponentData  
UNITY_DISABLE_MANAGED_COMPONENTS  
Hybrid Renderer V2 URP或者DHRP开启  
ENABLE_HYBRID_RENDERER_V2  

#### 包安装

```c#
// DOTS设置

// 推荐安装
// com.unity.entities
// com.unity.rendering.hybrid
// com.unity.dots.editor

// Domain重载
// 避免进入Play模式时，缓慢的 Domain Reload
// 在 Edit ProjectSettings Editor 下面选中 进入播放模式选项
// 但是不要选中 重新加载域 重新加载场景 框
// 但是要记住禁用域加载时，

// 禁用前
public class StaticCounterExample : MonoBehaviour {
    // this counter will not reset to zero when Domain Reloading is disabled
    static int counter = 0; 
    // Update is called once per frame
    void Update()    {
            if (Input.GetButtonDown("Jump"))            {
                    counter++;
                    Debug.Log("Counter: " + counter);
            }
    }
}

// 禁用后
public class StaticCounterExampleFixed : MonoBehaviour{
    static int counter = 0;
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]
    static void Init()    {
            Debug.Log("Counter reset.");
            counter = 0;   
    }
    // Update is called once per frame
    void Update()    {
            if (Input.GetButtonDown("Jump"))            {
                counter++;
                Debug.Log("Counter: " + counter);
            }
    }
}

// 禁用前
public class StaticEventExample : MonoBehaviour{
    void Start()    {
            Debug.Log("Registering quit function");
            Application.quitting += Quit;
    }
    static void Quit()    {
        Debug.Log("Quitting!");
    }
}

// 禁用后
public class StaticEventExampleFixed : MonoBehaviour{
    [RuntimeInitializeOnLoadMethod]
    static void RunOnStart()    {
            Debug.Log("Unregistering quit function");
            Application.quitting -= Quit;
    }
    void Start()    {
            Debug.Log("Registering quit function");
            Application.quitting += Quit;
    }
    static void Quit()    {
        Debug.Log("Quitting the Player");
    }
}

// 独立版本
//     !!!
//     警告：不要使用 File 的 Build And Run 构建DOTS项目
//     必须使用  Build Configuration Asset Inspector window 里面的 构建 构建并运行
//     实体子场景不包括在通过 构建并运行 菜单进行的构建中，并且无法加载
// com.unity.platforms.android
// com.unity.platforms.ios
// com.unity.platforms.linux
// com.unity.platforms.macos
// com.unity.platforms.web
// com.unity.platforms.windows

//  Asset Create Build 将子场景添加到 独立项目中，确保要添加场景
// 或者启用 生成当前场景
```

```
官方包

DOTS ECS                                 com.unity.entities
Rendering                                com.unity.rendering.hybrid
- Hybrid Renderer V2                     com.unity.render-pipelines.high-definition or com.unity.render-pipelines.universal
- Animation                              com.unity.animation
Audio                                    com.unity.audio.dspgraph
Physics                                  com.unity.physics or com.havok.physics
- Smooth Penetration Recovery            com.havok.physics
- Stable Object Stacking                 com.havok.physics
- Remove Speculative Contacts            com.havok.physics
- Rigidbody Sleeping                     com.havok.physics
- Visual Debugger                        com.havok.physics
Multiplayer                              com.unity.netcode
- Lag Compensation                       com.unity.physics
Project Building                         com.unity.platforms
- Android                                com.unity.platforms.android
- Linux	                                 com.unity.platforms.linux
- macOS	                                 com.unity.platforms.macos
- Web                                    com.unity.platforms.web
- Windows                                com.unity.platforms.windows
```

com.unity.entities  
Entities  
Jobs  
Burst  

Collections

com.unity.mathematics  
Mathematics  

com.unity.rendering.hybrid  
Hybrid Renderer  

com.unity.physics  
Unity Physics  

直接搜索  
Havok Physics for Unity  
ShaderGraph  

# 指南

## 第一部分
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
struct Job : IJob {
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
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast, 
    FloatPrecision = FloatPrecision.High)]
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

//等待执行完毕
JobHandle handle = job.Schedule(this, inputDeps);
handle.Complete();
return handle;

//或者
return job.Run(this);

//同时
NativeList<JobHandle> jobs = new NativeList<JobNative>(Allocator.Temp);
jobs.Add(job.Schedule());
jobs.Add(job.Schedule());
jobHandle.CompleteAll(jobs);
jobs.Dispose();
```

#### Entity Command Buffers

```c#
public struct PositionAndRotation : IComponentData {
    public float3 position;
    public bool delete;
}
public class MySystem : JobComponentSystem {
    EndSimulationEntityCommandBufferSystem m_EndSimulationEcbSystem;
    protected override void OnCreate() {
        base.OnCreate();
        m_EndSimulationEsbSystem = World.GetOrCreateSystem<EndSimulationEntityCommandBufferSystem>();
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var ecb = m_EndSimulationEcbSystem.CreateCommandBuffer().ToConcurrent();
        var jobHandle = Entities.ForEach((Entity entity, int entityInQueryIndex, 
            ref PositionAndRotation posAndRot) => {
                if (!posAndRot.delete) {
                    posAndRot.delete = true;
                    ecb.DestroyEntity(entityInQueryIndex, entity);
                }
        }).Schedule(inputDeps);
        // 添加到队列中，等待统一删除
        m_EndSimulationEcbSystem.AddJobHandleForProducer(jobHandle);
        return default;
    }
}
```

#### Job生命周期
```c#
// 动态创建 运行JobComponentSystem
[DisableAutoCreation]
public class MySystem : JobComponentSystem {}

World world = World.DefaultGameObjectInjectionWorld;
MySystem system = world.GetOrCreateSystem<MySystem>();
var simulationSystemGroup = world.GetOrCreateSystem<SimulationSystemGroup>();
simulationSystemGroup.AddSystemToUpdateList(system);
ScriptBehaviourUpdateOrder.UpdatePlayerLoop(world);

// 关闭
system.Enabled = false;

// 销毁
simulationSystemGroup.RemoveSystemFromUpdateList(system);
world.DestroySystem(system);
ScriptBehaviourUpdateOrder.UpdatePlayerLoop(world);
```

```c#
[DisableAutoCreation]
public class MySystem : ComponentSystem {
    protected override void OnCreate() {}
    protected override void OnDestroy() {}
    protected override void OnStartRunning() {}
    protected override void OnStopRunning() {}
    protected voerride void OnUpdate() {}
}
```

```c#
// 执行顺序
[UpdateBefore(typeof(MySystem))]
public class MySystem2 : ComponentSystem {}

[UpdateAfter(typeof(MySystem))]
public class MySystem3 : ComponentSystem {}
```

#### DOTS 数据交互

```c#
public struct A : IComponentData {
    public float aa;
}
public struct B : IComponentData {
    public float bb;
}
public class MySystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var jobHandle = Entities.ForEach((ref A a, in B b) => {
            a.aa += b.bb;
        }).Schedule(inputDeps);
        return jobHandle;
    }
}
```

```c#
// 普通赋值
protected override JobHandle OnUpdate(JobHandle inputDeps) {
    NativeArray<float> numbers = new NativeArray<float>(500, Allocator.TempJob);
    for (int i = 0; i < numbers.Length; i++)
        numbers[i] = i;
    return inputDeps;
}
// 在多线程中赋值
protected override JobHandle OnUpdate(JobHandle inputDeps) {
    NativeArray<float> numbers = new NativeArray<float>(500, Allocator.TempJob);
    JobHandle generateNumbers = Job.WithCode(()=>{
        for (int i = 0; i < numbers.Length; i++)
            numbers[i] = i;
    }).Schedule(inputDeps);
    generateNumbers.Complete();
    numbers.Dispose();
    return inputDeps;
}
```

```c#
return Entities.WithAll<LocalToWorld>()
    .WithAny<Rotation, Translation, Scale>()
    .WithNone<LocalToParent>()
    .ForEach((ref Destination outputData, in Source inputData)=>{
        ;
    })
    .Schedule(inputDeps);
```

查询WithSharedComponentFilter  
在Entities.ForEach中使用  
WithStoreEntityInField  

```c#
var jobHandle = Entities
    .ForEach((ref A a, in B b)=>{
        a.aa += b.bb;
    })
    .WithBurst(FloatMode.Default)
    .Schedule(inputDeps);
    //WithoutBurst()禁用Burst
```

#### IJob

```c#
[BurstCompile]
struct MyJob : IJobForEach<A, B> {
    public void Execute([WriteOnly] ref A aa, [ReadOnly] ref B bb) {
        aa.aa += bb.bb;
    }
}
```

```c#
//不能包含组件
[ExcludeComponent(typeof(Frozen))]
//必须包含组件
[RequireComponentTag(typeof(Gravity))]
[BurstCompile]
struct RotationSpeedJob : IJobForEach<RotationQuaternion, RotationSpeed>{}
```

#### IJobChunk

```c#
public struct A : IComponentData {
    public float aa;
}
public struct B : IComponentData {
    public float bb;
}
public struct C : IComponentData {
    public float cc;
}
public class MySystem : JobComponentSystem {
    private EntityQuery m_Query;
    protected override void OnCreate() {
        base.OnCreate();
        var queryDescription = new EntityQueryDesc() {
            //不能包含的组件
            None = new ComponentType[]{
                typeof(A);
            },
            //必须包含任意一个组件
            Any = new ComponentType[] {
                typeof(B),
                typeof(c)
            },
            //必须包含的组件
            All = new ComponentType[] {
                ComponentType.ReadWrite<B>();
                ComponentType.ReadOnly<C>();
            }
        };
        m_Query = GetEntityQuery(queryDescription);
    }

    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var job = new UpdateJob() {
            LastSystemVersion = this.LastSystemVersion,
            bb = GetArchetypeChunkComponentType<B>(false),
            cc = GetArchetypeChunkComponentType<C>(true)
        };
        return job.Schedule(m_Query, inputDeps);
    }
}

[BurstCompile]
struct UpdateJob : IJobChunk {
    public ArchetypeChunkComponentType<B> bb;
    [ReadOnly] public ArchetypeChunkComponentType<C> cc;
    public uint LastSystemVersion;

    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityindex) {
        var inputAChanged = chunk.DidChange(bb, LastSystemVersion);
        if (!(inputAChanged))
            return;
            var inputAs = chunk.GetNativeArray(bb);
            var readOnly = chunk.GetNativeArray(cc);

            for (var i = 0; i < inputAs.Length; i++) {
                inputAs[i] = new B { bb = inputAs[i].bb + readOnly[i].cc };
            }
    }
}

void Start()
{
    var entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
    var entity = entityManager.CreateEntity(typeof(A), typeof(B), typeof(C));
    entityManager.SetComponentData<C>(entity, new C() { cc = 100 });
}
```

#### 组

```c#
public struct A : IComponentData {
    public Color color;
}

var entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
var entity entityManager.CreateEntity(typeof(A));

public class ASystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var jobHandle = Entities
            .ForEach((ref A a) => {
                a.color = Color.red;
            })
            .WithEntityQueryOptions(EntityQueryOptions.FilterWriteGroup)
            .WithoutBurst()
            .Schedule(inputDeps);
        return jobHandle;
    }
}

//相当于添加到None，需要开启EntityQueryOptons.FilterWriteGroup
[WriteGrou(typeof(A))]
public struct B : IComponentData {
    public Color color;
}

public class BSystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var jobHandle = Entities
            .ForEach((ref B b)=>{
                b.color = Color.black;
            })
            .WithoutBurst()
            .Schedule(inputDeps);
        return jobHandle;
    }
}
```

#### Version

```c#
entity.Version
System.LastSystemVersion
Chunk.ChangeVersion[]
EntityManager.m_ComponentTypeOrderVersion[]
SharedComponentDataManager.m_SharedComponentVersion[]
```

#### World

```c#
EntityManager
ComponentSystems
Archetypes
```

#### Convert

```c#
using UnityEngine;
using Unity.Entities;

[System.Serializable]
public struct TestEntity : IComponentData {
    public int value;
}
public class Test : MonoBehaviour, IConvertGameObjectToEntity {
    public int value;
    public void Convert(Entity entity, EntityManager dstManager, 
        GameObjectConversionSystem conversionSystem) {
            dstManager.AddComponent(entity, typeof(TestEntity));
            TestEntity testEntity = new TestEntity { value = value };
            dstManager.AddComponentData(entity, testEntity);
    }
}

// 不需要每次new
[GenerateAuthoringComponent]
public struct TestEntity : IComponentData {
    public int value;
    public Entity entity;
}
```

```c#
public class TestJobSystem : JobComponentSystem {
    public struct TestJob : IJobForEach<TestEntity> {
        public int a;
        public void Execute(ref TestEntity test) {
            test.value = a;
        }
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var job = new TestJob();
        job.a = 100;
        return job.Schedule(this, inputDeps);
    }
}

// 简略
public class TestJobsystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        return this.Entities.ForEach((ref TestEntity test)=>{
            test.value = 100;
        }).Schedule(inputDeps);
    }
}
```

#### 多个系统同时修改一个实体组件

```c#
// 不推荐
public class TestJobSystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        JobHandle newHandle = this.Entities.ForEach((ref TestEntity test)=>{
            test.value = 100;
            })
            .WithoutBurst()
            .Schedule(inputDeps);
        newHandle.Complete();
        return newHandle;
    }
}

public class TestJobSystem2 : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        JobHandle newHandle = this.Entities.ForEach((ref TestEntity test)=>{
            test.value += 100;
            })
            .WithoutBurst()
            .Schedule(inputDeps);
        newHandle.Complete();
        return newHandle;
    }
}
```

```c#
//强制同步所有异步任务后，再执行自己的Job
[AlwaysSynchronizesystem]
public class TestJobSystem2 : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        JobHandle newHandle = this.Entities.ForEach((ref TestEntity test) => {
            test.value += 100;
            })
            .Schedule(intputDeps);
        return default;
    }
}
```

#### Job 的返回结果

```c#
public class TestJobSystem : JobComponentSystem {
    public struct TestJob : IJobForEach<TestEntity> {
        public int a;
        public int b;
        public int c;
        public void Execute(ref TestEntity test) {
            c = a + b;
        }
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var job = new TestJob();
        job.a = 100;
        job.b = 200;
        job.Run(this);

        Debug.Log(job.c);//预期是300，结果是0

        return inputDeps;
    }
}

//改
public class TestJobSystem : JobComponentSystem {
    public struct TestJob : IJobForEach<TestEntity> {
        public int a; 
        public int b;
        public NativeArray<int> c;
        public void Execute(ref TestEntity test) {
            c[0] = a + b;
        }
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        NativeArray<int> temp = new NativeArray<int>(1, Allocator.TempJob);
        var job = new TestJob();
        job.a = 100;
        job.b = 200;
        job.c = temp;
        job.Run(this);

        Debug.Log(temp[0]);//300

        temp.Dispose();
        return inputDeps;
    }
}
```

<div id="index_second"></div>

## 第二部分

#### Convert To Entity
转换为Entity对象  
Convert And Destroy  
Convert And Inject GameObject  

```c#
using System;
using Unity.Entities;
[Serializable]
public struct Speed : IComponentData {
    public int i;
    public float f;
    public double d;
}
public class SpeedProxy : ComponentDataProxy<Speed> {}

// 新写法
[GenerateAuthoringComponent]
public struct GameComponentData : IComponentData {
    public int x;
}
```

#### 共享组件

```c#
// 数组 字符串 贴图 材质等引用类型，无法序列化
using System;
using Unity.Entities;
using UnityEngine;
[Serializable]
public struct Speed2 : ISharedComponentData, IEquatable<Speed2> {
    public Texture2D texture;
    public bool Equals(Speed2 other) {
        return texture == other.texture;
    }
    public override int GetHashCode() {
        int hash = 0;
        if (!ReferenceEquals(texture, null))
            hash ^= texture.GetHashCode();
        return hash;
    }
}
public class Speed2Proxy : SharedComponentDataProxy<Speed2>{}
```

#### 实体组件

RenderMeshProxy  
网格  
Mesh Material SubMesh Layer Cast Shadows Receive Shadows  

Translation 位置  
Rotation 旋转  
NonUniformScaleProxy 缩放  

LocalToWorld  一个顶三个  
GameObject转换的Entity自动绑定  

```c#
Unity.Mathematics.quaternion.EulerZXY(1f * Mathf.Deg2Rad, 2f * Mathf.Deg2Rad, 3f * Mathf.Deg2Rad);
Quaternion.Euler(1f, 2f, 3f);

//Q = cos( a / 2 ) + i( x * sin(a / 2) ) + j( y * sin(a / 2) ) + k( z * sin(a / 2) )
public static quaternion EulerZXY(float3 xyz) {
    float3 s,c;
    sincos(0.5f * xyz, out s, out c);
    return quaternion(
        // s.x * c.y * c.z + s.y * s.z * c.x,
        // s.y * c.x * c.z - s.x * s.z * c.y,
        // s.z * c.x * c.y - s.x * s.y * c.z,
        // c.x * c.y * c.z + s.y * s.z * s.x
        float4(s.xyz, c.x) * c.yxxy * c.zzyz + 
            s.yxxy * s.zzyz * float4(c.xyz, s.x) * float4(1f, -1f, -1f, 1f);
    );
}
```

#### Prefab转Entity

```c#
public class Main : MonoBehaviour {
    public GameObject prefab;
    void Start() {
        var world = World.DefaultGameObjectInjectionWorld;
        var manager = world.EntityManager;
        var settings = GameObjectConversionSettings.FromWorld(world, null);
        var entityPrefab = GameObjectConversionUtility.ConvertGameObjectHierarchy(prefab, settings);
        for (int i = 0; i < 100; i++) {
            var instance = manager.Instance(entityPrefab);
#if UNITY_EDITOR
            manager.SetName(instance, prefab.name + instance.Index);
#endif
            var position = transform.TransformPoint(Random.onUnitySphere * 3f);
            manager.SetComponentData(instance, new Translate{ Value = position });
        }
    }
}
```

#### Entity对象父子关系

Prefab包含父子关系  
自动转换  
LinkedEntityGroup  
Child  
```c#
// Prefab转为Entity
var entityPrefab = GameObjectConversionUtility.ConvertGameObjectHierarchy(prefab, settings);
var instance = manager.Instantiate(entityPrefab);
manager.SetName(instance, "Prefab (Clone)");

var manager = World.DefaultGameObjectInjectionWorld.EntityManager;
//修改坐标
manager.SetComponentData(instance, new Translation {Value = Vector3.zero });
//删除实体
manager.DestroyEntity(instance);
```

只支持Mesh  不支持粒子特效 骨骼动画烘焙贴图  

#### Entity对象的脚本

```c#
[System.Serializable]
public struct TestEntity : IComponentData {
    public int value;
}

public class Test : MonoBehaviour, IConvertGameObjectToEntity {
    public int value;

    public void Convert(Entity entity, EntityManager dstManager, 
        GameObjectConversionSystem conversionSystem) {
            //将数据添加给TestEntity
            dstManager.AddComponent(entity, typeof(TestEntity));
            TestEntity testEntity = new TestEntity { value = value };
            dstManager.AddComponentData(entity, testEntity);
    }
}
```

场景转换Entity  
右键根节点New SubScene From Selection  
新root挂载Sub Scene  
不支持粒子特效 动画 Light Map烘焙贴图  

```c#
//断开LinkedEntityGroup 以及删掉子实体的Parent组件
Entities.WithAll().ForEach(entity=>{
    var manager = World.DefaultGameObjectInjectionWorld.EntityManager;
    BufferFromEntity buffers = GetBufferFromEntity();
    var buffers = buffers[entity];
    for (int i = 0; i < buffers.Length; i++) {
        var buffer = buffers[i].Value;
        if (manager.HasComponent(buffer)) {
            if(manager.GetComponnetData(buffer).Value == entity) {
                //删除Parent组件
                PostUpdateCommands.RemoveComponent(buffer);
                buffers.RemoveAt(i);
                i--;
            }
        }
    }
});
```

#### Entity支持烘焙贴图

目前RenderMesh不支持LightMap  
需要自己渲染  

```c++
Shader "Unlit/Customl"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _LightmapST("_LightmapST", Vector) = (0, 0, 0, 0)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                float3 uv1 : TEXCOORD1;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float2 uv1 : TEXCOORD1;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;
            float4 _LightmapST;
            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                //取出每一个物体的LightmapScaleOffset重新计算UV2
                o.uv1 = v.uv1.xy * _LightmapST.xy + _LightmapST.zw;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 co1 = tex2D(_MainTex, i.uv);
                col.rgb = DecodeLightmap(UNITY_SAMPLE_TEX2D(unity_Lightmap, i.uv1.xy));
                return col;
            }
            ENDCG
        }
    }
}
```

传入MeshRenderer Material LightMap  

```c#
public class Main :MonoBehaviour {
    public MeshRenderer meshRenderer;
    public Material m_Material;
    public Texture2D m_Lightmap;
    private MeshFilter m_MeshFilter;
    void Start() {
        meshRenderer.gameObject.SetActive(false);
        m_MeshFilter = meshRenderer.GetComponent<MeshFilter>();
        //将lightmap贴图和偏移传入
        m_Material.SetVector("_LightmapST", meshRenderer.lightmapScaleOffset);
        m_Material.SetTexture("unity_Lightmap", m_Lightmap);
    }
    void Update() {
        //开始渲染
        Graphics.DrawMesh(m_MeshFilter.mesh, 
            meshRenderer.transform.localToWorldMatrix, m_Material, 0);
    }
}
```

合并DrawCall  
借助GPU Instancing  

```c++
Shader "Unlit/Custom1"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _LightmapST("_LightmapST", Vector) = (0, 0, 0, 0)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #pragma multi_compile_instancing

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                float3 uv1 : TEXCOORD1;

                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float2 uv1 : TEXCOORD1;

                UNITY_VERTEX_INPUT_INSTANCE_ID

                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            UNITY_INSTANCING_BUFFER_START(Props)
            UNITY_DEFINE_INSTANCED_PROP(fixed4, _lightmapST)
            UNITY_INSTANCING_BUFFER_END(Props)

            v2f vert (appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                UNITY_TRANSFER_INSTANCE_ID(v, o);
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                //取出每一个物体的LightmapScaleOffset重新计算UV2
                fixed4 l = UNITY_ACCESS_INSTANCED_PROP(Props, _LightmapST);
                o.uv1 = v.uv1.xy * l.xy + l.zw;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 co1 = tex2D(_MainTex, i.uv);
                col.rgb = DecodeLightmap(UNITY_SAMPLE_TEX2D(unity_Lightmap, i.uv1.xy));
                return col;
            }
            ENDCG
        }
    }
}
```

```c#
public class Main : MonoBehaviour {
    public Transform root;
    public Material m_Material;
    public Texture2D m_Lightmap;
    private MeshFilter m_MeshFilter;
    private List<Matrix4x4> m_Materix = new List<Matrix4x4>();
    private List<Vector4> m_LightmapOffset = new List<Vector4>();
    private MaterialPropertyBlock m_Block;
    void Start() {
        // 获取节点下所有的MeshRenderer
        foreach ( var item in root.GetComponentsInChildren<MeshRenderer>(true)) {
            if (m_MeshFilter == null) {
                m_MeshFilter = item.GetComponent<MeshFilter>();
            }
            //保存每个物体的矩阵和lightmapScaleOffset
            m_Materix.Add(item.localToWorldMaterix);
            m_LightmapOffset.Add(item.lightmapScaleOffset);
        }
        //隐藏所有节点
        root.gameObject.SetActive(false);
        //启动LIGHTMAP_ON宏
        m_Material.EnableKeyword("LIGHTMAP_ON");
        //为了避免GC所有只new一次MaterialPropertyBlock
        m_Block = new MaterialPropertyBlock();
        //设置具体用哪张烘焙贴图
        m_Block.SetTexture("unity_Lightmap", m_Lightmap);
        //将每个物体的lightmapScaleOffset传入shader中
        m_Block.SetVectorArray("_LightmapST", m_LightmapOffset.ToArray());
    }
    void Update() {
        //开始渲染
        Graphics.DrawMeshInstanced(m_MashFilter.mesh, 0, m_Material, m_Materix, m_Block);
    }
}
```

Frame Debug 查看渲染详情  

#### Entity渲染原理

GPU Instancing和Entity没有直接关系，只要提供正确渲染数据就可以  
Graphics.DrawMeshInstanced  
渲染数据包括 材质 网格 VBO VAO 世界坐标矩阵  

将准备数据的过程，放入Job中  

Graphics.DrawMeshInstanced是底层API，在主线程调用，在多线程完成  
1，不支持镜头裁切，裁切需要自己做  
2，不支持渲染超过1024个元素，如果超过，自己控制多次调用  
3，需要在Update中每帧调用，传统渲染管线可以用CommandBuffer.DrawMeshInstance，新版URP不支持  
4，建议使用CommandBuffer渲染GPU Instancing，这样只有物体位置或者属性改变，才强制刷新  

```c#
//使用CommandBuffer渲染，减少开销
using UnityEngine;
using UnityEngitn.Rendering;
public class G1 : MonoBehaviour {
    //GPU Instancing材质
    public Material material;
    //GPU Instancing网格
    public Mesh mesh;
    //随便找个位置做随机
    public Transform target;
    //是否使用commandBuffer渲染
    public bool useCommandBuffer = false;
    //观察摄像机
    public Camera m_Camera;

    private Materix4x4[] m_materix4x4s = new Materix4x4[1023];

    void Start() {
        CommandBufferForDrawMeshInstanced();
    }
    void OnGUI() {
        if (GUILayout.Button("位置变化再更新)) {
            CommandBufferForDrawMeshInstanced();
        }
    }
    void Update() {
        if (!useCommandBuffer) {
            GraphicsForDrawMeshInstanced();
        }
    }
    void SetPos() {
        for (int i = 0; i < m_matrix4x4s.Length; i++) {
            target.position = Random.onUnitSphere * 10f;
            m_matrix4x4[i] = target.localToWorldMatrix;
        }
    }
    void GraphicsForDrawMeshInstanced() {
        if (!useCommandBuffer) {
            SetPos();
            Graphics.DrawMeshInstanced(mesh, 0, material, m_matrix4x4s, m_matrix4x4s.Length);
        }
    }
    void CommandBufferForDrawMeshInstanced() {
        if (useCommandBuffer) {
            SetPos();
            if (m_buff != null) {
                m_Camera.RemoveCommandBuffer(CameraEvent.AfterForwardOpaque, m_buff);
                CommandBufferPool.Release(m_buff);
            }
            m_buff = CommandBufferPool.Get("DrawMeshInstanced");
            for (int i = 0; i < 1; i++) {
                m_buff.DrawMeshInstanced(mesh, 0, material, 0, m_matrix4x4s, m_matrix4x4s.Length);
            }
            m_Camera.AddCommandBuffer(CameraEvent.AfterForwardOpaque, m_buff);
        }
    }

    CommandBuffer m_buff = null;
}
```

CommandBuffer只有当元素改变位置时才刷新，效率高了很多  
缺点是SRP不支持它，不过可以使用BatchRendererGroup  

#### 渲染原理

RenderMeshSystemV2是新渲染组件  
FrozenRenderSceneTag是场景冻结，这个元素不会被运行时修改  
```c#
//静态组
m_FrozenGroup = GetEntityQuery(
    ComponentType.ChunkComponentReadOnly<ChunkWorldRenderBounds>(),
    ComponentType.ReadOnly<WorldRenderBounds>(),
    ComponentType.ReadOnly<LocalToWorld>(),
    ComponentType.ReadOnly<RenderMesh>(),
    ComponentType.ReadOnly<FrozenRenderSceneTag>()
);
//动态组
m_DynamicGroup = GetEntityQuery(
    ComponentType.ChunkComponentReadOnly<ChunkWorldRenderBounds>(),
    ComponentType.Exclude<FrozenRenderSceneTag>(),
    ComponentType.ReadOnly<WorldRenderBounds>(),
    ComponentType.ReadOnly<LocalToWorld>(),
    ComponentType.ReadOnly<RenderMesh>()
);
```

找到静态和动态对象后，准备裁切  
WorldRenderBounds是包围区域  

```c#
//BatchRendererGroup强制需要裁切
BatchRendererGroup m_BatchRendererGroup;
void Start() {
    m_BatchRendererGroup = new BatchRendererGroup(this.OnPerformCulling);
}
public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, 
    BatchCullingContext cullingContext) {
        //在这里可以开启一个Job进行裁切
        return default;
}
```

```c#
BatchRendererGroup m_BatchRendererGroup;
public Mesh mesh;
public Material material;
void Start() {
    m_BatchRendererGroup = new BatchRendererGrouop(this.OnPerformCulling);
    //绘制10个元素，Bounds就是物体的区域
    int batchIndex = m_BatchRendererGroup.AddBatch(mesh, 0, material, 0, ShadowCastringMode.On, 
        true, false, new Bounds(Vector3.zero, Vector3.one), 10, null, null);
    //获取指定Batch的索引
    var matrixes = m_BatchRendererGroup.GetBatchMatrices(batchIndex);
    //设置坐标
    for (int i = 0; i < matrixes.Length; i++) {
        Matrix4x4 offset = Matrix4x4.identity;
        //标准矩阵的坐标在原点，这里我修改每个元素的X轴+1
        offset.m03 = i;
        matrixes[i] = offset;
    }
}
```

```c#
// 裁切掉第三个元素，不让它渲染
public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, 
    BatchCullingContext cullingContext) {
        //遍历Batch 因为前面只加了一个Batch，i只有1一个值
        for (var i = 0; i < cullingContext.batchVisibility.Length; i++) {
            //只有一个Batch对象，但是它绘制了10个元素
            var batchVisibility = cullingContext.batchVisibility[i];
            var visibleInstancesIndex = 0;
            for (var j = 0; j < batchVisibility.instancesCount; ++j) {
                //当绘制2号元素的时候跳过
                //batchVisibility.offset表示索引的开始
                if (j == 2) continue;
                cullingContext.visibleIndices[visibleInstancesIndex] = batchVisibility.offset + j;
                visibleInstancesIndex++;
            }
            //这样参数显示的数量就变成9
            batchVisibility.visibleCount = visibleInstancesIndex;
            //重新赋值batchVisibility对象
            cullingContext.batchVisibility[i] = batchVisibility;
        }
        return default;
}
```

目的是将超过摄像机显示的区域的物体剔除，有了Bounds组件其实还不够  
还要根据摄像机方向，远近裁面，每个物体Bounds才能求得是否在镜头中  

```c#
BatchRendererGRoup m_BatchRendererGroup;
void Start() {
    m_BatchRendererGroup = new BatchRendererGroup(this.OnPerformCulling);
    //绘制十个元素
    int batchIndex = m_BatchRendererGroup.AddBatch(mesh, 0, material, 0, ShodowCastingMode.On, true, false,
        //因为每个立方体长度是1，所以乘以10
        new Bounds(Vector3.zero, Vector3.one * 10f), 10, null, null);
    //获取指定Batch的索引
    var matrixes = m_BatchRendererGroup.GetBatchMatrices(batchIndex);
    //在多线程中添加数据
    AddJob add = new AddJob() {
        matrixes = matrixes
    };
    //等待多线程结束
    add.Run(matrixes.Length);
}

//负责添加，不需要考虑顺序，可以使用并行任务
[BurstCompile]
public struct AddJob : IJobParallelFor {
    public NativeArray<Matrix4x4> matrixes;
    public void Execute(int index) {
        Matrix4x4 offset = Matrix4x4.identity;
        //标准矩阵的坐标在原点，这里修改每个元素的x轴+1
        offset.m03 = index;
        matrixes[index] = offset;
    }
}

//裁切处理
JobHandle m_Handle;
public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, 
    BatchCullingContext cullingContext) {
        CullingJob culling = new CullingJob() {
            //因为只有一个批次，id是0，多次需要修改
            matrixes = m_BatchRendererGroup.GetBatchMatrices(0);
            planes = Unity.Rendering.FrustumPlanes.BuildSOAPlanePackets(
                    cullingContext.cullingPlanes, Allocator.TempJob),
            batchVisibility = cullingContext.batchVisibility,
            visibleIndices = cullingContext.visibleIndices,
        };
        var handle = culling.Schedule(m_Handle);
        m_Handle = JobHandle.CombineDependencies(handle, m_Handle);
        m_Handle.Complete();
        return handle;
}
```

```c#
//为什么要用IJob，因为只添加了一个渲染批次，渲染这一个批次对顺序有要求
//如果前面添加了多个渲染批次，就可以用IJobParallelFor，可以让每个批次并行计算，但是每个批次内要顺序执行

//FrustumPlanes.Intersect2(planes, aabb)
//这个方法可以判断每个物体的AABB是否在摄像机显示区域，返回三个状态
//Unity.Rendering.FrustumPlanes.IntersectRersult.In
//Unity.Rendering.FrustumPlanes.IntersectRersult.Partial
//Unity.Rendering.FrustumPlanes.IntersectRersult.Out
[BurstCompile]
public struct CullingJob : IJob {
    [ReadOnly] public NativeArray<Matrix4x4> matrixes;
    [DeallocateOnJobCompletion] [ReadOnly] public 
        NativeArray<Unity.Rendering.FrustumPlane.PlanePacket4> planes;
    public NativeArray<BatchVisibility> batchVisibility;
    public NativeArray<int> visibleIndices;
    public void Execute() {
        //遍历这个批次的每个元素
        for (var i = 0; i < batchVisibility.Length; i++) {
            var batchVisib = batchVisibility[i];
            int visibleInstancesIndex = 0;
            //因为立方体的是1米，所以Extents就是0.5，如果画多个不同Mesh，这个值从外面传入
            AABB localBond = new AABB() { Center = Vector3.zero, Extents = Vector3.one * 0.5f };
            for (int j = 0; j < matrixes.Length; j++) {
                //计算每个AABB的世界坐标系的区域
                var aabb = AABB.Transform(matrixes[j], localBond);
                //判断每个元素是否在摄像机中
                if (Unity.Rendering.FrustumPlanes.Intersect2(planes, aabb) != 
                    Unity.Rendering.FrustumPlanes.IntersectResult.Out) {
                        //如果摄像机能看到正确设置的index
                        visibleIndices[visibleInstancesIndex] = batchVisib.offset + j; 
                        //添加参与渲染的元素的数量
                        visibleInstancesIndex++;
                    }
            }
            //最终计算出真正参与渲染的元素数量，以及不显示的元素
            batchVisib.visibleCount = visibleInstancesIndex;
            batchVisibility[i] = batchVisib;
        }
    }
}

private void OnDestroy() {
    if (m_BatchRendererGroup != null) {
        m_BatchRendererGroup.Dispose();
        m_BatchRendererGroup = null;
    }
}
```

这里的裁切要在Frame Debugger中查看，Scene中看不出来  
可以等物体改变再裁切，固定间隔裁切，不需要每帧运行  

#### 更新颜色
需要自己实现shader，更改color属性  
```c#
[MaterialProperty("_Color", MaterialPropertyFormat.Float4)]

public struct MyOwnColor : IComponentData {
    public float4 Value;
}

class AnimateMyOwnColorSystem : SystemBase {
    protected override void OnUpdate() {
        Entities.ForEach((ref MyOwnColor color, in MyAnimationTime t)=>{
            color.Value = new float4(
                math.cos(t.Value + 1f),
                math.cos(t.Value + 2f),
                math.cos(t.Value + 3f),
                1f
            );
        }).Schedule();
    }
}
```

#### RenderMesh原理

就是在多线程中准备数据，再调用绘制API  

```c#
protected override JobHandle OnUpdate(JobHandle inputDeps) {
    inputDeps.Complete(); // todo

    m_InstancedRenderMeshBatchGroup.CompleteJobs();
    m_InstancedRenderMeshBatchGroup.ResetLod();

    Profiler.BeginSample("UpdateFrozenRenderBatches");
    UpdateFrozenRenderBatches();
    Profiler.EndSample();

    Profiler.BeginSample("UpdateDnamicRenderBatches");
    UpdateDynamicRenderBatches();
    Profiler.EndSample();

    m_InstancedRenderMeshBatchGroup.LastUpdatedOrderVersion = 
        EntityManager.GetComponentOrderVersion<RenderMesh>();

    return new JobHandle();
}
```

#### ECS优化头顶血条

GPU Instancing必须满足相同Mesh和相同材质  
选学MaterialPropertyBlock  

创建SpriteAtlas图集，可以拖文件夹进去  
写脚本，传入Shader，Sprite  
BatchRendererGRoup.AddBatch  
ECS负责修改坐标  
1，参与渲染的sprite数量变化，重新AddBatch  
2，参与渲染的sprite坐标变化，Job里计算坐标
3，参与渲染的sprite无变化，重新刷新MaterialPropertyBlock  

```c#
using System.Collections.Generic;
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;
using unityEngine.Rendering;
using static Unity.Mathematics.math;

public class GSprite : MonoBehaviour {
    public struct ECS {
        public Vector3 position;
        public Vector3 scale;
        public Vector2 pivot;
    }
    private BatchRendererGroup m_BatchRendererGroup;
    private Mesh m_Mesh;
    private Material m_Material;
    private int m_BatchIndex = -1;
    private JobHandle m_Handle;
    private NativeList<ECS> m_SpriteData;
    private List<Vector4> m_SpriteDataOffset = new List<Vector4>();

    public Sprite sprite1;
    public Sprite sprite2;
    public Sprite sprite3;
    public Shader shader;

    void Start() {
        m_SpriteData = new NativeList<ECS>(100, Allocator.Persistent);
        m_Mesh = Resources.GetBuiltionResource<Mesh>("Quad.fbx");
        m_Material = new Material(shader) { enableInstancing = true };
        m_Material.mainTexture = sprite1.texture;

        //显示图片的一部分
        AddSprite(sprite1, Vector3.zero, Vector3.one, 0.5f);
        //显示完整图片，缩小
        AddSprite(sprite2, Vector3.zero + new Vector3(1,0,0), Vector3.one * 0.5f, 1f);
        //显示完整图片
        AddSprite(sprite3, Vector3.zero + new Vector3(1,1,0), new Vector3(10,2,1), 1f);

        Refresh();
    }

    void AddSprite(Sprite sprite, Vector2 localPosition, Vector2 localScale, float slider) {
        float perunit = sprite.pixelsPerUnit;
        Vector3 scale = new Vector3((sprite.rect.width / perunit) * localScale.x,
            (sprite.rect.height / perunit) * localScale.y, 1f);
        Vector4 rect = GetSpreiteRect(sprite);
        scale.x *= slider;
        rect.x *= slider;

        ECS obj = new ECS();
        obj.position = localPosition;
        obj.piovt = new Vector2(sprite.piovt.x / perunit * localScale.x,
            sprite.piovt.y / perunit * localScale.y);
        obj.scale = scale;
        m_SpriteData.Add(obj);
        m_SpriteDataOffset.Add(rect);
    }

    private void Refresh() {
        //1，参与渲染的sprite数量发生变化，需要重新addbatch
        RefreshElement();
        //2，参与渲染的sprite坐标发生变化，在job重新计算坐标
        RefreshPosition();
        //3，无变化，重新刷新MaterialPropertyBlock
        //RefreshBlock();
    }

    private void RefreshElement() {
        if (m_BatchRendererGroup == null) {
            m_BatchRendererGroup = new BatchRendererGroup(OnPerformCulling);
        }
        else {
            m_BatchRendererGroup.RemoveBatch(m_BatchIndex);
        }
        MaterialPropertyBlock block = new MaterialPropertyBlock();
        block.SetVectorArray("_Offset", m_SpriteDataOffset);
        m_BatchIndex = m_BatchRendererGroup.AddBatch(
            m_Mesh,
            0,
            m_Material,
            0,
            ShadowCastingMode.Off,
            false,
            false,
            default(Bounds),
            m_SpriteData.Length,
            block,
            null
        );
    }

    void RefreshPosition() {
        m_Handle.Complete();

        m_Handle = new UpdateMatrixJob {
            Matrices = m_BatchRendererGroup.GetBatchMatrices(m_BatchIndex),
            objects = m_SpriteData,
        };
        m_Handle.Schedule(m_SpriteData.Length, 32);
    }

    void RerfreshBlock() {
        MaterialPropertyBlock block = new MaterialPropertyBlock();
        block.SetVectorArray("_Offset", m_SpriteDataOffset);
        m_BatchRendererGroup.SetInstancingData(m_BatchIndex, m_SpriteData.Length, block);
    }

    public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, 
        BatchCullingContext cullingContext) {
            //sprite不需要处理镜头裁切，这里直接完成job
            m_Handle.Complete();
            return m_Handle;
    }

    [BurstCompile]
    private struct UpdateMatrixJob : IJobParallelFor {
        public NativeArray<Matrix4x4> Matrices;
        [ReadOnly] public NativeList<ECS> objects;
        public void Execute(int index) {
            //通过锚点计算sprite实际的位置
            ECS go = objects[index];
            var position = go.position;
            float x = position.x + (go.scale.x * 0.5f) - go.pivot.x;
            float y = position.y + (go.scale.y * 0.5f) - go.pivot.y;

            Matrices[index] = Matrix4x4.TRS(float3(x, y, position.z),
                Unity.Mathematices.quaternion.identity,
                objects[index].scale);
        }
    }

    private Vector4 GetSpriteRect(Sprite sprite) {
        var uvs = sprite.uv;
        Vector4 rect = new Vector4();
        rect[0] = uvs[1].x - uvs[0].x;
        rect[1] = uvs[0].y - uvs[2].y;
        rect[2] = uvs[2].x;
        rect[3] = uvs[2].y;
        return rect;
    }

    private void OnDestroy() {
        if (m_BatchRendererGroup != null) {
            m_BatchRendererGroup.Dispose();
            m_BatchRendererGroup = null;
        }
        m_SpriteData.Dispose();
    }
}
```

```c++
//渲染区域需要动态传入Shader
Shader "ECSSprite"
{
    Properties
    {
        _MainTex("Texture", 2D) = "white" {}
        _Offset("_Offset", Vector) = (1,1,0,0)
    }
    SubShader
    {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" }
        Blend
        SrcAlpha
        OneMinusSrcAlpha
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing
            #include "UnityCG.cginc"
            #pragma target 3.0

            struct appdata
            {
                float4 vertex : POSITION;
                UNITY_VERTEX_INPUT_INSTANCE_ID
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            UNITY_INSTANCING_BUFFER_START(Props)
            UNITY_DEFINE_INSTANCED_PROP(fixed4, _Offset)//渲染区域
            UNITY_INSTANCING_BUFFER_END(Props)

            v2f vert (appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                o.vertex = UnityObjectToClipPos(v.vertex);
                fixed4 l = UNITY_ACCESS_INSTANCED_PROP(Props, _Offset);
                o.uv = v.uv.xy * 1.xy + 1.zw;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                //sample the texture
                fixed col = tex2D(_MainTex, i.uv);
                return col;
            }
            ENDCG
        }
    }
}
```
结果：一个DrawCall渲染了多个图片，还能修改位置  

#### GPU蒙皮

骨骼有Animation关键帧动画  
蒙皮标记每个骨骼对应影响到的那些顶点  
权重控制一个顶点受到多个骨骼影响  

骨骼数量较少， 60到上百  
顶点数以万计  
因此蒙皮计算量更大，考虑在GPU蒙皮  

ProjectSetting GPU Skinning  
需要如 OpenGL ES 3.0 Transform feedback传回功能  
将顶点计算放在shader的顶点着色器中，将每帧骨骼变换的结果传入Shader  
顶点Shader计算完成，将蒙皮结果返回给CPU， 可以缓存起来下次使用  
防止GPU多次计算蒙皮信息  

GPU骨骼变化  
动画文件要转换成贴图，挂点要保存静态数据，查在哪个位置  
占内存，GPU不好就没办法用  

```c#
InstanceData data = block.Value.instanceData;
if (useInstancing) {
#if UNITY_EDITOR
    PreparePackageMaterial(package, vertexCache, k);
#endif
    package.propertyBlock.SetFloatArray("frameIndex", data.frameIndex[k][i]);
    package.propertyBlock.SetFloatArray("preFrameIndex", data.preFrameIndex[k][i]);
    package.propertyBlock.SetFloatArray("transitionProgress", data.transitionProgress[k][i]);
    Graphics.DrawMeshInstanced(vertexCache.mesh,
        j,
        package.material[j];
        data.worldMatrix[k][i],
        package.instancingCount,
        package.propertyBlock,
        vertexCache.shadowcastingMode,
        vertexCache.receiveShadow,
        vertexCache.layer
        );
}
```

```c++
//动画烘焙的图片上记录着每一帧骨骼变换的数据，Shader中要知道当前运行到第几帧，前一帧进度
//需要从CPU传递给shader
UNITY_INSTANCING_CBUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(float, preFrameIndex)
    UNITY_DEFINE_INSTANCED_PROP(float, frameIndex)
    UNITY_DEFINE_INSTANCED_PROP(float, transitionProgress)
UNITY_INSTANCING_CBUFFER_END
```

```c++
//顶点着色器，顶点对应的信息在蒙皮图片中
//每个顶点都计算出当前帧的矩阵信息，动画就播放了
int preFrame = curFrame;
int nextFrame = curFrame + 1.0f;
half4x4 localToWorldMatrixPre = loadMatFromTexture(preFrame, bone.x) * w.x;
localToWorldMatrixPre += loadMatFromTexture(preFrame, bone.y) * max(0, w.y);
localToWorldMatrixPre += loadMatFromTexture(preFrame, bone.z) * max(0, w.z);
localToWorldMatrixPre += loadMatFromTexture(preFrame, bone.w) * max(0, w.w);

half4x4 localToWorldMatrixNext = loadMatFromTexture(nextFrame, bone.x) * w.x;
localToWorldMatrixNext += loadMatFromTexture(preFrame, bone.y) * max(0, w.y);
localToWorldMatrixNext += loadMatFromTexture(preFrame, bone.z) * max(0, w.z);
localToWorldMatrixNext += loadMatFromTexture(preFrame, bone.w) * max(0, w.w);
```

#### Job动画系统

如果用cpu动画，Unity提供了Job骨骼动画  
播放动画时，修改骨骼某个节点的朝向，之前无法做到  

利用AnimationScriptPlayable可以播放动画，不用创建AnimationController  
新版Animator效率提升  

```c#
[RequireComponent(typeof(Animator))]
public class PlayAnimationSample : MonoBehaviour {
    public AnimationClip clip;
    void Start() {
        PlayableGraph playableGraph = PlayableGraph.Create();
        playableGraph.SetTimeUpdateMode(DirectorUpdateMode.GameTime);
        var playableOutput = AnimationPlayableOutput.Create(playableGraph,
            "Animation", GetComponent<Animator>());
        var clipPlayable = AnimationClipPlayable.Create(playableGraph, clip);
        playableOutput.SetSourcePlayable(clipPlayable);
        playableGraph.Play();
    }
}
```

#### 物理引擎

PhysicsBody代替Rigidbody  
PhysicShape代替Collider  
PhysicMaterial取消  

```c#
//Raycast
public Entity Raycast(float3 RayFrom, float3 RayTo) {
    var world = World.DefaultGameObjectInjectionWorld;
    var physicsWorldSystem = world.GetExistingSystem<Unity.Physics.Systems.BuildPhysicsWorld>();
    var collisionWorld = physicsWorldSystem.PhysicsWorld.CollisionWorld;
    RaycastInput input = new RaycastInput() {
        Start = RayFrom,
        End = RayTo,
        Filter = new CollisionFilter() {
            BelongsTo = ^0u,
            CollidesWith = ^0u,
            GroupIndex = 0
        }
    };

    Unity.Physics.RaycastHit hit = new Unity.Physics.RaycastHit();
    bool haveHit = collisionWorld.CastRay(input, out hit);
    if (haveHit) {
        Entity e = physicsWorldSystem.PhysicsWorld.Bodies[hit.RigidBodyIndex].Entity;
        return e;
    }
    return Entity.Null;
}

private void Update() {
    if (Input.GetMouseButton(0)) {
        UnityEngine.Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        //发送射线找到点击的实体
        Entity entity = Raycast(ray.origin, ray.direction * 100f);
        if (entity != Entity.Null) {
            //修改实体渲染的颜色
            var manager = World.DefaultGameObjectInjectionWorld.EntityManager;
            var renderMesh = manager.GetSharedComponentData<RenderMesh>(entity);
            renderMesh.material.color = Color.black;
        }
    }
}
```

```c#
// Trigger
class TriggerSystem : JobComponentSystem {
    [BurstCompile]
    struct TriggerJob : ITriggerEventsJob {
        public void Execute(TriggerEvent triggerEvent) {
            //碰撞时同时出发的两个实体对象
            Debug.Log("A: " + triggerEvent.Entities.EntityA);
            Debug.Log("B: " + triggerEvent.Entities.EntityB);
        }
    }
    BuildPhysicsWorld word;
    StepPhysicsWorld step;
    protected override void OnCreate() {
        base.OnCreate();
        word = World.GetOrCreateSystem<BuildPhysicsWorld>();
        step = World.GetOrCreateSystem<StepPhysicsWorld>();
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        TriggerJob triggerJob = new TriggerJob();
        //开启碰撞检测Job
        return triggerJob.Schedule(step.Simulation, ref word.PhysicsWorld, inputDeps);
    }
}
```

```c#
// 动态添加碰撞
public Entity CreateDynamicSphere(RenderMesh displayMesh, float radius, float3 position, quaternion orientation) {
    BlobAssetReference spCollider = Unity.Physics.SphereCollider.Create(float3.zero, radius);
    return CreateBody(displayMesh, position, orientation, spCollider, float3.zero, float3.zero, 1.0f, true);
}

public unsafe Entity CreateBody(RenderMesh displayMesh, float3 position, quaternion orientation, 
    BlobAssetReference collider, float3 linearVelocity, float3 angularVelocity, float mass, bool isDynamic) {
        EntityManager entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
        ComponentType[] componentTypes = new ComponentType[isDynamic ? 7 : 4];

        componentTypes[0] = typeof(RenderMesh);
        componentTypes[1] = typeof(TranslationProxy);
        componentTypes[2] = typeof(RotationProxy);
        componentTypes[3] = typeof(PhysicsCollider);

        if (isDynamic) {
            componentTypes[4] = typeof(PhysicsVelocity);
            componentTypes[5] = typeof(PhysicsMass);
            componentTypes[6] = typeof(PhysicsDamping);
        }

        Entity entity = entityManager.CreateEntity(componentTypes);
        entityManager.SetSharedComponentData(entity, displayMesh);
        entityManager.AddComponentData(entity, new Translation {Value = position});
        entityManager.AddComponentData(entity, new Rotation {Value = orientation});
        entityManager.SetComponentData(entity, new PhysicsCollider {Value = collider});
        
        if (isDynamic) {
            Collider* colliderPtr = (Collider*)collider.GetUnsafePtr();
            entityManager.SetComponetnData(entity, PhysicsMass.CrateDynamic(colliderPtr->MassProperties, mass));

            float3 angularVelocityLocal = math.mul(math.inverse(colliderPtr->MassProperties.MassDistribution.Transform.rot), angularVelocity);
            entityManager.SetComponentData(entity, new PhysicsVelocity() {
                Linear = linearVelocity,
                Angular = andularVelocityLocal
            });
            entityManager.SetComponentData(entity, new PhysicsDamping() {
                Linear = 0.01f,
                Angular = 0.05f
            });
        }
        return entity;
}
```

#### 加载效率 更新效率

```c#
//加载
float t = Time.realtimeSinceStartup;
for (int i = 0; i < initCount; i++) {
    Vector3 pos = UnityEngine.Random.insideUnitSphere * 5f;
    Instantiate(prefab, pos, Quaternion.identity);
}
t = (Time.realtimeSinceStartup - t) * 1000f;
Debug.LogFormat("Instantiate: {0} ms", t);

float t1 = Time.realtimeSinceStartup;
for (int i = 0; i < initCount; i++) {
    Vector3 pos = UnityEngine.Random.insideUnitSphere * 5f;
    Entity entity = m_Manager.Instantiate(m_EntityPrefab);
    m_Manger.SetComponentData(entity, new Translation {Value = pos});
}
t1 = (Time.realtimeSinceStartup - t1) * 1000f;
Debug.LogFormat("ECS Instantiate: {0} ms", t1);
```

```c#
//更新
private void Update() {
    if(!useEcs) {
        for (int i = 0; i < initCount; i++) {
            var go = m_GameObjects[i];
            float3 pos = go.transform.position;
            pos.y += 0.1f;
            if (pos.y > 5f)
                pos.y = m_InitPos[i].y;
            go.transform.position = pos;
        }
    }
}

[BurstCompile]
private struct MyJob : IJobForEach<Translation, InitData> {
    public void Execute(ref Translation translation, [ReadOnly] ref InitData initData) {
        float3 pos = translation.Value;
        pos.y = 0.1f;
        if (pos.y > 5f)
            pos.y = initData.Value.y;
        translation.Value = pos;
    }
}
```

#### 场景切块加载

ECS提供了将场景批量转换的功能，可以用这个功能将场景分块  
SubScene  
根据角色行走的坐标来动态启动，卸载对应块  
最好保证每个块的顺序是从左到右从上到下的  
默认取消勾选AutoLoadScene，启动游戏就不会加载这个块了  

```c#
using UnityEngine;
using Unity.Entities;
using System.Collections.Generic;
using Unity.Scenes;

public calss MapSlice : MonoBehaviour {
    const int CHUCK_COUT = 4;
    public Transform hero;
    public SubScene[] subScenes;
    private SceneSystem m_SceneSystem;
    private int m_LastX = -1, m_LastZ = -1;
    private Dictionary<int, SubScene> m_LastLoadScene = new Dictionary<int, SubScene>();
    private HashSet<int> m_CurrentLoadSceneIndex = new HashSset<int>();

    private void Start() {
        var world = World.DefaultGameObjectInjectionWorld;
        m_SceneSystem = world.GetOrGreateSystem<SceneSystem>();
    }

    private void Update() {
        int x = (int)(hero.transform.position.x / 10f);
        int z = (int)(hero.transform.position.z / 10f);
        bool isChange = (x != m_LastX) || (z != m_LastZ);
        if (isChange) {
            m_LastX = x;
            m_LastZ = z;
            m_CurrentLoadSceneIndex.Cleaer();
            for (int i = x - 1; i <= x + 1; i++) {
                for (int j = z - 1; j <= z + 1; j++) {
                    if (i >= 0 && j >= 0 && i < CHUNK_COUT && j <CHUNK_COUT) {
                        int index = j * CHUNK_COUT + i;
                        if (index < subScenes.Length) {
                            m_LastLoadScene[index] = subScenes[index];
                            m_CurrentLoadSceneIndex.Add(index);
                            m_SceneSystem.LoadSceneAsync(subScenes[index].SceneGUID);
                        }
                    }
                }
            }
            //卸载不在当前九宫格的块
            List<int> keys = new List<int>(m_LastLoadScene.Keys);
            for (int i = 0; i < keys.Count; i++) {
                var key = keys[i];
                if (!m_CurrentLoadSceneIndex.Contains(key)) {
                    m_SceneSystem.UnloadScene(m_LastLoadScene[key].SceneGUID);
                    m_LastLoadScene.Remove(key);
                }
            }
        }
    }
}
```

#### Dynamic Buffers
```c#
[InternalBufferCapacity(8)]//设置动态缓冲区大小
public struct DynamicData : IBufferElementData {
    public int Value;
}

public struct StaticData : IComponentData {
    public int Value;
}

private void Start() {
    var world = World.DefaultGameObjectInjectionWorld;
    var entityManager = world.EntityManager;
    Entity entity = entityManager.Create(typeof(DynamicData), typeof(StaticData));
    DynamicBuffer<DynamicData> buffers = entityManager.AddBuffer<DynamicData>(entity);
    //根据情况添加0-8之间的动态数据
    for (int i = 0; i < 7; i++)
        buffers.Add(new DynamicData() { Value = i});
    //传统的静态数据，一个组件只能添加一个整形数据，无法形成数组
    entityManager.SetComponentData<StaticData>(entity, new StaticData() {Value = 0});
    //获取数据的地方可能在另外一个类中，只要能拿到entity就可以遍历这段buffer
    DynamicBuffer<DynamicData> GetBuffers = entityManager.GetBuffer<DynamicData>(entity);
    foreach (DynamicData data in GetBuffers)
        Debug.Log(data.Value);
    //可以强制转换成指定的数组    
    foreach (int integer in GetBuffers.Reinterpret<int>())
    Debug.Log(integer);
}
```

#### 寻找目标
```c#
var world = World.DefaultGameObjectInjectionWorld;
var example5MoveSystem = world.GetOrCreateSystem<Example5MoveSystem>();
example5MoveSystem.heroTransform = hero.transform;//设置中心位置

public class Example5MoveSystem : JobComponentSystem {
    //外部负责传入中心坐标
    public Transform heroTransform;
    EndSimulationEntityCommandBufferSystem m_EndSimulationEcbSystem;
    protected override void OnCreate() {
        base.OnCreate();
        m_EndSimulationEcbSystem = world.GetOrCreateSystem<EndSimulationEntityCommandBufferSystem>();
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        float3 heroPos = heroTransform.position;
        float dt = Time.DeltaTime;
        float speed = 5f;
        var ecb = m_EndSimulationEcbSystem.CreateCommandBuffer().ToConcurrent();

        var JobHandle = this.Entities.WithAll<Example5Alias>()
            .ForEach((Entity entity, int entityInQueryIndex, ref Translation translation, ref Rotation rotation)=>{
                //每个符合条件的实体朝向中心移动
                float3 heading = heroPos - translation.Value;
                rotation.Value = quaternion.LookRotation(heading, math.up());
                translation.Value = translation.Value + (dt * speed * math.forward(rotation.Value));
                //当距离小于0.1时加入缓冲区
                if(math.sqrDistance(translation.Value, heroPos) <= 0.01f)
                    ecb.DestroyEntity(entityInQueryIndex, entity);
            }).Schedule(inputDeps);

        //统一执行缓冲区的操作
        m_EndSimulationEcbSystem.AddJobHandleForEach(jobHandle);
        return jobHandle;
    }
}

//利用缓冲区创建实体，添加删除组件
var e = ecb.CreateEntity(entityInQueryIndex);
ecb.AddComponent<Translation>(entityInQueryIndex, e);

//ComponentSystem在主线程执行，更加简洁
//使用PostUpdateCommands 可以创建，删除实体，添加，删除组件
public class Example5MoveSystem : CompnentSystem {
    public Transform heroTransform;
    protected override void OnUpdate() {
        float3 heroPos = heroTransform.position;
        float dt = Time.DeltaTime;
        float speed = 5f;
        this.Entities.WithAll<Example5Atlas>()
            .ForEach((Entity entity, ref Translation translation, ref Rotation rotation)=>{
                //每个符合条件的实体朝向中心移动
                float3 heading = heroPos - translation.Value;
                rotation.Value = quaternion.LookRotation(heading, math.up());
                translation.Value = translation.Value + (dt * speed * math.forward(rotation.Value));
                //当距离小于0.1时加入缓冲区
                if(math.sqrDistance(translation.Value, heroPos) <= 0.01f)
                    PostUpdateCommands.DestroyEntity(entity);
            });
    }
}
```

#### 监听删除的实体和组件
系统状态组件，如果有这个组件，DestroyEntity方法不会删除实体  
ISystemStateComponentData  
PostUpdateCommands.RemoveComponent删掉组件后  
实体对象也会删除  

```c#
public struct Example6Alias : IComponentData{}
public struct Example6AliasSystem : ISystemStateComponentData {}

private void Update() {
    if (Time.time - m_LastInitTime > 1f) {
        m_LastInitTime = Time.time;

        for (int i = 0; i < initCount; i++) {
            Vector3 pos = UnityEngine.Random.insideUnitSphere * 10f;
            Entity entity = m_Manager.Instantiate(m_EntityPrefab);
            m_Manager.SetComponentData(entity, new Translation { Value = pos });
            m_Manager.AddComponentData(entity, new Example6Alias());
            //添加系统状态组件
            m_Manager.AddComponentData(entity, new Example6AliasSystem());
        }
    }
}

//调用DestroyEntity后，实体没有删除，只剩下ISystemStateComponentData组件
public class Example6EventMoveSystem : ComponentSystem {
    protected override void OnUpdate() {
        //找到所有包含Example6Alias组件但是不包含Example6Alias组件的实体
        Entities.WithAll<Example6AliasSystem>()
            .WithNone<Example6Alias>()
            .ForEach((Entity entity) => {
                Debug.LogFormat("entity {0}被删除! ", entity.Index);
                //删除最后的系统状态组件，系统会自动删除实体
                PostUpdateCommands.RemoveComponent<Example6AliasSystem>(entity);
            });
    }
}
```

#### 优化查询遍历的速度

只对坐标发生改变的组件进行更新  
监听变化的组件  
WithChangeFilter  
不够精准  
整个数据块都会被查询到  
```c#
[AlwaysSynchronizeSystem]
public class Example7RemoveSystem : JobComponentSystem {
    public Transform heroTransform;
    EndSimulationEntityCommandBufferSystem m_EndSimulationEcbSystem;
    protected override void OnCreate() {
        base.OnCreate();
        m_EndSimulationEcbSystem = World.GetOrCreateSystem<EndSimulationEntityCommandBufferSystem>();
    }
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        float3 heroPos = heroTransform.position;
        var ecb = m_EndSimulationEcbSystem.CreateCommandBuffer().ToConcurrent();
        var jobHandle = this.Entities
            .WithAll<Example7Alias>()
            .WithChangeFilter<Translation>()//监听变化的组件
            .ForEach((Entity entity, int entityInQueryIndex, Translation transflation)=>{
                //当它们距离小于0.1米时删除
                if (math.sqrDistance(transflation.Value, heroPos) <= 0.1f)
                    ecb.DestroyEntity(entityInQueryIndex, entity);
            })
            .WithoutBurst()
            .Schedule(inputDeps);
        m_EndSimulationEcbSystem.AddJobHandleForProducer(jobHandle);
        return jobHandle;
    }
}
```

建议使用IJobChunk，根据块遍历  
```c#
[BurstCompile]
struct MyJob : IJobChunk {
    //传入坐标组件
    [ReadOnly] public ArchetypeChunkComponentType<Translation> translationType;
    //传入实体类型
    [ReadOnly] public ArchetypeChunkEntityType entityType;
    public float3 heroPos;
    //缓冲区用于删除组件
    public EntityCommandBuffer.Concurrent CommandBuffer;
    //系统版本号
    public unit LastSystemVersion;

    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
        //通过系统版本号和组件类型比较是否发生变化
        var translationsChanged = chunk.DidChange(translationType, LastSystemVersion);
        //如果块中的Translation数组没有变化则不进行遍历
        if (!translationsChanged)
            return;
        //取出坐标组件数组
        var translations = chunk.GetNativeArray(translationType);
        //取出坐标组件对应的实体数组
        var entities = chunk.GetNativeArray(entityType);

        for (var i = 0; i < translations.Length; i++) {
            if (math.sqrDistance(translation[i].Value, heroPos) <= 0.01f)
                CommandBuffer.DestroyEntity(chunkIndex, entities[i]);
        }
    }
}

Entity m_Query = GetEntityQuery(typeof(Example7Alias), typeof(Translation));

protected override JobHandle OnUpdate(JobHandle inputDeps) {
    float3 heroPos = heroTransform.position;
    var myJob = new MyJob() {
        CommandBuffer = m_EndSimulationEcbSystem.CreateCommandBuffer().ToConcurrent(),
        entityType = this.GetArchetypeChunkEntityType(),
        translationType = this.GetArchetypeChunkComponentType<Translation>(true);
        heroPos = heroPos,
        LastSystemVersion = this.LastSystemVersion
    };
    var jobHandle = myJob.Schedule(m_Query, inputDeps);
    m_EndSimulationEcbSystem.AddJobHandleForProducer(jobHandle);
    return jobHandle;
}
```

#### GPU Instancing 和 Entity结合

创建实体并且绑定RenderSprite和RenderSpriteECS  
```c#
public struct RenderSprite : ISharedComponentData, IEquatable<RenderSprite> {
    public Sprite sprite;
    public bool Equals(RenderSprite other) {
        return sprite == other.sprite;
    }
    public override int GetHashCode() {
        int hash = 0;
        if (!ReferenceEquals(sprite, null))
            hash ^= sprite.GetHashCode();
        return hash;
    }
}
public struct RenderSpriteECS : IComponentData {
    public Vector3 position;//显示坐标
    public Vector3 scale;//显示缩放
    public Vector2 piovt;//锚点
    public int index;//索引
}

//Sprite位置变化再刷新，要监听变化
public class Example11MoveSystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var matrices = Example11.m_BatchRendererGroup.GetBatchMatrices(0);
        var jobHandle = this.Entities
            .WithAll<Example11Alias>()
            .ForEach((ref RenderSpriteECS go)=>{
                var position = go.position;
                float x = position.x + (go.scale.x * 0.5f) - go.pivot.x;
                float y = position.y + (go.scale.y * 0.5f) - go.pivot.y;
                //根据RenderSpriteECS组件的坐标信息，修改BatchRendererGroup
                matrices[go.index] = Matrix4x4.TRS(float3(x, y, position.z),
                    Unity.Mathematics.quaternion.identity,
                    go.scale);
            })
            .WithChangeFilter<RenderSpriteECS>()//这里当RenderSprite变化再更新
            .Schedule(inputDeps);
            jobHandle.Complete();
            return inputDeps;
    }
}
```

GPU Instancing最大支持1023个，超过了就要多次调用Graphics.DrawMeshInstanced  
这里用m_BatchRendererGroup.AddBatch代替Graphics.DrawMeshInstanced  
超过1023个会自动分成多次调用，更方便  

#### Animation Instancing

需要把带有Animator的Prefab生成动画贴图  
生成完成，在SkinnedMeshRenderer组件绑定自定义着色器，内含贴图  

将Prefab绑定ConvertToEntity组件  
选Convert And Inject Game Object不删除原始对象  
已经有了Entity，使用ECS方式加载  
```c#
private void Update() {
    float startTime = Time.realtimeSinceStartup;
    if (useEcs) {
        MyJob job = new MyJob();
        job.frameCount = Time.frameCount;
        job.rotations = m_Rotations;
        job.Schedule(initCount, 54).Component();//放到job中完成

        for (int i = 0; i < m_Transforms.Count; i++)
            m_Transform[i].rotation = m_Rotations[i];
    }
    else {
        for (int i = 0; i < m_Transforms.Count; i++) {
            test();//耗时操作
            m_Transforms[i].rotation = quaternion.AxisAngle(math.up(), (Time.frameCount % 360) * Mathf.Deg2Rad);
        }
    }
    Debug.Log((Time.realtimeSinceStartup - startTime) * 1000) + "ms");//统计耗时
}

[BurstCompile]
struct MyJob : IJobParallelFor {
    [WriteOnly] public NativeArray<quaternion> rotations;
    [ReadOnly] public int frameCount;
    public void Execute(int index) {
        test();//耗时操作
        rotations[index] = quaternion.AxisAngle(math.up(), (frameCount % 360) * Mathf.Deg2Rad);
    }
}
```

```c#
class MySystem : JobComponentSystem {
    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        float startTime = Time.realtimeSinceStartup;
        float a = UnityEngine.Time.frameCount;
        this.Entities.WithAll<Example12Alias>();
            .ForEach((Transform rotation)=>{
                rotation.rotation = quaternion.AxisAngle(math.up(), (a % 360) * Mathf.Deg2Rad);
            })
            .WithoutBurst()
            .Run();
        Debug.Log((Time.realtimeSinceStartup - startTime) * 1000) + "ms");
        return inputDeps;
    }
}
```
官方工程：  
https://github.com/Unity-Technologies/Animation-Instancing  
推荐第三方库：  
https://github.com/joeante/Unity.GPUAnimation  

#### 骨骼动画Demo

Packages目录修改manifest.json  
添加  
"com.unity.animation":"0.3.0-preview.7",

https://github.com/Unity-Technologies/Unity.Animation.Samples  

```c#
public class MyFirstAnimationClip : MonoBehaviour, IConvertGameObjectToEntity {
    public AnimationClip Clip;

    public void Convert(Entity entity, EntityManager dstManager, 
        GameObjectConversionSystem conversionSystem) {
            dstManager.AddComponentData(entity, new PlayClipComponent{
                Clip = ClipBuilder.AnimationClipToDenseClip(Clip)
            });
    }
}
```

#### 生命周期

还是自己打印日志清楚一点  
Initialization在引擎初始化最后执行  
SimulationSystemGroup在引擎Update最后执行  
LateSimulationSystemGroup在引擎LateUpdate之前执行  

```c#
//[UpdateInGroup(typeof(InitializationSystemGroup))]
//[UpdateInGroup(typeof(SimulationSystemGroup))]
[UpdateInGroup(typeof(PresentationSystemGroup))]
public class MySystem : ComponentSystem {}

[BurstCompile][RequireComponentTag(typeof(Tag_Button))]
struct ActivateButtonJob : IJobForEachWithEntity<MouseClicked> {
    public EntityCommandBuffer.Concurrent ECBSimulationBegin;
    public EntityCommandBuffer.Concurrent ECBSimulationEnd;

    public void Execute(Entity entity, int index, [ReadOnly] ref MouseClicked mouse) {
        ECBSimulationBegin.AddComponent(index, target.Entity, typeof(Tag_Activated));
        ECBSimulationEnd.RemoveComponent(index, target.Entity, typeof(Tag_Activated));
    }
}
```

<div id="index_documents"></div>

# 文档与博客

### Hybrid Renderer  
```
Unity 2020.1.0b3以上添加宏 ENABLE_HYBRID_RENDERER_V2
如果entity拥有组件 LocalToWorld RenderMesh RenderBounds
HybridRenderer自动条件需要的渲染组件，将处理后的entities添加到批处理
用Unity现有的渲染体系去渲染它们

如果要运行时添加实体，最好实例化prefab，而不是从头构建实体
转换期间，prefab已经转换为最佳数据布局，从而提高性能

HybridRenderer包含将各种GameObject转换为entities的转换系统
如果要将GameObject转换为entities，可以将他们放进子场景，
或者给子场景添加ConvertToEntity组件
子场景在编辑器中转换并存储到磁盘
ConvertToEntity会导致运行时转换
Unity编辑器中的转换可显著提高场景加载性能

转换过程：
1转换系统将MeshRenderer和MeshFilter组件转换成entity上的RenderMesh组件
依赖于Project中设置的渲染管道，转换系统还可能添加其他渲染依赖的组件
2转换系统将GameObject层次结构中的LODGroup组件转换为MeshLODGroup组件
LODGroup组件引用的每个entity都有一个MeshLODComponent
3转换系统将每个Transform转换为entity的LocalToWorld
根据变换的属性，转换系统还可能添加Translate,Rotation,NonUniformScale组件
```

```
Hybrid Renderer V1
支持的特性有限，不再继续开发
不支持：
Motion blur 运动模糊  (motion vectors missing) 运动矢量缺失
Temporal antialiasing TXAA抗锯齿 (motion vectors missing) 运动矢量缺失
LOD Fade
Lightmaps
RenderLayer (layered lighting) 分层照明
TransformParams (correct lighting for inverse scale) 反比例的正确照明
URP不支持的功能：
Point lights and spot lights
Light probes
Reflection probes

只支持基于 ShaderGraph 的shader
内建shader像是 HDRP Lit 不支持

如何设置shaders
不同版本操作不同
2019.3+ 在 ShaderGraph 自定义属性 勾选 Hybrid Instanced

如果哪个shader或者material没设置正确，可能有显示问题
还没有报错信息
常见问题：
1所有实体渲染到 (0,0,0)位置
2闪烁或者颜色不正确，特别在DX12，Vulkan，Metal
3闪烁/拉伸多边形，特别在DX12，Vulkan，Metal

重要!!!
Universal Render Pipeline 通用渲染管道URP
Unity2020.1和SRP8.0.0增加了对URP的官方HybridRendererV1的支持
旧版本问题多
```

```
Hybrid Renderer V2
2020.1中的新技术，在主要开发中
新的GPU-persistent data model，GPU持久化模型
直接从 Burst C# Jobs增量更新到显存

V1的主线程瓶颈已经不存在
渲染线程性能也得到改善
新数据模型支持我们从C#提供shader的built-in数据
使我们能实现缺少的HDRP和URP功能

V2已经完全兼容以下着色器
ShaderGraph, HDRP/Lit, HDRP/Unlit, URP/Lit, URP/Unlit
并将支持更多

新特性 2020.1+ hybridRenderer0.3.6：
Official URP support (minimal feature set)
Support for non-ShaderGraph shaders
Motion blur (motion vectors)
Temporal antialiasing (motion vectors)
RenderLayer (layered lighting)
TransformParams (correct lighting for inverse scale)
Hybrid Entities: Subscene support for Light, Camera and other managed components
混合实体：对光，相机，其他托管组件的子场景支持
DisableRendering component (disable rendering of an entity)

重要！！！
实验性功能，已经在DX11，Vulkan，Matal下的编辑器和独立版本中验证
目标是验证2020.2的DX12，移动设备，独立平台
```

```
HDRP & URP material 属性重载
V2支持每个实体重载各种HDRP和URP材质属性

有一个内置的IComponentData组件库
可以添加到实体以重载材质属性

可以通过C#/Burst代码
在运行时 setup and animate 材质

HDRP支持重载的属性:
AlphaCutoff
AORemapMax
AORemapMin
BaseColor
DetailAlbedoScale
DetailNormalScale
DetailSmoothnessScale
DiffusionProfileHash
EmissiveColor
Metallic
Smoothness
SmoothnessRemapMax
SmoothnessRemapMin
SpecularColor
Thickness
ThicknessRemap
UnlitColor (HDRP/Unlit)

URP支持重载的属性:
BaseColor
BumpScale
Cutoff
EmissionColor
Metallic
OcclusionStrength
Smoothness
SpecColor

如果想重载这里不支持的属性，
可以通过自定义ShaderGraph材质重载
勾选Hybrid Instanced

创建自定义ShaderGraph属性，在IComponentData中暴露
可以写C#/Burst代码控制自定义的shader的输入
```

```c#
//重要！！！
//每个自定义的ShaderGraph属性
//创建匹配的IComponentData结构体
//勾选Hybrid Instanced

//如果不这样，V1数据未初始化，会闪烁
//V2数据用0填充

[MaterialProperty("_Color', MaterialPropertyFormat.Float4)]
public struct MyOwnColor : IComponentData {
    public float4 Value;
}

class AnimateMyOwnColorSystem : SystemBase {
    protected override void OnUpdate() {
        Entities.ForEach((ref MyOwnColor color, in MyAnimationTime t)=>{
            color.Value = new float4(
                math.cos(t.Value + 1.0f),
                math.cos(t.Value + 2.0f),
                math.cos(t.Value + 3.0f),
                1.0f);
        })
        .Schedule();
    }
}
```

#### ShaderGraph

图形化着色器，URP和HDRP自带  
和builtin shader不兼容  

### ECS

#### 实体查询  
```c#
EntityQuery query = GetEntityQuery(typeof(RotationQuaternion), ComponentType.ReadOnly<RotationSpeed>());

var queryDescription = new EntityQueryDesc{
    None = new ComponentType[] { typeof(Frozen) },
    All = new ComponentType[] { typeof(RotationQuaternion), ComponentType.ReadOnly<RotationSpeed>() }
};
EntityQuery query = GetEntityQuery(queryDescription);
// 不要在EntityQueryDesc中包括可选组件，要处理可选组件，请使用ArchetypeChunk
// 同一个块中的所有实体都具有相同的组件，所以检查一个可选组件
// 一个块只需要一次，而不是每个实体查询一次
```

```c#
// Options变量
// IncludePrefab ：包括原型，该原型包含特殊的Prefab标签组件
// IncludeDisabled ：包括原型，该原型包含特殊的Disabled标签组件
// FilterWriteGroup ：考虑查询中任何组件的WriteGroup
//     仅包含明确查询的那些实体，排除具有任何其他组件的实体
//     虽然可以使用None，但是写入组无需更改其他系统使用的查询
//     写入组这是以重拓展现有系统的机制，例如：如果C1和C2是在另一个系统中定义的，则可以将
//     C3与C2放在同一个写入组中以更改C1的更新方式。对于添加C3组件的任何实体，系统都会更新C1，
//     而原始系统不会。对于其他没有C3的实体，原始系统像以前一样更新C1
// 解释：C1， C1+C3不一起更新，有不同的更新方法
public struct C1 : IComponentData {}

[WriteGroup(typeof(C1))]
public struct C2 : IComponentData {}

[WriteGroup(typeof(C1))]
public struct C3 : IComponentData {}

public class ECSSystem : SystemBase {
    protected override void OnCreate() {
        var queryDescription = new EntityQueryDesc{
            All = new ComponentType[] { ComponentType.ReadWrite<C1>(),
                                        ComponentType.ReadOnly<C3>() },
            Options = EntityQueryOptions.FilterWriteGroup
        };
        var query = GetEntityQuery(queryDescription);
    }

    protected override void OnUpdate() {
        return default;
    }
}
```

```c#
//合并查询
var desc1 = new EntityQueryDesc {
    All = new ComponentType[] { typeof(RotationQuaternion) }
};

var desc2 = new EntityQueryDesc {
    All = new ComponentType[] { typeof(RotationSpeed) }
};

EntityQuery query = 
    GetEntityQuery(new EntityQueryDesc[] { desc1, desc2 });
```

```c#
//缓存查询不考虑过滤器设置，如果在查询上设置过滤器，
//下次GetEntityQuery将设置相同的过滤器
//ResetFileter清除现有过滤器

//Entities.ForEach
//系统类之外，可以通过EntityManager创建查询
EntityQuery query = entityManager.CreateEntityQuery(typeof(RotationQuaternion), ComponentType.ReadOnly<RotationSpeed>());
//在系统类内部，可以从系统获取查询
//系统会缓存创建的所有查询并且返回缓存的实例
public class RotationSpeedSys : SystemBase {
    private EntityQuery query;
    protected override void OnUpdate() {
        float deltaTime = Time.DeltaTime;

        Entities
            .WithStoreEntityQueryInField(ref query)
            .ForEach(
            (ref RotationQuaternion rotation, in RotationSpeed speed)=> {
                rotation.Value
                    = math.mul(
                        math.normalize(rotation.Value),
                        quaternion.AxisAngle(math.up(),
                            speed.RadiansPerSecond * deltaTime)
                    );
            })
            .Schedule();
    }
}

//IJobChunk
//使用GetEntityQuery
public class RotationSystem : SystemBase {
    private EntityQuery query;
    protected override void OnCreate() {
        query = GetEntityQuery(typeof(RotationQuaternion),
            ComponentType.ReadOnly<RotationSpeed>());
    }
    protected override void OnUpdate() {
        return default;
    }
}
```

```c#
// 共享组件过滤器：根据共享组件的特定值过滤实体集
// 更改过滤器：根据特定组件类型的值是否已经更改过来过滤实体集
// SetSharedComponentFilter传入ISharedComponent类型的结构
// 最多添加两个不同的共享组件

// 可以随时更改过滤器，然后重新ToComponentDataArray<T> 或者 ToEntityArray
struct SharedGrouping : ISharedComponentData {
    public int Group;
}

class ImpulseSystem : SystemBase {
    EntityQuery query;
    protected override void OnCreate() {
        query = GetEntityQuery(typeof(Position),
            typeof(Displacement),
            typeof(SharedGrouping));
    }
    protected override void OnUpdate() {
        // Only iterate over entities that have the SharedGrouping data set to 1
        query.SetSharedComponentFilter(new SharedGrouping { Group = 1 });

        var positions = query.ToComponentDataArray<Position>(Allocator.Temp);
        var displacements = query.ToComponentDataArray<Displacement>(Allocator.Temp);

        for (int i = 0; i < positions.Length; i++) {
            positions[i] = new Position{ Value = positions[i].Value + displacements[i].Value };
        }
    }
}
```

```c#
//更换过滤器
//如果仅在组件更改后才需要更新实体，
//可以使用SetChangedVersionFilter函数将该组件添加到EntityQuery过滤器中

//仅包含来自另一个系统已经写入到Translation组件的块的实体
EntityQuery query;
protected override void OnCreate() {
    query = GetEntityQuery(typeof(LocalToWorld),
        ComponentType.ReadOnly<Translation>());
    query.SetChangedVersionFilter(typeof(Translation));
}
// 为了提高效率，更改过滤器适用于整个块，而不适用于单个实体。
// 更改筛选器还仅检查系统是否已经运行了已声明对组件进行写访问的系统
// 而不检查它是否实际上更改了任何数据
// 换句话说，如果另一个具有写入该类型组件能力的Job访问该块
// 则更改过滤器将包括该块中的所有实体。
// 这就是为什么应该始终声明对不需要修改的组件的只读访问权限的原因
```

```c#
//执行查询
ToEntityArray 返回所选实体的数组
ToComponentDataArray返回所选实体的类型的组件的数组
CrateArchetypeChunkArray返回包含所选实体的所有块。
因为查询操作的原型，共享组件值和更改筛选器对于块中的所有实体都是相同的，
所以在返回的块中存储的实体集与ToEntityArray返回的实体集完全相同
```

#### 世界
一个EntityManager  
一系列Systems  
可以创建一个模拟世界以及一个渲染或演示世界  

默认情况下，一运行就会创建一个世界，如果有CompnentSystemBase  
但是可以禁用  
#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_RUNTIME_WORLD  
#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_EDITOR_WORLD   
#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP  

#### 组件

##### 简介

```
IComponentData 通用 和 块组件
IBufferElementData 将 动态缓冲区与实体相关联
ISharedComponentData 按原型中的值对实体进行分类或分组
ISystemStateComponentData 将特定于系统的状态与实体相关联，并检测何时创建或销毁单个实体
ISharedSystemStateCompnentData 共享状态和系统状态数据的组合
Blob assets 从技术上讲，并不是组件，但是可以用Blob资产存储数据，
    Blob资产可以由一个或多个组件使用BlobAssetReference进行引用，并且是不可变的
    可用Blob资产在资产之间共享数据并访问C# Jobs 中的数据

EntityManager将组件的独特组合组织到 Archtypes 中
将具有相同原型的所有实体的组件一起存储在称为 chunk 的内存块中
给定块中的实体均具有相同的组件原型

共享组件和块组件是例外，ECS将它们存储在块外部
对于chunk 一即是众，众即是一
动态缓冲区可以存储在块之外
```

##### 通用组件

```
通用组件 ComponentData
接口 IComponentData

非托管IComponentData
    不得包含对托管对象的引用
    修改数据：
    var transform = group.transform[index];//Read
    transform.heading = playerInput.move;//Modify
    transform.position += deltaTime * playerInput.move * settings.playerMoveSpeed;
    group.transform[index] = transform;//Write

托管的 IComponentData
    使用 class 而不是声明 struct
    有助于将现有代码以零碎方式移植到ECS，与不适合的托管数据进行互操作 ISharedComponentData
    或者为数据布局提供原型
    如果不需要托管组件支持，设置宏 UNITY_DISABLE_MANAGED_COMPONENTS

    不能用Burst
    不能用在Job structs
    不能用块内存
    需要垃圾收集

    应该限制托管组件的数量，尽可能使用blittable类型
    必须实现IEquatable<T>接口并重载Object.GetHashCode()
    必须默认可构造

    必须在主线程射设置Component，使用EntityManager或EntityCommandBuffer
    由于是引用类型，和ISharedComponentData不同，可以更改组件的值，无需修改块，不会创建同步点

    尽管存储逻辑不是块，仍然受到EntityArchetype定义
    创建新的托管组件仍会创建新的原型，并将实体移至新的Chunk

有关示例，参阅 Packages/com.unity.entities/Unity.Entities/IComponentData.cs
```

##### 共享组件

```
共享组件 SharedComponentData
具有相同共享数据值的实体在同一个块中

共享组件Rendering.RenderMesh定义了mesh,material,receiveShadows
渲染时，一起处理所有具有相同字段值的3D对象，EntityManager将匹配的实体放置在内存中
以便渲染系统可用有效地对其进行迭代

！！！如果过度使用共享组件，则可能导致不佳的块利用率。
因为，基于原型 和 每个共享组件字段 的 每个唯一值的内存块数量的组合扩展
避免添加将实体分类到共享组件类别中不需要的任何字段

如果从实体中添加或删除组件，或更改共享组件的值，EntityManager会将实体移动到其他块
在必要时创建新块

应该将IComponentData用于不同实体之间变化的数据，例如，存储世界位置，生命值，例子生存时间
共享组件用于：DOTS软件包的Boids演示，许多实体从同一个Prefab实例化，RenderMesh都是相同的

[System.Serializable]
public struct RenderMesh : ISharedComponentData {
    public Mesh mesh;
    public Material material;
    public ShadowCastingMode castShadows;
    public bool receiveShadows;
}

！！！
SharedComponentData是每个块存储一次，而不是每个实体都存储，实体上内存开销为0
可用EntityQuery遍历具有相同类型的所有实体，还可以用EntityQuery.SetFilter()专门针对具有特定SharedComponentData值的实体进行迭代
由于数据布局的原因，此迭代开销很低
可用EntityManager.GetAllUniqueSharedComponents来检索SharedComponentData添加到任何活动实体的所有唯一性
ECS自动引用计数SharedComponentData
SharedComponentData很少改变，如果需要更改，需要使用 memcpy 将该实体的所有ComponentData复制到另一个块中

有关此示例，参阅 Packages/com.unity.entities/Unity.Rendering.Hybrid/RenderMeshSystemV2.cs
```

##### 系统状态组件

```
系统状态组件
SystemStateComponentData
跟踪系统内部的资源，根据需要创建和销毁这些资源，无需依赖各个回调

SystemStateComponentData  SystemStateSharedComponentData
类比ComponentData SharedComponentData
但是SystemStateComponentData在销毁实体时，ECS不会删除

DestroyEntity：
当entity被销毁时，ECS通常
1 查找引用特定实体id的所有组件
2 删除那些组件
3 回收实体ID以供重用

但是，如果SystemStateComponentData存在，ECS不会回收ID，这使系统有机会清理与
实体ID相关的任何资源或状态。ECS仅在SystemStateComponent删除entityID后才重新使用

何时使用系统状态组件？
系统可能需要保持状态，基于ComponentData，例如，可用分配资源
系统还需要能将状态作为值进行管理，其他系统可能会进行状态更改，
例如，当组件的值更改时，添加删除相关组件时

没有回调，是ECS设计规划的重要组成部分
SystemStateComponent预期的一般用法 将镜像用户组件，以提供内部状态
例如：给定 FooComponent 是ComponentData 用户分配
FooStateComponent 是SystemComponentData 系统分配

检测何时添加组件？
创建组件时，系统状态组件不存在。系统更新没有系统状态组件的实体，添加系统状态组件，和需要的内部状态

检测何时删除组件？
删除组件时，系统状态组件还在。系统更新只有系统状态组件的实体，删除系统状态组件，修复需要的内部状态

检测实体何时被破坏？
SystemStateComponentData在DestroyEntity删除最后一个组件之前，不会删除，并且不会回收实体。
再删除SystemStateComponentData，删除实体

struct FooStateComponent : IShareStateComponentData {
}
系统状态组件数据将在创建它的系统之外只读

struct FooStateSharedComponent : ISystemStateSharedComponentData {
    public int Value;
}
```

```c#
//例子，使用系统状态组件
// m_newEntities选择具有通用用途但不具有系统状态组件的实体。该查询查找系统之前从未见过的新实体
// 系统使用添加了系统状态组件的新实体查询来运行作业
// m_activeEntities选择同时具有通用和系统状态组件的实体。在实际的应用程序中，其他系统可能是处理或销毁实体的系统。
// m_destroyedEntities选择具有系统状态但不具有通用组件的实体。由于系统状态组件永远不会自己添加到实体，因此此系统
// 或其他系统必须删除此查询选择的实体。系统重用销毁的实体查询以运行作业并从实体中删除系统状态组件，这使ECS代码可用
// 回收实体标识符
//例子在系统中不维护任何状态，系统状态组件的目的之一是跟踪和是需要分配或清除持久性资源
using Unity.Entities;
using Unity.Jobs;
using Unity.Collections;

public struct GeneralPurposeComponentA : IComponnetData {
    public int LeftTime;
}

public struct StateComponentB : ISystemStateComponnet {
    public int State;
}

public class StatefulSystem : SystemBase {
    private EntityCommandBufferSystem ecbSource;

    protected override void OnCreate() {
        ecbSource = World.GetExistingSystem<EndSimulationEntityCommandBufferSystem>();

        // Create some test entities
        // This runs on the main thread, but it is still faster to use a command buffer
        EntityCommandBuffer creationBuffer = new EntityCommandBuffer(Allocator.Temp);
        EntityArchetype archetype = EntityManager.CreateArchetype(typeof(GeneralPurposeComponentA));
        for (int i = 0; i < 10000; i++) {
            Entity newEntity = creationBuffer.CreateEntity(archetype);
            creationBuffer.SetComponent<GeneralPurposeComponentA> (
                newEntity,
                new GeneralPurposeComponentA() { Lifetime = i }
            );
        }
        // Execute the command buffer
        creationBuffer.Playback(EntityManager);
    }

    protected override void OnUpdate() {
        EntityCommandBuffer.ParalleWriter parallelWriterECB = ecbSource.CreateCommandBuffer().AsParallelWriter();

        // Entities with GeneralPurposeComponentA but not StateComponentB
        Entities
            .WithNone<StateComponentB>()
            .ForEach(
                (Entity entity, int entityInQueryIndex, in GeneralPurposeComponentA gpA)=>{
                    // Add an ISystemStateComponentData instance
                    parallelWriterECB.AddComponent<StateComponentB>(
                        entityInQueryIndex,
                        entity,
                        new StateComponentB() { State = 1 }
                    );
                })
            .ScheduleParallel();
        ecbSource.AddJobHandleForProducer(this.Dependency);

        // Create new command buffer
        parallelWriterECB = ecbSource.CreateCommandBuffer().AsParallelWriter();

        // Entities with both GeneralPurposeComponentA and StateComponentB
        Entities
            .WithAll<StateComponentB>()
            .ForEach(
                (Entity entity, int entityInQueryIndex, ref GaneralPurposeComponentA gpA)=>{
                    // Process entity, in this case by decrementing the Lifetime count
                    gpA.Lifetime--;

                    // If out of time, destroy the entity
                    if (gpA.Lifetime <= 0) {
                        parallelWriterECB.DestroyEntity(entityInQueryIndex, entity);
                    }
                })
            .ScheduleParallel();
        ecbSource.AddJobHandleForProducer(this.Dependency);

        // Create new command buffer
        parallelwriterECB = ecbSource.CreateCommandBuffer().AsParallelWriter();

        // Entities with StateComponentB but not GeneralPurposeComponentA
        Entities
            .WithAll<StateComponentB>()
            .WithNone<GeneralPurposeComponentA>()
            .ForEach(
                (Entity entity, int entityInQueryIndex)=>{
                    // This system is responsible for removing any ISystemStateComponentData instances it adds
                    // Othewise, the entity is never truly destroyed.
                    paralleWriterECB.RemoveComponent<StateComponentB>(entityInQueryIndex, entity);
                })
            .ScheduleParallel();
        ecbSource.AddJobHandleForProducer(this.Dependency);
    }

    protected override void OnDestroy() {
        // Implement OnDestroy to cleanup any resources allocated by this system.
        // (This simplified example does not allocate any resources, so there is nothing to clean up.)
    }
}
```

##### 动态缓冲区组件

```
动态缓冲区组件
BufferElementData
使用动态缓冲区组件将类似数组的数据与实体相关联。动态缓冲区是ECS组件，可用容纳可变数量的元素，并根据需要自动调整大小。
public struct IntBufferElement : IBufferElementData {
    public int Value;
}

可用在实体查询中，或者在添加删除缓冲区组件时使用
必须用不同函数访问缓冲区组件，并且这些函数提供了DynamicBuffer实例

指定内部容量，使用 InternalBufferCapacity属性
内部容量定义了动态缓冲区与实体的其他组件一起储存在ArchetypeChunk中的元素
除了内部容量，缓冲区还会在当前块之外分配一个堆内存块，并将所有现有元素移走。
ECS会自动管理该外部缓冲区内存，在删除缓冲区时释放内存。

！！！如果缓冲区的数据不是动态的，则可用使用Blob asset代替动态缓冲区
```

```c#
//声明缓冲区元素类型
// internalBufferCapacity specifies how many elements a buffer can have before
// the buffer storage is moved outside the chunk.
[InternalBufferCapacity(8)]
public struct MyBufferElement : IBufferElementData {
    // Actual value each buffer element with store.
    public int Value;

    // The following implicit conversions are optional, but can be convenient.
    public static implicit operator int(MyBufferElement e) {
        return e.Value;
    }

    public static implicit operator MyBufferElement(int e) {
        return new MyBufferElement{ Value = e };
    }
}

//向实体添加缓冲区类型
EntityManager.AddBuffer<MyBufferElement>(entity);
//使用原型
Entity e = EntityManager.CreateEntity(typeof(MyBufferElement));

//使用[GenerateAuthoringComponent]属性
// 为仅包含一个字段的简单IBufferElementData实现生成创作组件。
// 设置后，可将ECS IBufferElementData添加到GameObject，以便于在
// 编辑器设置缓冲区元素。

//可以将这个直接添加奥编辑器中的GameObject
[GenerateAuthoringComponent]
public struct IntBufferElemetn : IBufferElementData {
    public int Value;
}
//在后台，Unity生成一个IntBufferElementAuthoring的MonoBehaviour
//该类公开一个List<int>，当GameObject转换为entity，该列表转换为
// Dynamicbuffer<IntBufferElement>

//!!!
// 单个C#文件中只有一个组件可以具有 [GenerateAuthoringComponent]
// 并且不能包含另一个MonoBehaviour
// IBufferElementData对于包含多个字段的类型，无法自动生成
// IBufferElementData无法为具有显示布局的类型自动生成
```

```c#
// 使用EntityCommandBuffer
// 将命令添加到entity的命令缓冲区时，可以添加或设置缓冲区组件
// 使用Addbuffer为实体创建一个新的缓冲区，这将更改实体的原型。
// 使用SetBuffer清楚现有，必然存在的缓冲区，并在其位置创建新的空缓冲区
// 这两个函数都返回一个DynamicBuffer实例，可以用该实例填充新缓冲区

using Unity.Entities;
using Unity.Jobs;

public class CreateEntitiesWithBuffers : SystemBase {
    // A command buffer system executes command buffers in its own OnUpdate
    public EntityCommandBufferSystem CommandBufferSystem;

    protected override void OnCreate() {
        CommandBufferSystem = World.DefaultGameObjectInjectionWorld.GetExistingSystem<EndSimulationEntityCommandBufferSystem>();
    }

    protected override void OnUpdate() {
        // The command buffer to record commands,
        // which are executed by the command buffer system later in the frame
        EntityCommandBuffer.ParalleWriter commandBuffer = CommandBufferSystem.CreateCommandBuffer().AsParalleWriter();
        // The DataToSpawn component tells us how many entities with buffers to create
        Entities.ForEach((Entity spawnEntity, int entityInQueryIndex, in DataToSpawn data)=>{
            for (int e = 0; e < data.EntityCount; e++) {
                // Create a new entity for the command buffer
                Entity newEntity = commandBuffer.CreateEntity(entityInQueryIndex);

                // Create the dynamic buffer and add it to the new entity
                DynamicBuffer<MyBufferElement> buffer = commandBuffer.AddBuffer<MyBufferElement>(entityInQueryIndex, newEntity);

                // Reinterpret to plain int buffer
                DynamicBuffer<int> intBuffer = buffer.Reinterpret<int>();

                // Optionally, populate the dynamic buffer
                for (int j = 0; j < data.ElementCount; j++) {
                    intBuffer.Add(j);
                }
            }

            // Destroy the DataToSpawn entity since it has down its job
            commandBuffer.DestroyEntity(entityInQueryIndex, spawnEntity);
        }).ScheduleParallel();

        CommandBufferSystem.AddJobHandleForProducer(this.Dependency);
    }
}
```

```c#
//访问缓冲区

//EntityManager
DynamicBuffer<MyBufferElemetn> dynamicBuffer = EntityManager.GetBuffer<MyBufferElement>(entity);

//BufferFromEntity
BufferFromEntity<MyBufferElement> lookup = GetBufferFromEntity<MyBufferElement>();
var buffer = lookup[entity];
buffer.Add(17);
buffer.RemoveAt(0);

//SystemBase
public class DynamicBufferSystem : SystemBase {
    protected override void OnUpdate() {
        var sum = 0;

        Entities.ForEach((DynamicBuffer<MyBufferElement> buffer)=>{
            for (int i = 0; i < buffer.Length; i++) {
                sum += buffer[i].Value;
            }
        }).Run();

        Debug.Log("Sum of all buffers: " + sum);
    }
}
// sum在实例中可以直接写入捕获的变量，因为使用Run
// 如果安排在作业中运行，则要写入NativeArray
```

```c#
//IJobChunk
// 将缓冲区数据类型传递给作业，然后使用该数据类型获取BufferAccessor
// 缓冲区访问器类似于数组，可提供对当前块中所有动态缓冲区的访问

// IJobChunk还可以在每个块上并行运行，因此在示例中，首先将每个缓冲区的中间和存贮
// 在NativeArray中，然后使用第二个作业来计算最终和。
// 这种情况，中间数组为每一个块保存一个结果，而不是为每一个实体保存一个结果
public class DynamicBufferJobSystem : SystemBase {
    private EntityQuery query;

    protected override void OnCreate() {
        // Create a query to find all entities with a dynamic buffer
        // containing MyBufferElement
        EntityQueryDesc queryDescription = new EntityQueryDesc();
        queryDescription.Add = new[] {ComponentType.ReadOnly<MyBufferElement>()};
        query = GetEntityQuery(queryDescription);
    }

    public struct BuffersInChunks : IJobChunk {
        // The data type and safety object
        public BufferTypeHandle<MyBufferElement> BufferTypeHandle;

        // An array to hold the output, intermediate sums
        public NativeArray<int> sums;

        public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
            // A buffer accessor is a list of all the buffers in the chunk
            BufferAccessor<MyBufferElement> buffers = chunk.GetBufferAccessor(BufferTypeHandle);

            for (int c = 0; c < chunk.Count; c++) {
                // An individual dynamic buffer for a specific entity
                DynamicBuffer<MyBufferElement> buffer = buffers[c];
                for (int i = 0; i < buffer.Length; i++) {
                    sums[chunkIndex] += buffer[i].Value;
                }
            }
        }
    }

    // Sums the intermediate results into the final total
    public struct SumResult : IJob {
        [DeallocateOnJobCompletion]
        public NativeArray<int> sums;
        public NativeArray<int> result;
        public void Execute() {
            for (int i = 0; i < sums.Length; i++) {
                result[0] += sums[i];
            }
        }
    }

    protected override void OnUpdate() {
        // Create a native array to hold the intermediate sums
        int chunksInQuery = query.CalculateChunkCount();
        NativeArray<int> intermediateSums = new NativeArray<int>(chunksInQuery, Allocator.TempJob);

        // Shcedule the first job to add all the buffer elements
        BuffersInChunks bufferJob = new BuffersInChunks();
        bufferJob.BufferTypeHandle = GetBufferTypeHandle<MyBufferElement>();
        bufferJob.sums = intermediateSums;
        this.Dependency = bufferJob.ScheduleParallel(query, this.Dependency);

        // Schedule the second job, which depends on the first
        SumResult finalSumJob = new SumResult();
        finalSumJob.sums = intermediateSums;
        NativeArray<int> finalSum = new NativeArray<int>(1, Allocator.Temp);
        finalSumJob.result = finalSum;
        this.Dependency = finalSumJob.Schedule(this.Dependency);

        this.CompleteDependency();
        Debug.Log("Sum of all buffers: " + finalSum[0]);
        finalSum.Dispose();
    }
}

// 重新解释缓冲区
// 安全，引用了原始数据
// 注意：强制类型转换必须有相同长度 比如 uint float
DynamicBuffer<int> intBuffer = EntityManager.GetBuffer<MyBufferElement>(entity).Reinterpret<int>();

// 缓冲区引用无效
// 每次结构更改都会使动态缓冲区的所有引用无效。
// 结构变化通常会导致实体从一个小块移动到另一个小块
// 小型动态缓冲区可以引用块中的内存，而不是主存中的内存
// 结构更改后需要重新获取
var entity1 = EntityManager.CrateEntity();
var entity2 = EntityManager.CrateEntity();

DynamicBuffer<MyBufferElement> buffer1 = EntityManager.AddBuffer<MyBufferElement>(entity1);
// This line causes a structural change and invalidates
// the previously acquired dynamic buffer
DynamicBuffer<MyBufferElement> buffer2 = EntityManager.AddBuffer<MyBufferElement>(entity1);
// This line will cause an error:
buffer1.Add(17);
```

##### 块组件

```
Chunk component data
与特定块关联
每个块存储一个，块中实体原型都有
因此如果实体中删除了块组件，实体会被移动到其他块
如果将块组件添加到实体，实体也会被移动到其他块，因为原型被更改

在实体中更改块组件的值，该块中所有实体的值都被改变
如果更改实体原型，以便于将其移入具有相同类型的新组块，那么目标组块中的现有值不受影响
！如果将实体移动到新创建的块，ECS会为该块创建一个新的块组件并分配其默认值

块组件和通用组件的主要区别在于，使用不同的功能来添加，设置和删除
```

```
//方法列表
Declaration     IComponentData

ArchetypeChunk methods
Read GetChunkComponentData<T>(ArchetypeCHunkComponentType<T>)
Check [HasChunkComponet<T>(ArchetypeChunkComponentType<T>)]
Write SetChunkComponentData<T>(ArchetypeChunkComponentType<T>)

EntityManager methods
Create AddChunkComponentData<T>(Entity)
Create AddChunkComponentData<T>(EntityQuery, T)
Create AddComponents(Entity, ComponentTypes)
Get type info [GetComponentTypeHandle]
Read [GetChunkComponentData<T>(ArchetypeChunk)]
Read GetChunkComponentData<T>(Entity)
Chunk HasChunkComponent<T>(Entity)
Delete RemoveChunkComponent<T>(Entity)
Delete RemoveChunkComponentData<T>(EntityQuery)
Write EntityManager.SetChunkComponentData<T>(ArchetypeChunk, T)
```

```c#
//声明块组件
public struct ChunkComponentA : IComponentData {
    public float Value;
}

//创建一个块组件
//要直接添加块组件，在目标块中使用一个entity，或使用选择一组目标块的EntityQuery
//不能再job里面添加块组件，也不能使用EntityCommandBuffer添加

//块组件可以作为EntityArchetype的一部分，或ECS用于创建entity的ComponentType列表的一部分
//ECS为每个块创建块组件，并存储具有该原型的实体

//通过这些方法使用ComponentType.ChunkComponent<T>或[ComponentType.ChunkComponentReadOnly<T>]
EntityManager.AddChunkComponentData<ChunkComponentA>(entity);
//使用此方法时，不能立即为块组件设置值

//EntityQuery
EntityQueryDesc ChunksWithoutComponentADesc = new EntityQueryDesc() {
    None = new ComponentType[] { ComponentType.ChunkComponent<ChunkComponentA>() }
};
EntityQuery ChunksWithoutChunkComponentA = GetEntityQuery(ChunksWithoutComponentADesc);

EntityManager.AddChunkComponentData<ChunkComponentA>(ChunksWithoutChunkComponentA,
    new ChunkComponentA() { Value = 4});
//使用此方法，可以为所有新块组件设置相同的初始值

//EntityArchetype
EntityArchetype ArchetypeWithChunkComponent = EntityManager.CreateArchetype(
    ComponentType.ChunkComponent(typeof(ChunkComponentA)),
    ComponentType.ReadWrite<GeneralPurposeComponentA>());
Entity newEntity = EntityManager.CreateEntity(ArchetypeWithChunkComponent);
//组件类型列表
ComponentType[] comTypes = {ComponentType.ChunkComponent<ChunkComponentA>(),
                            ComponentType.ReadOnly<GeneralPurposeComponentA>()}
Entity entity = EntityManager.CreateEntity(compTypes);
```

```c#
//读取块组件

//使用ArchetypeChunk实例
NativeArray<ArchetypeChunk> chunks = ChunksWithChunkComponentA.CreateArchetypeChunkArray(Allocator.TempJob);
foreach (var chunk in chunks) {
    var compValue = EntityManager.GetChunkComponentData<ChunkComponentA>(chunk);
}
chunks.Dispose();

//通过实体
if (EntityManager.HasChunkComponent<ChunkComponentA>(entity)) {
    ChunkComponentA chunkComponentValue = EntityManager.GetChunkComponentData<ChunkComponentA>(entity);
}
```

```c#
//更新块组件
//可以在引用块的基础上更新块组件
//IJobChunk中，可以调用[ArchetypeChunk.SetChunkComponnetData]
//在主线程上，可以使用EntityManager版本[EntityManager.SetChunkComponentData]
//！！！注意：无法使用 SystemBase Entities.ForEach访问块组件，因为无权访问ArchetypeChunk对象或EntityManager

//使用ArchetypeChunk实例
//和读取应该时一样的

//EntityManager
EntityManager.SetChunkComponentData<ChunkComponentA>(chunk, new ChunkComponentA() { Value = 7 });

//如果有一个块中的实体，而不是块本身，可以使用EntityManager获取包含该实体的块，再进行这一步操作

//！！！注意：如果只想读取而不是写入，则在定义实体查询时 应该使用
// ComponentType.ChunkComponentReadOnly
// 以避免创建不必要的作业调度约束
```

```c#
//删除块组件
EntityManager.RemoveChunkComponent

//可以删除 实体 的 块组件
//实体会移动到其他块

//可以删除 实体查询 选择的 块 中 所有块的组件
```

```c#
//在查询中使用块组件

//使用
ComponentType.ChunkComponent<T>;
[ComponentType.ChunkComponentReadOnly<T>]

EntityQueryDesc ChunkWithChunkComponentADesc = new EntityQueryDesc() {
    All = new ComponentType[] { ComponentType.ChunkComponent<ChunkComponentA>() }
};
```

```c#
//遍历块 设置块组件
// 可以创建一个实体查询，实体查询选择正确的块，使用EntityQuery对象
// 获取ArchetypeChunk实例的列表作为本机数组。ArchetypeChunk对象允许将新值写入块组件
public class ChunkComponentExamples : SystemBase {
    private EntityQuery ChunksWithChunkComponentA;
    protected override void OnCreate() {
        EntityQueryDesc ChunksWithComponentADesc = new EntityQueryDesc() {
            All = new ComponentType[] {ComponentType.ChunkComponent<ChunkComponentA>()}
        };

        ChunksWithChunkComponentA = GetEntityQuery(ChunksWithComponentADesc);
    }

    [BurstCompile]
    struct ChunkComponentCheckerJob : IJobChunk {
        public ComponentTypeHandle<ChunkComponentA> ChunkComponentATypeHandle;
        public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
            var compValue = chunk.GetChunkComponentData(ChunkComponentATypeHandle);

            var squared = compValue.Value * compValue.Value;
            chunk.SetChunkComponentData(ChunkComponentATypeHandle, new ChunkComponentA() { Value = squared })
        }
    }

    protected override void OnUpdate() {
        var job = new ChunkComponentCheckerJob() {
            ChunkComponentATypeHandle = GetComponentTypeHandle<ChunkComponentA>()
        };
        this.Dependency = job.Schedule(ChunksWithChunkComponentA, this.Dependency);
    }
}

//如果需要读取块中的组件以确定块组件的正确值，应该使用IJobEntityBatch
// 以下代码:包含 具有LocalToWorld组件的实体  的所有块  计算出轴对齐的边界框
public struct ChunkAABB : IComponentData {
    public AABB Value;
}

[UpdateInGroup(typeof(PresentationSystemGroup))]
[UpdateBefore(typeof(UpdateAABBSystem))]
public class AddAABBSystem : Systembase {
    EntityQuery queryWithoutChunkComponent;
    protected override void OnCreate() {
        queryWithoutChunkComponent = GetEntityQuery(new EntityQueryDesc() {
            All = new ComponentType[] { ComponentType.ReadOnly<LocalToWorld>() },
            None = new ComponentType[] { ComponentType.ChunkComponent<ChunkAABB>() }
        });
    }

    protected override void OnUpdate() {
        // This is a structual change and a sync point
        EntityManager.AddChunkComponentData<ChunkAABB>(queryWithoutChunkComponent, new ChunkAABB());
    }
}

[UpdateInGroup(typeof(PresentationSystemGroup))]
public class UpdateAABBSystem : SystemBase {
    EntityQuery queryWithChunkComponent;
    protected override void OnCreate() {
        queryWithChunkComponent = GetEntityQuery(new EntityQueryDesc() {
            All = new ComponentType[] { ComponentType.ReadOnly<LocalToWorld>(),
                                        ComponentTYpe.ChunkComponent<ChunkAABB>() }
        });
    }

    [BurstCompile]
    struct AABBJob : IJobChunk {
        [ReadOnly] public ComponentTypeHandle<LocalToWorld> LocalToWorldTypeHandleInfo;
        public ComponentTypeHandle<ChunkAABB> ChunkAabbTypeHandleInfo;
        public uint L2WChangeVersion;
        public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
            bool chunkHasChanges = chunk.DidChange(LocalToWorldTypeHandleInfo, L2WChangeVersion);

            if (!chunkHasChanges)
                reutrn; //early out if the chunk transforms haven't changed

            NativeArray<LocalToWorld> transforms = chunk.GetNativeArray<LocalToWorld>(LocalToWorldTypeHandleInfo);
            UnityEngine.Bounds bounds = new UnityEngine.Bounds();
            bounds.center = transforms[0].Position;
            for (int i = 1; i < transforms.Length; i++)
                bounds.Encapsulate(transforms[i].Position);
            chunk.SetChunkComponentData(ChunkAabbTypeHandleInfo, new ChunkAABB { Value = bounds.ToAABB() });
        }
    }

    protected override void OnUpdate() {
        var job = new AABBJob() {
            LocalToWorldTypeHandleInfo = GetComponentTypeHandle<LocalToWorld>(true),
            ChunkAabbTypeHandleInfo = GetComponentTypeHandle<ChunkAABB>(false),
            L2WChangeVersion = this.LastSystemVersion
        };
        this.Dependency = job.Schedule(queryWithChunkComponent, this.Dependency);
    }
}
```

#### 系统

##### 简介

```
实例化系统
使用 <系统属性> <父组> <该系统在父组中的顺序> 确定顺序
如果没有指定父项，则Unity将添加到默认世界的Simulation系统组中
可以使用属性禁用自动创建

更新循环由 父ComponentSystemGroup 驱动
ComponentSystemGroup是一种特殊的系统，负责更新子系统
组可以嵌套
系统从运行的世界中获取 时间 数据
时间由 UpdateWorldTimeSystem  更新
```

```
系统类型
实现游戏行为 数据转换 ：SystemBase
其他系统有特殊用途。
通常使用 EntityCommandBufferSystem  ComponentSystemGroup 类的 现有实例

SystemBase  创建系统要实现的基类

EntityCommandBufferSystem 为其他系统提供 EntityCommandBuffer 实例
每个默认系统组在其子系统列表的开头和结尾都维护一个  实体命令缓冲区系统
这样可以对结构更改进行分组，以使它们在框架中产生更少的  同步点

ComponentSystemGroup 为其他系统提供嵌套的组织和更新顺序
默认情况下，UnityECS创建多个组件系统组

GameObjectConversionSystem 将游戏的基于GameObject的编辑器内表示
转换为高效的基于实体的运行时表示 游戏转换系统在Unity编辑器中运行
```

##### 建立系统

```
        SystemBase
        OnCreate 可选               创建系统时调用
        OnStartRunning() <--        在第一个OnUpdate之前 每次系统恢复运行时 调用
循环    OnUpdate 必选       |        只要有工作 ShouldRunSystem控制 且 Enabled=true
        OnStopRunning()   --        系统停止更新 Enabled=false 或 找不到查询的实体
        OnDestroy()                 销毁

系统组触发子系统的OnUpdate，系统组Enabled=false，也会更改子系统状态
子系统也可以独立于 父群组 改变装，详细请看 系统更新顺序

所有系统事件都在主线程上运行
理想情况下，OnUpdate函数可以调度 jobs 执行大部分工作
要从系统安排 job：
Entities.ForEach  迭代ECS组件数据 最简单的方法
Job.WithCode 将lambda函数作为单个后台Job执行
IJobChunk 低级 的机制，逐块迭代ECS组件数据
C# Jobs System 创建 Schedule 普通的 C# jobs
```

```c#
// Entities.ForEach
public struct Position : IComponentData {
    public float3 Value;
}
public struct Velocity : IComponentData {
    public float3 Value;
}

public class ECSSystem : SystemBase {
    protected override void OnUpdate() {
        // Local variable captured in ForEach
        float dT = Time.DeltaTime;

        Entities
            .WithName("Update_Displacement")
            .ForEach(
                (ref Position position, in Velocity velocity)=>
                {
                    position = new Position()
                    {
                        Value = position.value + velocity.Value * dT
                    };
                }
            )
            .ScheduleParallel();
    }
}
```

##### 使用实体

```c#
// Entities.ForEach
// 传入一个 lambda 函数， 使用 Schedule 和 ScheduleParallel 安排 job
// 或者 Run 立即在主线程执行
class ApplyVelocitySystem : SystemBase {
    protected override void OnUpdate() {
        Entities
            .ForEach(
                (ref Translation translation, in Velocity velocity) =>
                {
                    translation.Value += velocity.Value;
                }
            )
            .Schedule();
    }
}
//!!! 注意 ref 和 in
// ref 是打算写的数据
// in 是只读的
// 将组件标记为只读可以帮助job 高效执行

// 使用 WithAll WithAny WithNone 细化查询
// 示例： 选择具有以下成分的实体 Destination Source LocalToWorld
// 有至少一个就行
// 但是不能有 LocalToParent
Entities.WithAll<LocalToWorld>()
    .WithAny<Rotation, Translation, Scale>()
    .WithNone<LocalToParent>()
    .ForEach(
        (ref Destination outputData, in Source inputData)=>
        {
            // 只能在 lambda函数 内部访问 Destination 和 Source 组件
        }
    )
    .Schedule();

//访问EntityQuery对象
// 如果要访问 Entities.ForEach创建的 EntityQuery对象
// 使用 [WithStoreEntityQueryInField (ref query)] 注意 ref
// EntityQuery是在 OnCreate 中创建的， 此方法提供该查询的副本，可以随时使用

// 示例： 如何访问Entities.ForEach 隐式构造的 EntityQuery对象
// 使用 EntityQuery对象调用 CalculateEntityCount 方法， 用这个计数创建NativeArray
private EntityQuery query;
protected override void OnUpdate() {
    int dataCount = query.CalculateEntityCount();
    NativeArray<float> dataSquared = new NativeArray<float>(dataCount, Allocator.Temp);

    Entities
        .WithStoreEntityQueryInField(ref query)
        .ForEach(
            (int entityInQueryIndex, in Data data)=>
            {
                dataSquared[entityInQueryIndex] = data.Value * data.Value;
            }
        )
        .ScheduleParallel();

    Job
        .WithCode(
            ()=>
            {
                // Use dataSquared array
                var v = dataSquared[dataSequred.Length - 1];
            }
        )
        .WithDisposeOnCompletion(dataSquared)
        .Schedule();
}
```

```
可选组件
无法创建指定可选组件的查询  使用  WithAny<T, U>
无法在 lambda 中 访问这些组件
如果需要读取或写入 可选组件
可以将 Entities.ForEach 拆分成多个
每个可选组件用一个

例如：
如果有两个可选组件
需要三个ForEach
第一个包含A
第二个包含B
第三个包含AB

另一种是选择使用 IJobChunk 逐块进行迭代
```

```c#
// 变更筛选
// SystemBas 上次运行后 该组件 另一个实体 发生更改时 处理这个实体
// 可以用 WithChangeFilter<T> 启用 筛选
// lambda函数使用的参数需要在参数列表中，或者必须时WithAll<T>语句的一部分
Entities
    .WithChangeFilter<Source>()
    .ForEach(
        (ref Destination outputData, in Source inputData)=>
        {
            // Do work
        }
    )
    .ScheduleParallel();
//!!! 最多支持两种类型的更改过滤
// 更改过滤应用于块级别
// 如果有任何代码通过 写访问 访问块中的某个组件
// 则该块中该组件的类型被标记为已经更改，即使并未更改
// 因此： 筛选对写访问 无效
```

```c#
//共享组件过滤
// 具有共享组件的实体
// WithSharedComponentFilter

//示例 选择按同类组 ISharedComponentData分组的实体
// lambda函数根据实体的同类群组设置 DisplayColor IComponentData
public class ColorCycleJob : SystemBase {
    protected override void OnUpdate() {
        List<Cohort> cohorts = new List<Cohort>();
        EntityManager.GetAllUniqueSharedComponentData<Cohort>(cohorts);
        foreach (Cohort cohort in cohorts) {
            DisplayColor newColor = ColorTable.GetNextColor(cohort.Value);
            Entities
                .WithSharedComponentFilter(cohort)
                .ForEach(
                    (ref DisplayColor color)=>
                    {
                        color = newColor;
                    }
                )
                .ScheduleParallel();
        }
    }
}
//
```

```c#
//定义ForEach函数
//ForEach可以声明参数

//典型的lambda
Entities.ForEach(
    (Entity entity, int entityInQueryIndex, ref Translation translation, in Movement move)=>{}
)
// 默认情况下，最多可以将 8 个参数传递给lambda
// 如果需要更多参数，可以 自定义委托
// 使用标准委托时，需要按顺序对参数分组
// 1 值传递 2 ref 3 in
//所有组件都应该用 ref in参数修饰，否则，struct就不是引用


// 自定义 delegates
// 三个特殊 命名参数 
//    entity 当前实体的Entity实例，参数名随意，类型是Entity就行
//    entityInQueryIndex 该实体在查询选择的所有实体列表的索引
//          使用 NativeArray可以用来索引
//          并非的EntityCommandBuffer可以sortKey后使用索引
//    nativeThreadIndex 执行lambda函数 当前迭代的线程的唯一索引。
//          使用Run 执行lambda函数时， nativeThreadIndex始终是0
//          不要native
// 不可以使用 ref  in
struct class BringYourOwnDelegate {
    // Declare the delegate that takes 12 parameters. T0 is used for the Entity argument
    [Unity.Entities.CodeGenerateJobForEach.EntitiesForEachCompatible]
    public delegate void CustomForEachDelegate<T0, T1, T2, T3, T4, T5, T6, T7, T8, T9, T10, T11>
    (T0 t0, in T1 t1,in T2 t2,in T3 t3,in T4 t4,in T5 t5,
    in T6 t6,in T7 t7,in T8 t8,in T9 t9,in T10 t10,in T11 t11);

    // Declare the function overload
    public static TDescription ForEach<TDescription, T0, T1, T2, T3, T4, T5, T6, T7, T8, T9, T10, T11>
    (this TDescription description, CustomForEachDelegate<T0, T1, T2, T3, T4, T5, T6, T7, T8, T9, T10, T11> codeToRun)
        where TDescription : struct, Unity.Entities.CodeGeneratedJobForEach.ISupportForEachWithUniversalDelegate=>
        LambdaForEachDescriptionConstructionMethods.ThrowCodeGenException<TDescription>();
}

// A system that uses the custom delegate and overload
public class MayParamsSystem : SystemBase {
    protected override void OnUpdate() {
        Entities.ForEach(
            (
                Entity entity0, 
                in Data1 d1,
                in Data2 d2,
                in Data3 d3,
                in Data4 d4,
                in Data5 d5,
                in Data6 d6,
                in Data7 d7,
                in Data8 d8,
                in Data9 d9,
                in Data10 d10,
                in Data11 d11,
            )
            .Run();
        )
    }
}
//!! 为 ForEach lambda函数选择了默认的 8 个参数限制，
//因为声明太多的委托和重载会对IDE性能产生负面影响
// ref in value 和 参数数量的每种组合都需要唯一的委托类型和ForEach重载
```

```c#
//组件参数
// 要访问与实体关联的组件，必须将该组件的参数传递给lambda函数
// 编译器会将传递给函数的所有组件作为必须组件添加到实体查询中

// 要更新组件的值，必须用ref
// 提高效率 只读的要用 in
Entities.ForEach(
    (ref Destination outputData, in Source inputData)=>
    {
        outputData.Value = inputData.Value;
    }
)
.ScheduleParallel();
//!! 不能将块组件传递给 ForEach lambda函数

// 动态缓冲区 要用 DynamicBuffer<T>
public class BufferSum : SystemBase {
    private EntityQuery query;

    //Schedules the two jobs with a dependency between them
    protected override void OnUpdate() {
        // The query variable can be accessed here because we are
        // usign WithStoreEntityQueryInField(query) in the entities.ForEach below
        int entitiesInQuery = query.CalculateEntityCount();

        // Create a native array to hold the intermediate sums
        // (one element per entity)
        NativeArray<int> intermediateSums = new NativeArray<int>(entitiesInQuery, Allocator.TempJob);

        //Schedule the first job to add all the buffer elements
        Entities
            .ForEach
            (
                (int entityInQueryIndex, in DynamicBuffer<IntBufferData> buffer)=>
                {
                    for (int i = 0; i < buffer.Length; i++)
                    {
                        intermediateSums[entityInQueryIndex] += buffer[i].Value;
                    }
                }
            )
            .WithStoreEntityQueryInField(ref query)
            .WithName("IntermediateSums")
            .ScheduleParallel();// Execute in parallel for each chunk of entities

        // Shcedule the second job, which depends on the first
        Job
            .WithCode(
                ()=>
                {
                    int result = 0;
                    for (int i =0; i < intermediateSums.Length; i++)
                    {
                        result += intermediateSums[i];
                    }
                    //Not burst compatible:
                    Debug.Log("Final Sum is " + result);
                }
            )
            .WithDisposeOnCompletion(intermediateSums)
            .WithoutBurst()
            .WithName("FinalSum")
            .Shcedule();// Execute on a single, background thread
    }
}
```

```
捕获变量
捕获 Entity.ForEach lambda函数的局部变量
使用 job 执行函数，Schedule而不是Run时，捕获的变量有限制
--只能捕获 Native containers 本地容器 和 blittable types 可以直接复制的类型
--job只能用 Native containers捕获的变量 单个值也要用Native数组传递

Native如果是只读的 用 WithReadOnly(variable)传递
可以指定的属性包括 NativeDisableParallelForRestriction等
c# 中作为函数提供，因为变量不支持attibutes

在ForEach后，Native容器 需要卸载
WithDisposeOnCompletion(variable)

Run可以写入Native以外的变量，但是尽可能还是用blittable类型，用Burst编译
```

```
支持的功能
                             RUN      Schedule      ScheduleParallel
捕获local值类型                x          x                 x
捕获local引用类型               x非burst
写入捕获的变量                  x
使用System的字段               x非burst
引用类型的方法                  x非burst
共享组件                       x非burst
托管组件                       x非burst
结构变化                       x非burst和WithStructuralChanges
SystemBase.GetComponent       x          x              x
SystemBase.SetComponent       x          x
GetComponentDataFromEntity    x          x              x仅ReadOnly
HasComponent                  x          x              x
WithDisposeOnComponent        x          x              x

ForEach使用专门的中间语言IL处理翻译，有些功能不支持
Dynamic code in .With invocations
ref 的 SharedComponent参数
嵌套 Entities.ForEach 的lambda函数
Entities.ForEach 被标记[ExecuteAlways] 正在修复中
使用存储在变量，字段，方法中的delegate进行调用
具有lambda的SetComponet
具有可写lambda参数的GetComponent
Lambdas中的通用参数
在具有通用参数的系统中
```

```
Dependency
依赖关系
默认情况下，Entities.ForEach和 [Job.WithCode]创建的
每个作业按它们在OnUpdate函数中出现的顺序添加Dependency作业句柄中

还可以通过将[JobHandle]传递给函数来手动管理作业依赖关系，然后返回结果依赖关系
```

##### 使用 Job.WithCode

```c#
// SystemBase 类 提供 Job.WithCode
// 可以在主线程运行，仍然可以用Burst

// 示例： 用随机数填充本机数组，用另一个job将数字相加
public class RandomSumJob : SystemBase {
    private uint seed = 1;
    protected override void OnUpdate() {
        Random randomGen = new Random(seed++);
        NativeArray<float> randomNumbers = new NativeArray<float>(500, Allocator.TempJob);

        Job.WithCode(()=>
            {
                for (int i = 0; i < randomNumbers.Length; i++)
                {
                    randomNumbers[i] = randomGen.NextFloat();
                }
            }
        ).Schedule();

        // To get data out of a job, you must use a NativeArray
        // even if there is only one value
        NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);

        Job.WithCode(()=>
            {
                for (int i = 0; i < randomNumbers.Length; i++)
                {
                    result[0] += randomNumbers[i];
                }
            }
        ).Schedule();

        // This completes the scheduled jobs to get the result immediately, but for
        // better efficiency you should schedule jobs early in the frame with one
        // system and get the results late in the frame with a different system.
        this.CompleteDependency();
        UnityEngine.Debug.Log("The sum of " + randomNumbers.Length + " numbers is " + result[0]);

        randomNumbers.Dispose();
        result.Dispose();
    }
}
// !!! 要运行并行job，需要实现 IJobFor，可以使用系统OnUpdate函数中的ScheduleParallel进行 Schedule调度
```

```
变量
Variables
不能将参数传递给Job.WithCode lambda函数或返回值
可以在OnUpdate中捕获局部变量
使用C# Jobs System，使用Schedule
-- 捕获的变量必须是 NativeArray或者其他Native容器或blittable类型
-- 要返回数据，将返回值写进NativeArray，使用 Run

WithReadOnly用来指定 不更新容器
WithDisposeOnCompletion在作业后，自动处理容器
这一点 Job.WithCode Entities.ForEach 都是一样的
```

```
函数
Schedule 将函数作为单个非并行作业执行，调度作业在后台线程上运行代码，因此可以很好的利用cpu
Run 立即在主线程执行函数。大多数情况下，可以用Burst，即使在主线程上，也很快

调用Run 自动完成 Job.WithCode构造的所有依赖。如果没有将JobHandle对象显示传递给系统
假定当前的 Dependency属性表示函数的依赖关系。 如果函数没有依赖关系，传入新的JobHandle
```

```
依赖关系
默认按照OnUpdate的顺序添加到 Dependency作业句柄中。
还可以通过将JobHandle传递给函数来手动管理作业依赖，然后将结果依赖返回
```


##### 使用 Entity Batch jobs

```
在System中实现 IJobEntiyBatch 或者 IJobEntityBatchWithIndex
在OnUpdate中，调度IJobEntityBatch作业时，系统会使用传递给调度函数的EntityQuery来表示Chunk
job为这些块中的每一批实体调用一次函数，默认情况，批处理大小是完整的Chunk
但是可以在计划作业时，将批处理大小设置为Chunk的一部分
无论批次大小如何，给定批次中的实体始终存储在同一个Chunk中
可以逐个entity遍历每个批次内的数据

IJobEntityBatchWithIndex，提供批次内所有实体的Index
IJobEntityBatch效率更好，因为不需要提供Index

要实施批处理作业：
1 使用EntityQuery查询数据，标识要处理的实体
2 使用IJobEntityBatch或IJobEntityBatchWithIndex定义job结构
3 声明访问的数据。在job结构上，包括ComponentTypeHandle对象的字段，
    这些字段标识作业必须直接访问的组件类型。指定 ref in
    还可以表示要查找的数据，这些数据用于查找不属于查询的实体，用于非实体数据的字段
4 编写作业结构的Execute函数，以转换数据。获取作业读取或写入的组件的NativeArray
    遍历当前批处理，执行所需的job
5 在System的OnUpdate中调度job，将标识 要处理的实体 的EntityQuery传递给调度函数

注意：
与Entities.ForEach相比，IJobEntityBatch或者IJobEntityBatchWithindex进行迭代更加复杂
    需要更多代码设置，并且仅仅应该在必要时，或者更有效率时使用
IJobEntityBatch取代IJobChunk 主要区别在于，可以安排IJobEntityBatch遍历比完整的块更小的实体批次
如果每个批次中的实体都需要作业范围的索引，可以使用IJobEntityBatchWithIndex
```

```
使用EntityQuery查询数据
一个EntityQuery定义了一组部件类型的一个EntityArchetype必须包含System处理其相关联的组件和实体
原型可以具有其他组件，但是必须至少具有Query定义的组件。还可以排除包含而特定类型组件的原型。

注意：
不要再EntityQuery中包含完全可选的组件。要处理可选组件，使用IJobEntityBatch.Execute的
ArchetypeChunk.Has方法 来确定当前的ArchetypeChunk是否具有可选组件
同一批次中的所有实体都具有相同的组件，每个批次 只需要检查一次，而不是每个实体一次
```

```c#
// 定义 Job Struct
// job 要有 Execute函数 和声明Execute函数使用的数据的字段
public struct UpdateTranslationFromVelocityJob : IJobEntityBatch {
    public ComponentTypeHandle<VelocityVector> velocityTypeHandle;
    public ComponentTypeHandle<Translation> translationTypeHandle;
    public float DeltaTime;

    [BurstCompile]
    public void Execute(ArchetypeChunk batchInChunk, int batchIndex)
    {
        NativeArray<VelocityVector> velocityVectors = 
            batchInChunk.GetNativeArray(velocityTypeHandle);
        NativeArray<Translation> translations = 
            batchInChunk.GetNativeArray(translationTypeHandle);

        for (int i = 0; i < batchInChunk.Count; i++)
        {
            float3 translation = translations[i].Value;
            float3 velocity = velocityVectors[i].Value;
            float3 newTranslation = translation + velocity * DeltaTime;

            translations[i] = new Translation() { Value = newTranslation };
        }
    }
}
```

```
IJobEntityBatch IJobEntityBatchWithIndex
唯一区别：
indexOfFirstEntityInQuery传递这个参数，EntityQuery查询的所有列表中当前批次中第一个实体的索引

每个实体需要单独的索引时，用IJobEntityBatchWithIndex
例如：
为每个实体计算唯一的结果，则可以使用此索引将每个结果写入本机数组的其他元素。

如果不需要indexOfFirstEntityInQuery，使用IJobEntityBatch，避免计算索引值的开销

注意：
将命令添加到[EntityCommandBuffer.ParallelWriter]时，可以将该 batchIndex 参数用作sortKey命令缓冲区的参数
无需仅使用IJobEntityBatchWithIndex来获取每个实体的唯一排序键
batchIndex两种作业类型均可用的参数可用于此目的
```

```
声明 job 访问的数据
job struct中的字段声明可以用于Execute函数的数据。
1 ComponentTypeHandle字段 组件句柄字段 Execute函数可以访问存储在当前块中的实体组件和缓冲区
2 ComponentDataFromEntity BufferFromEntity 来自实体的数据 Execute函数可以查找任何实体的数据，无论存在何处
    这种类型的随机访问时访问数据最 低效 的方法，应该仅在必要时使用
3 其他字段 可以根据需要为结构声明其他字段 可以在每次计划job作业时设置此类字段的值。
4 输出字段 除了更新作业中的 可写实体组件 和 缓冲区 ，还可以写入为作业结构声明的Native容器
```

```c#
// 访问实体组件和缓冲区数据
// 首先 必须在 job struct上 定义 ComponentTypeHandle
public ComponentTypeHandle<Translation> translationTypeHandle;
// 接下来 使用Execute方法内的handle 访问包含该类型组件数据的数组 NativeArray
NativeArray<Translation> translations = batchInChunk.GetNativeArray(translationTypeHandle);
// 最后，在安排job时，在 OnUpdate中，可以使用 ComponentSystemBase.GetComponentTypeHandle
// 将值分配给类型句柄字段
// this is SystemBase subclass
updateFormVelocityJob.translationTypeHandle
    = this.GetComponentTypeHandle<Translation>(false);
// 每次计划作业时，始终设置作业的句柄字段，不要缓存

// 索引相同
// 批处理中每个组件数据数组都经过对齐，以使给定索引对应于所有数组中的同一个实体

// 可以使用 ComponentTypeHandle遍历来访问不包含在 EntityQuery中的组件
// 尝试访问之前，必须检查 batch 包含这个组件 Has功能

// ComponentTypeHandle字段是 读写 job 数据时，防止竞争条件ECS工作安全系统的一部分
// 始终设置 GetComponentTypeHandle 函数的 isReadOnly 参数， 准确反映访问权限 就是后面的 true false
```

```
查找其他实体的数据

通过EntityQuery和IJobEntityBatch作业 或者 Entities.ForEach访问数据几乎总时最有效的方法
但是很多情况下需要查找数据，例如， 当一个实体依赖于另一个实体时
执行查找，需要通过job struct将不同类型的句柄传递给job

ComponentDataFromEntity-访问具有该组件类型的任何实体的组件
BufferFromEntity-访问具有该缓冲区类型的任何实体类型的缓冲区

除了效率较低的问题，还有可能创建竞争条件
```

```
访问其他数据

例如：要更新移动的对象，则很可能需要传递自上次更新以来经过的时间。
为此，可以定义一个名为DeltaTime的字段，在OnUpdate中设置值， 在job的Execute中使用该值
在每个框架上，都需要DeltaTime 在新框架job安排作业之前，需要设置新的值
```

```c#
// Execute

// IJobEntityBatch.Execute方法
void Execute(ArchetypeChunk batchInChunk, int batchIndex)

// IJobEntityBatchWithIndex.Execute
void Execute(ArchetypeChunk batchInChunk, int batchIndex, int indexOfFirstEntityInQuery)

// batchInChunk
// 提供ArchetypeChunk的实例，该实例包含此作业的迭代的实体和组件
// 因为块只能包含一个原型，所以块中的所有实体都具有相同的组件集
// 默认情况，该对象的所有实体都在一个块中
// 如果ShceduleParallel调度作业，则可以指定批处理仅包含块中实体数量的一小部分

// 使用 batchInChunk 参数获取访问组件数据所需的NativeArray实例
// 还必须声明一个具有相应组件类型handle的字段，并在计划作业时设置该字段


// batchIndex
// 为当前作业创建的所有批次的列表中，当前批次的索引
// 作业中的批次不一定按索引顺序处理

// 有一个NativeArray，每个批次要向里面写入一个元素，要在其中写入Execute中
// 计算得到的值  这时，可用batchIndex作为容器中的数组索引


// indexOfFirstEntityInQuery
// IJobEntityBatchWithIndex才有这个参数
// 如果将 查询选择的实体 描述为单个列表
// 这个遍历僵尸当前批次中第一个实体在该列表的索引
// 作业中的批次不一定按照索引顺序处理
```

```c#
//可选组件
// 如果有Any过滤器，或者查询中根本没有出现完全可选的组件，则可以使用ArchetypeChunk.Has

// If entity has Rotation and LocalToWorld components,
// slerp to align to the velocity vector
if (batchInChunk.Has<Rotation>(rotationTypeHandle) && 
    batchInChunk.Has<LocalToWorld>(l2wTypeHandle))
{
    NativeArray<Rotation> rotations = batchInChunk.GetNativeArray(rotationTypeHandle);
    nativeArray<LocalToWorld> transforms = batchInchunk.GetNativeArray(l2wTypeHandle);

    // By putting the loop inside the check for the
    // optional components, we can check once per batch
    // rather than once per entity.
    for (int i = 0; i < batchInChunk.Count; i++)
    {
        float3 direction = math.normalize(velocityVectors[i].Value);
        float3 up = transforms[i].Up;
        quaternion rotation = rotations[i].Value;

        quaternion look = quaternion.LookRotation(direction, up);
        quaternion newRotation = math.slerp(rotation, look, DeltaTime);

        rotations[i] = new Rotation() { Value = newRotation };
    }
}
```

```c#
// Schedule job
public class UpdateTranslationFromVelocitySystem : SystemBase {
    EntityQuery query;

    protected override void OnCreate() {
        // Set up the query
        var description = new EntityQueryDesc()
        {
            All = new ComponentType[]
                {
                    ComponentType.ReadWrite<Translation>(),
                    ComponentType.ReadOnly<VelocityVector>()
                }
        };
        query = this.GetEntityQuery(description);
    }

    protected override void OnUpdate() {
        // Instantiate the job struct
        var updateFromVelocityJob = new UpdateTranslationFromVelocityJob();

        // Set the job component type handles
        // "this" is your SystemBase subclass
        updateFromVelocityJob.translationTypeHandle
            = this.GetComponentTypeHandle<Translation>(false);
        updateFromVelocityJob.velocityTypeHandle
            = this.GetComponentTypeHandle<VelocityVector>(true);

        // Set other data need in job, such as time
        updateFromVelocityJob.DeltaTime = World.Time.DeltaTime;

        // Schedule the job
        this.Dependency = updateFromVelocityJob.ScheduleParallel(query, 1, this.Dependency);
    }
}
// 调用 GetComponentTypeHandle时， 确保 isReadOnly 将读取但不写入的参数设置为 true
// 这些访问模式设置 必须和 struct定义和 EntityQuery中的 等效项 匹配

//不要将 GetComponentTypeHandle 的返回值缓存，必须每次调用时获取
```

```
Schedule 选项

Run 在当前 主线程 上立即执行 job  还可以完成当前job依赖的所有计划作业 批处理大小始终为1 整个块
Schedule 计划作业在当前作业依赖的任何计划作业之后 在工作线程上运行
        对于由实体查询选择的每个块，将对作业执行函数调用一次 块按顺序处理 批次大小始终为1
ScheduleParallel 和Schedule相似，不同之处在于可以指定批处理大小，
        并且并行处理批处理（假定工作线程可用） 而不是顺序处理

ScheduleParallel:
    设置批次大小
    ScheduleParallel方法来调度作业并将batchesPerChunk参数设置为正整数
    使用1 将批处理大小设置为完整块

    用于计划作业的查询所选择的每个块均分为所指定的批次数batchesPerChunk
    同一个块中的每个批次都包含大约相同数量的实体
    但是，来自不同块的批次可能包含数量非常不同的实体
    最大的批处理大小为1，这意味着每个块中的所有实体都在一次调用Execute函数的过程中一起吹了
    来自不同块的实体永远不能包含在同一批中

    注意：通常最有效的方法是 batchesPerChunk=1 但是并非总时如此
        例如：实体数量少，函数执行的算法昂贵
        则可用通过使用少量实体，从并行中得到好处
```

```c#
// 跳过具有不变实体的块

// 如果仅在组件值更改后才需要更新实体
// 则可用将该组件类型添加到EntityQuery的更改筛选器中
// 该筛选器选择作业的实体和块

// 例如：如果系统读取两个组件，并且仅在前两个组件中的一个已经更改时，才需要更改第三个组件
// 则可用用以下方法
EntityQuery query;
protected override void OnCreate() {
    query = GetEntityQuery(
        new ComponentType[]
        {
            ComponentType.ReadOnly<InputA>(),
            ComponentType.ReadOnly<InputB>(),
            ComponentType.ReadWrite<Output>(),
        }
    );

    query.SetChangedVersionFilter(
        new ComponentType[]
        {
            typeof(InputA),
            typeof(InputB)
        }
    );
}
// 该EntityQuery过滤器最多可支持两个组成部分
// 如果向进行更改检查，或者不用EntityQuery
// 则可用手动进行检查
// 要进行此检查，使用ArchetypeChunk.DidChange函数将组件的块的更改版本
//      与System的LastSystemVersion进行比较

// 必须使用struct将LastSystemVersion从System传递到job
struct UpdateOnChangeJob : IJobEntityBatch {
    public ComponentTypeHandle<InputA> InputATypeHandle;
    public ComponentTypeHandle<InputB> InputBTypeHandle;
    [ReadOnly] public ComponentTypeHandle<Output> OutputTypeHandle;
    public uint LastSystemVersion;

    [BurstCompile]
    public void Execute(ArchetypeChunk batchChunk, int batchIndex) {
        var inputAChanged = batchInChunk.DidChange(InputATypeHandle, LastSystemVersion);
        var inputBChanged = batchInChunk.DidChange(InputBTypeHandle, LastSystemVersion);

        // If neither component changed, skip the current batch
        if (!(inputAChanged || inputBChanged))
            return;

        var inputAs = batchInChunk.GetNativeArray(InputATypeHandle);
        var inputBs = batchInChunk.GetNativeArray(InputBTypeHandle);
        var outputs = batchInChunk.GetNativeArray(OutputTypeHandle);

        for (var i = 0; i < outputs.Length; i++)
        {
            outputs[i] = new Output{ Value = inputAs[i].Value + inputBs[i].Value };
        }
    }
}
// 和所有job struct一样，在计划作业之前，必须分配值
public class UpdateDataOnChangeSystem : SystemBase {
    EntityQuery query;
    protected override void OnCreate() {
        query = GetEntityQuery(
            new ComponentType[]
            {
                ComponentType.ReadOnly<InputA>(),
                ComponentType.ReadOnly<InputB>(),
                ComponentType.ReadWrite<Output>(),
            }
        );
    }
    protected override void OnUpdate() {
        var job = new UpdateOnChangeJob();

        job.LastSystemVersion = this.LastSystemVersion;

        job.InputATypeHandle = GetComponentTypeHandle<InputA>(true);
        job.InputBTypeHandle = GetComponentTypeHandle<InputB>(true);
        job.InputBTypeHandle = GetComponentTypeHandle<Output>(false);

        this.Dependency = job.ScheduleParallel(query, 1, this.Dependency);
    }
}
//！！！ 为了提高效率 更改版本适用于整个块 而不是单个实体
// 如果另一个具有写入 该类型组件功能的 作业 访问了一个块
// ECS将增加该组件的更改成本  并且DicChange函数将返回true
// 即使声明对组件进行写访问的作业实际并未更改组件值，ECS也会增加更改版本
// 这是读取组件数据而不更新它时，始终应该设置只读的原因
```

##### 使用 IJobChunk 不再使用，将来会被弃用？

IJobChunk已经被IJobEntityBatch取代，应该使用新的代码  

```
1 创建 EntityQuery 标识要处理的实体
2 定义job struct 包括 ArchetypeChunkComponentType 对象的字段，
    这些对象标识作业必须直接访问的组件的类型。指定 读取 写入 权限
3 实例化作业结构 在 System 的OnUpdate中安排job
4 在Execute 函数中 获取NativeArray 作业读取或写入的组件
    在当前块上迭代，执行代码
```

```c#
public class RotationSpeedSystem : SystemBase {
    private EntityQuery m_Query;
    protected override void OnCreate() {
        m_Query = GetEntityQuery(ComponentType.ReadOnly<Rotation>(),
            ComponentType.ReadOnly<RotationSpeed>());
    }
}

// 更复杂的情况 使用 EntityQueryDesc
// All 必须存在
// Any 至少一种
// None 不能存在
proetcte override void OnCreate() {
    var queryDescription = new EntityQueryDesc()
    {
        None = new ComponentType[]
        {
            typeof(Static)
        },
        All = new ComponentType[]
        {
            ComponentType.ReadWrite<Rotation>(),
            ComponentType.ReadOnly<RotationSpeed>()
        }
    };
    m_Query = GetEntityQuery(queryDescription);
}

// 组合查询 逻辑或
// 包含 A B AB
protected override void OnCrate() {
    var queryDescription0 = new EntityQueryDesc
    {
        All = new ComponentType[] { typeof(Rotation) }
    };
    var queryDescription1 = new EntityQueryDesc
    {
        All = new ComponentType[] { typeof(RotationSpeed)}
    };
    m_Query = GetEntityQuery(new EntityQueryDesc[] { queryDescription0, queryDescription1 })
}
// 可选组件要通过chunk.Has在Execute中判断

// EntityQuery 应该在OnCreate中查询，存储在变量中
```

```c#
// IJobChunk结构
// 要访问Execute方法的块内组件数组，必须 ArchetypeChunkComponentType<T>
// 必须检查可选组件
[BurstCompile]
struct RotationSpeedJob : IJobChunk {
    public float DeltaTime;
    public ComponentTypeHandle<Rotation> RotationTypeHandle;
    [ReadOnly] public ComponentTypeHandle<RotationSpeed> RotationSpeedTypeHandle;

    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
        var chunkRotations = chunk.GetNativeArray(RotationTypeHandle);
        var chunkRotationSpeed = chunk.GetNativeArray(RotationSpeedTypeHandle);
        for (var i = 0; i < chunk.Count; i++)
        {
            var rotation = chunkRotations[i];
            var rotationSpeed = chunkRotationSpeeds[i];

            // Rotate something about its up vector at the speed given by RotationSpeed.
            chunkRotations[i] = new Rotation
            {
                Value = math.mul(math.normalize(rotation.Value), 
                    quaternion.AxisAngle(math.up(), rotationSpeed.RadiansPerSecond * DeltaTime))
            };

            if (chunk.Has<OptionalComp>(OptionalCompType))
            {

            }
        }
    }
}
// chunkIndex参数作为sortKey参数传递给命令缓冲区函数
```

```c#
// 跳过具有不变实体的块
private EntityQuery m_Query;
protected override void OnCreate() {
    m_Query = GetEntityQuery(
        ComponentType.ReadWrite<Output>(),
        ComponentType.ReadWrite<InputA>(),
        ComponentType.ReadWrite<InputB>()
    );
    m_Query.SetChangedVersionFilter(
        new ComponentType[]
        {
            ComponentType.ReadWrite<InputA>(),
            ComponentType.ReadWrite<InputB>()
        }
    );
}
// 同样的 最多支持两个组件，可用手动检查 ArchetypeChunk.DidChange()
// LastSystemVersion
[BurstCompile]
struct Updatejob : IJobChunk {
    public ComponentTypeHandle<InputA> InputATypeHandle;
    public ComponentTypeHandle<InputB> InputBTypeHandle;
    [ReadOnly] public ComponentTypeHandle<Output> OutputTypeHandle;
    public unit LastSystemVersion;

    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex) {
        var inputAChanged = chunk.DidChange(InputATypeHandle, LastSystemVersion);
        var inputBChanged = chunk.DidChange(InputBTypeHandle, LastSystemVersion);

        // If neither component changed, skip the current chunk
        if (!(inputAChanged || inputBChanged))
            return;

        var inputAs = chunk.GetNativeArray(InputATypeHandle);
        var inputBs = chunk.GetNativeArray(InputBTypeHandle);
        var outputs = chunk.GetNativeArray(OutputTypeHandle);

        for (var i = 0; i < outputs.Length; i++) {
            outputs[i] = new Output{ Value = inputAs[i].Value + inputBs[i].Value };
        }
    }
}

protected override void OnUpdate() {
    var job = new UpdateJob();

    job.LastSystemVersion = this.LastSystemVersion;

    job.InputATypeHandle = GetComponentTypeHandle(InputA)(true);
    job.InputBTypeHandle = GetComponentTypeHandle(InputB)(true);
    job.OutputTypeHandle = GetComponentTypeHandle(Output)(false);

    this.Dependency = job.ScheduleParallel(m_Query, this.Dependency);
}
```

```c#
//实例化并安排job
protected override void OnUpdate()
{
    var job = new RotationSpeedJob()
    {
        RotationTypeHandle = GetComponentTypeHandle<Rotation>(false);
        RotationSpeedTypeHandle = GetComponentTypeHandle<RotationSpeed>(true);
        DeltaTime = Time.DeltaTime
    };
    this.Dependency = job.ScheduleParallel(m_Query, this.Dependency);
}
```

##### 手动迭代

```c#
// 可用在NativeArray中显式请求所有块，用例如 IJobParalleFor进行迭代
// 如果需要用  不适用于EntityQuery所有块 进行迭代的模型来遍历
// 可用下面方法：
public class RotationSpeedSystem : SystemBase
{
    [BurstCompile]
    struct RotationSpeedJob : IJobParallelFor
    {
        [DeallocateOnJobCompletion] public NativeArray<ArchetypeChunk> Chunks;
        public ArchetypeChunkComponetType<RotationQuaternion> RotationType;
        [ReadOnly] public ArchetypeChunkComponentType<RotationSpeed> RotationSpeedType;
        public float DeltaTime;

        public void Execute(int chunkIndex)
        {
            var chunk = Chunks[chunkIndex];
            var chunkRotation = chunk.GetNativeArray(RotationType);
            var chunkSpeed = chunk.GetNativeArray(RotationSpeedType);
            var instanceCount = chunk.Count;

            for (int i = 0; i < instanceCount; i++)
            {
                var chunk = Chunks[chunkIndex];
                var chunkRotation = chunk.GetNativeArray(RotationType);
                var chunkSpeed = chunk.GetNativeArray(RotationSpeedType);
                var instanceCount = chunk.Count;

                for (int i = 0; i < instanceCount; i++)
                {
                    var rotation = chunkRotation[i];
                    var speed = chunkSpeed[i];
                    rotation.Value = math.mul(math.normalize(rotation.Value), 
                            quaternion.AxisAngle(math.up(), speed.RadiansPerSecond * DeltaTime));
                    chunkRotation[i] = rotation;
                }
            }
        }
    }

    EntityQuery m_Query;

    protected override void OnCreate()
    {
        var queryDesc = new EntityQueryDesc
        {
            All = new ComponentType[]{ typeof(RotationQuaternion), ComponentType.ReadOnly<RotationSpeed>() }
        };

        m_Query = GetEntityQuery(queryDesc);
    }

    protected override void OnUpdate()
    {
        var rotationType = GetArchetypeChunkComponentType<RotationQuaternion>();
        var rotationSpeedType = GetArchetypeChunkComponentType<RotationSpeed>(true);
        var chunks = m_Query.CreateArchetypeChunkArray(Allocator.TempJob);

        var rotationsSpeedJob = new RotationSpeedJob
        {
            Chunks = chunks,
            RotationType = rotationType,
            RotationSpeedType = rotationSpeedType,
            DeltaTime = Time.deltaTime;
        };
        this.Dependency = rotationSpeedJob.Schedule(chunks.Length, 32, this.Dependency);
    }
}
```

```c#
// 用EntityManager手动遍历，只能用在测试 调试中 或者时完全受控制的实体集合的孤立世界中
// 才能用
var entityManager = World.Active.EntityManager;
var allEntities = entityManager.GetAllEntities();
foreach (var entity in allEntities)
{
    //...
}
allEntities.Dispose();

// 所有块
var allChunks = entityManager.GetAllChunks();
foreach (var chunk in allChunks)
{
    //...
}
allChunks.Dispose();
```

##### 系统更新顺序

```
组件系统组
ComponentSystemGroup类表示应该按照特定顺序一起更新的相关组件系统的列表
ComponentSystemBase是基类，因此它在所有重要方面都像组件系统一样工作
可以对其他系统进行排序，具有OnUpdate方法
最关键的：可以将组件系统组嵌套在其中其他的组件系统组，形成层次结构

默认情况下，Update调用ComponentSystemGroup时，系统上每个成员都调用Update
如果任何系统成员本身时系统组，它们会递归更新自己的成员，树的深度遍历优先顺序

系统order属性
[UpdateInGroup]
指定此系统应该是其成员的ComponentSystemGroup 默认SimulationSystemGroup
[UpdateBefore]
[UpdateAfter]
声明同一组成员，相对顺序
[DisableAutoCreation]
防止在默认世界初始化期间创建系统
显式创建和更新系统
可以将带有此标签的系统添加到 ComponentSystemGroup 的更新列表中
然后它将像该列表中的其他系统一样自动进行更新。

默认系统组 此列表的特定内容可能会变
InitializationSystemGroup  updated at the end of the Initialization phase of the player loop
    BeginInitializationEntityCommandBufferSystem
    CopyInitialTransformFromGameObjectSystem
    SubSceneLiveLinkSystem
    SubSceneStreamingSystem
    EndInitializationEntityCommandBufferSystem
SimulationSystemGroup  updated at the end of the Update phase of the player loop
    BeginSimulationEntityCommandBufferSystem
    TransformSystemGroup
        EndFrameParentSystem
        CopyTransformFromGameObjectSystem
        EndFrameTRSToLocalToWorldSystem
        EndFrameTRSToLocalToParentSystem
        EndFrameLocalToParentSystem
        CopyTransformToGameObjectSystem
    LateSimulationSystemGroup
    EndSimulationEntityCommandBufferSystem
PresentationSystemGroup  updated at the end of the PreLateUpdate phase of the player loop
    BeginPresentationEntityCommandBufferSystem
    CreateMissingRenderBoundsFromMeshRenderer
    RenderingSystemBootstrap
    RenderBoundsUpdateSystem
    RenderMeshSystem
    LODGroupSystemV1
    LodRequirementsUpdateSystem
    EndPresentationEntityCommandBufferSystem
```

```c#
多个世界

// 可以创建多个世界，同一个组件系统类可以在多个世界中初始化，
// 每个实例可以从更新顺序的不同点以不同速率更新

// 当前无法在给定的世界中手动更新每个系统
// 但是可以控制在哪个世界中创建那些系统以及应该将其添加到那些现有的系统组中
// 因此，WorldB可以实例化SystemX和SystemY，将SystemX添加到WorldA的
// SimulationSystemGroup，并将SystemY添加到默认的WorldA PresentationSystemGroup
// 这些系统可以向往常一样进行排序，并将相应的组一起更新

// 自定义 初始化 Systems ，在多个 世界 中注册系统组
public interface ICustomBootstrap
{
    // Returns the systems which should be handled by the default bootstrap process.
    // If null is returned the default world will not be created at all.
    // Empty list creates default world and entrypoints
    List<Type> Initialize(List<Type> systems);
}
//实现这个接口，组件系统类型的完整列表将 Initialize在默认世界初始化之前
// 传递给classes方法
// 定制的引导程序可以遍历这个列表，并在所需要的任何 World中创建系统
// 可以从initialize 方法返回系统列表 这些系统将作为常规默认世界初始化的一部分创建

//例如，自定义 MyCustomBoostrap.Initialize()的典型过程
// 1 创建任何其他Worlds及其顶级ComponentSystemGroups
// 2 对于系统类型列表中的每个类型
//    1 向上遍历ComponentSystemGroup层次结构以找到此系统Type的顶级组
//    2 如果它是在步骤1中创建的组之一，请在该世界中创建系统，
//      然后使用 group.AddSystemToUpdateList() 将其添加到层次结构中
//    3 如果不是，请将此类型附加到默认世界组之一
// 3 在新的顶级组上调用 group.SortSystemUpdateList()
//    1 可选 将它们添加到默认世界组之一
// 4 将未处理的系统列表返回给 DefaultWorldInitialization

// 注意！！！ ECS框架通过 反射 查找 ICustomBootstrap 实现
```

```
提示 和 最佳做法
1   使用 [UpdateInGroup] 为编写的每个系统指定系统组。
    如果未指定，默认 SimulationSystemGroup
2   使用手动选定的 ComponentSystemGroups 来更新Unity Player Loop
    将 [DisableAutoCreation] 属性添加到组件系统 或 系统组
    可防止将其创建或添加到默认系统组
    可以用 World.GetOrCreateSystem手动创建系统，
    并通过从主线程手动调用MySystem.Update进行更新
    这是在Unity Player循环中的其他位置插入系统的简便方法
3   EntityCommandBufferSystem如果可能的化，使用现有的，而不是添加新的
    一个EntityCommandBufferSystem代表同步点，在该同步点，主线程在处理任何未完成的
    EntityCommandBuffers 之前先等待工作线程完成
4   避免在ComponentSystemGroup中加入自定义逻辑
    由于从ComponentSystemGroup功能上来说本身就是一个组件系统，
    因此可能很想在其OnUpdate中添加自定义处理，执行一些工作
    但是不建议这样，因为从外部尚不清楚自定义逻辑在更新组成员前还是之后执行
    最好将系统限制为一种分组机制，并且在相对于该组，显式排序
```

##### Job 依赖

```
Unity根据 System 读取和写入的ECS组件 分析每个 System 的数据依赖性
如果在框架中较早更新的系统读取了较新系统写入的数据，或者写入了较新系统读取的数据
则第二个系统依赖于第一个系统
为了避免竞争，作业调度程序确保在运行System job之前，System依赖的所有Job都完成

系统的Dependency属性是JobHandle，代表与系统的ECS相关的依赖关系
在OnUpdate之前，Dependency属性反映了系统对先前作业的传入依赖关系
默认情况下，系统根据在系统中Schedule Job 时 读取 和 写入 的组件来更新Dependency

要覆盖默认行为，需要使用 Entities.ForEach 和 Job.WithCode的重载版本
这些重载版本将job dependency作为参数，并将更新后的dependency作为JobHandle返回
使用这些构造函数的显式版本时，ECS不会自动将作业句柄与系统Dependency属性结合在一起
必须在需要时手动组合

注意，系统的Dependency属性不会跟踪job对NativeArrays数据可能具有的依赖关系
如果在一个job中编写NativeArray并在另一个作业中读取该数组，则必须手动添加第一个作业的
JobHandle作为第二个作业中的依赖项 通常使用 JobHandle.CombineDependencies

当调用 Entities.ForEach.Run 时，作业计划程序会在开始ForEach迭代之前，
完成系统依赖的所有计划作业
如果还使用WithStructuralChanges作为构造的一部分，则作业计划程序将完成所有正在运行的计划作业。
结构上的更改也会使对组件数据的任何直接引用无效
```

##### 查找数据

```
访问和修改ECS数据最有效的方法 是使用带有EntityQuery和Job的System
这样可以以最少的 内存高速缓存未命中 来最佳利用CPU资源

实际上，数据设计的目标之一 应该是 使用最有效，最快的路径执行大部分数据转换
但是 有时需要在程序的任意位置访问任意实体的任意组件

给定一个Entity对象，可以在其 IComponentData 和 动态缓冲区 中查找数据
方法的不同取决于 Entities.ForEach 还是 [IJobChunk] 还是 主线程
```

```c#
//在系统中查找实体数据

// 使用 GetComponent<T>(Entity) 从System的Entities.ForEach
// 或Job.WithCode函数内部查找存储在任意实体组件中的数据

// 例如，如果 目标 组件的实体字段标识了要定位的实体，可以使用以下代码
// 跟踪实体向其目标旋转
public partial class TranckingSystem : SystemBase {
    protected override void OnUpdate() {
        float deltaTime = this.Time.DeltaTime;

        Entities
            .ForEach(
                (ref Rotation orientation, in LocalToWorld transform, in Target target)=>
                {
                    // Check to make sure the target Entity still exists and has
                    // the needed component
                    if (!HasComponent<LocalToWorld>(target.entity))
                        return;

                    // Look up the entity data
                    LocalToWorld targetTransform = GetComponent<LocalToWorld>(target.entity);
                    float3 targetPosition = targetTransform.Position;

                    // Calculate the rotation
                    float3 displacement = targetPosition - transform.Position;
                    float3 upReference = new float3(0, 1, 0);
                    quaternion lookRotation = quaternion.LookRotationSafe(dispacement, upReference);

                    orientation.Value = math.slerp(orientation.Value, lookRotation, deltaTime);
                }
            )
            .ScheduleParallel();
    }
}

//访问存储在 动态缓冲区 中的数据需要额外步骤
// 必须在 OnUpdate 方法中声明 BufferFromEntity类型的局部变量
// 在 lambda函数中 捕获 局部变量
public struct BufferData : IBufferElementData {
    public float Value;
}
public partial class BufferLookupSystem : SystemBase {
    protected override void OnUpdate() {
        BufferFromEntity<BufferData> buffersOfAllEntities = 
            this.GetBufferFromEntity<BufferData>(true);

        Entities
            .ForEach((ref Rotation orientation, in LocalToWorld transform, in Target target)=>{
                // Check to make sure the target Entity with this buffer type still exists
                if (!buffersOfAllEntities.HasComponent(target.entity))
                    return;

                // Get a reference to the buffer
                DynamicBuffer<BufferData> bufferOfOneEntity = 
                    buffersOfAllEntities[target.entity];
                
                // Use the data in the buffer
                float avg = 0;
                for (var i = 0; i < bufferOfOneEntity.Length; i++)
                {
                    avg += bufferOfOneEntity[i].Value;
                }
                if (bufferOfOneEntity.Length > 0)
                    avg /= bufferOfOneEntity.Length;
            })
            .ScheduleParallel();
    }
}
```

```c#
// 在IJobEntityBatch中查找实体数据

// 使用 ComponentDataFromEntity 或者 BufferFromEntity来获取组件的 类似数组的接口，用Entity对象索引

// 例子：如果 目标组件 的 实体字段标识了要定位的实体，则可以将以下字段添加到作业结构中以查找目标的世界位置
[ReadOnly]
public ComponentDataFromEntity<LocalToWorld> EntityPosition;
// 请注意，此声明使用"只读"属性。除非写入所访问的组件，否则应始终将组件数据从Entity对象声明为只读对象。

var job = new ChaserSystemJob();
job.EntityPositions = this.GetComponentDataFromEntity<LocalToWorld>(true);

// 在job的Execute内，可以使用entity对象查找组件的值
float3 targetPosition = EntityPositions[targetEntity].Position;

// 以下完整示例，显示了System，将具有包含目标的Entity对象的target字段的实体移向目标的当前位置
public class MoveTowardsEntitySystem : SystemBase {
    private EntityQuery query;

    [BurstCompile]
    private struct MoveTowardsJob : IJobEntityBatch {
        // Read-write data in the current chunk
        public ComponentTypeHandle<Translation> PositionTypeHandleAccessor;

        // Read-only data in the current chunk
        [ReadOnly]
        public ComponentTypeHandle<Target> TargetTypeHandleAccessor;

        // Read-only data stored (potentially) in other chunks
        [ReadOnly]
        public ComponentDataFromEntity<LocalToWorld> EntityPositions;

        // Non-entity data
        public float deltaTime;

        public void Execute (ArchetypeChunk batchInChunk, int batchIndex) {
            // Get arrays of the components in chunk
            NativeArray<Translation> positions
                = batchInChunk.GetNativeArray<Translation>(PositionTypeHandleAccessor);
            NativeArray<Target> targets
                = batchInChunk.GetNativeArray<Target>(TargetTypeHandleAccessor);

            for (int i = 0; i < positions.Length; i++) {
                // Get the target entity object
                Entity targetEntity = targets[i].entity;

                // Check that the target still exists
                if (!EntityPositions.HasComponent(targetEntity))
                    continue;

                // Update translation to move the chasing entity toward the target
                float3 targetPosition = EntityPositions[targetEntity].Position;
                float3 chaserPosition = positions[i].Value;

                float3 displacement = targetPosition - chaserPosition;
                positions[i] = new Translation
                {
                    Value = chaserPosition + displacement * deltaTime;
                }
            }
        }
    }

    protected override void OnCreate() {
        // Select all entities that have Translation and Target Componentx
        query = this.GetEntityQuery
        (
            typeof(Translation),
            ComponentType.ReadOnly<Target>()
        );
    }

    protected override void OnUpdate() {
        // Create the job
        var job = new MoveTowardsJob();

        // Set the chunk data accessors
        job.PositionTypeHandleAccessor = 
            this.GetComponentTypeHandle<Translation>(false);
        job.TargetTypeHandleAccessor = 
            this.GetComponentTypeHandle<Target>(true);

        // Set the component data lookup field
        job.EntityPositions = this.GetComponentDataFromEntity<LocalToWorld>(true);

        // Set non-ECS data fields
        job.deltaTime = this.Time.DeltaTime;

        // Schedule the job using Dependency property
        this.Dependency = job.ScheduleParallel(query, 1, this.Dependency);
    }
}
```

```
数据存取错误

如果 要查找的数据 和 直接在job中 读取 写入 的数据重叠，
随机访问会导致争用情况和错误

如果确定要在作业中直接读取或者写入特定实体数据 和 正在随机读取写入的特定实体数据之间没有重叠
则可以用 NativeDisableParallelForRestriction 属性标记访问器对象
```

##### EntityCommandBuffer

```
实体命令缓冲区 解决两个重要问题
1 在工作时，无法访问EntityManager
2 执行结构更改，如创建实体时，将创建一个同步点，并且必须等待所有作业完成

实体命令缓冲区 允许排队变化，无论从工作线程还是主线程，使他们能在主线程生效

实体命令缓冲区系统
在帧中明确定义的位置 调用 在ECB中排队的命令
可以从同一个 实体命令缓冲区系统 中获取多个ECB，系统将按照更新时创建他们的顺序来播放所有ECB
系统创建一个同步点，而不是每个ECB创建一个同步点

默认的World初始化提供了 三个系统组 initialization simulation presentation
在一个组内，有一个实体命令缓冲区系统，在组中任何其他系统前运行
另一个在该组中的所有其他系统后运行

最好使用现有的EntityCommandBuffer，不要自己创建，以减少同步点

如果要使用 parallel job中的ECB，必须确保首先通过调用 ToConcurrent 将其转换为并非ECB
为了确保ECB中命令的顺序不取决于jobs分配，还必须将当前查询中实体的索引传递给每个操作
```

```c#
struct LifeTime : IComponentData {
    public byte Value;
}

partial class LifetimeSystem : SystemBase {
    EndSimulationEntityCommandBufferSystem m_EndSimulationEcbSystem;
    protected override void OnCreate() {
        base.OnCreate();
        // Find the ECB system once and store it for later usage
        m_EndSimulationEcbSystem = World.GetOrCreateSystem<EndSimulationEntityCommandBufferSystem>();
    }
    protected override void OnUpdate() {
        // Acquire an ECB and convert it to a concurrent one to be able
        // to use it from a parallel job.
        var ecb = m_EndSimulationEcbSystem.CreateCommandBuffer().AsParallelWriter();
        Entities
            .ForEach((Entity entity, int entityInQueryIndex, ref Lifetime, lifetime)=>{
                // Track the lifetime of an entity and destroy it once
                // the lifetime reaches zero
                if (lifetime.Value == 0) {
                    // pass the entityInQueryIndex to the operation so
                    // the ECB can play back the commands in the right
                    // order
                    ecb.DestroyEntity(entityInQueryIndex, entity);
                }
                else
                {
                    lifetime.Value -= 1;
                }
            }).ScheduleParallel();

            // Make sure that the ECB system knows about our job
            m_EndSimulationEcbSystem.AddJobHandleForProducer(this.Dependency);
    }
}
```

#### 同步点

```
Sync Point 是程序执行中的一个点，它等待到目前为止 已经调度的所有job的完成
同步点会限制一段时间内使用 作业系统中所有可用工作线程的能力
因此应该避免同步点

结构变化
同步点是由 在组件上执行任何其他作业时 无法安全执行的操作引起的
ECS中数据的结构更改是同步点的主要原因
-- 创建实体
-- 删除实体
-- 向实体添加组件
-- 从实体中删除组件
-- 更改共享组件的值

广义上讲，任何更改实体原型或导致块中实体顺序更改的操作都是结构性更改。
结构更改只能在主线程执行

结构更改不仅需要同步点，而且还会使对任何组件数据的所有直接引用无效
包括 DynamicBuffer 的实例以及提供直接访问组件
例如： ComponentSystemBase.GetComponentDataFromEntity

避免同步点
可以使用 ECB 实体命令缓冲区 将结构过呢刚刚排入队列，而不是立即执行
存储在ECB中的命令 可以在帧中的稍后一点 调用
当 调用 ECB时，会将跨帧分布的多个同步点减少到单个

每个标准ComponentSystemGroup实例都提供一个EntityCommandBufferSystem
作为组中更新第一个和最后一个的系统
通过从这些标准ECB系统之一获取ECB对象，组内的所有结构更改都发生在帧中的同一点
这样只有一个同步点而不是多个

ECB还允许记录job中的结构更改，如果没有ECB，则只能在主线程上进行结构更改
即使在主线程上，在EBC中记录命令 然后调用 通常比使用EntityManager类本身进行一次一个的结构更改要快

如果不能将ECBS用于任务，可以尝试将系统按照执行顺序进行结构更改的所有系统组合
如果两个系统都进行结构更改，则只有顺序才能产生同步点
```

#### Write groups

```
常见的ECS模式是 系统读取一组输入组件 并将其写入另一组组件 作为输出
但是，在某些情况下，可能需要覆盖系统的输出，基于不同的输入集 使用不同的系统来更新输出组件

Write Groups为一个系统提供了一种覆盖另一个系统的机制，即使不能更改另一个系统也是如此

目标组件类型的Write Groups由ECS应用 WriteGroup属性的 所有其他组件类型组成
并且以该目标组件类型作为参数
作为系统创建者，可以使用WriteGroup，以便系统用户可以排除系统将选择和处理的实体
通过这种过滤机制，系统用户可以根据自己的逻辑为排除的实体更新组件，同时让系统在其余组件上正常运行

要使用WriteGroup，必须在系统中的查询上使用 Write Group filter option
这会从查询中排除所有具有某个组件的写入组中所有组件的实体，这些组件在查询中被标记为可写

要覆盖使用WriteGroup的系统，要将自己的组件类型标记为该系统的输出组件类型的WriteGroup的一部分
原始系统将忽略具有组件的任何实体，并且可以使用自己的系统更新这些实体的数据
```

```c#
// 示例：使用外部程序包 根据游戏中所有角色的健康状况为其着色
public struct HealthComponent : IComponentData {
    public int Value;
}

public struct ColorComponent : IComponentData {
    public float4 Value;
}

// ComputeColorFromHealthSystem 读取 HealthComponent 写入 ColorComponent
// RenderWithColorComponent 读取 ColorComponent

// 为了标识玩家何时使用道具，并且角色无敌
// 可以在角色的实体上附加一个 InvincibleTagComponent 
// 这种情况 角色改为单独不同的颜色，上面的示例无法容纳该颜色

// 可以创建自己的系统 覆盖ColorComponent的值，但 理想情况下
// ComputeColorFromHealthSystem 不会为实体计算颜色
// 应该忽略具有任何 InvincibleTagComponent的实体

// 当知道计算的值将被覆盖，可以忽略系统查询的实体
// 1 在 InvincibleTagComponent 必须标记为WriteGroup的一部分ColorComponent
[WriteGroup(typeof(ColorComponent))]
struct InvincibleTagComponent : IComponnetData {}
// 写入组ColorComponent 包括所有具有WriteGroup属性typeof(ColorComponent)作为参数的组件类型
// 2 在 ComputeColorFromHealthSystem必须明确支持写入组
// 为此，系统需要 EntityQueryOptions.FilterWriteGroup为其所有查询指定选项
protected override void OnUpdate() {
    Entities
        .WithName("ComponentColor")
        .WithEntityQueryOptions(EntityQueryOptions.FilterWriteGroup) // support write groups
        .ForEach((ref ColorComponent color, in HealthComponent health)=>{
            // compute color here
        }).ScheduleParallel();
}
// 执行此操作时：
// 1 系统检测到 WriteGroup ColorComponent
// 2 查找写入组ColorComponent 查找其中的 InvincibleTagComponent
// 3 排除具有 InvincibleTagComponent 的实体

// 示例参考 Unity.Transforms代码，为每个组件使用WriteGroup LocalToWorld
```

```c#
//创建WriteGroup
// 不要将W加入自己的WriteGroup
public struct W : IComponentData {
    public int Value;
}
[WriteGroup(typeof(W))]
public struct A : IComponentData {
    public int Value;
}
[WriteGroup(typeof(W))]
public struct B : IComponentData {
    public int Value;
}
```

```c#
//启用过滤
public class AddingSystem : Systembase {
    protected override void OnUpdate() {
        Entities
            // support write groups by setting entityQueryOptions
            .WithEntityQueryOptions(EntityQueryOptions.FilterWriteGroup)
            .ForEach((ref W w, in B b)=>{
                // perform computation here
            }).ScheduleParallel();
    }
}
// 对于EntityQueryDesc ，在创建时设置
public class AddingSystem : SystemBase {
    private EntityQuery m_Query;
    protected override void OnCreate() {
        var queryDescription = new EntityQueryDesc
        {
            All = new ComponentType[]{
                ComponentType.ReadWrite<W>,
                ComponentType.ReadOnly<B>
            },
            Options = EntityQueryOptions.FilterWriteGroup
        };
        m_Query = GetEntityQuery(queryDescription);
    }
}
// 排除任何 A， 因为 W可写，A属于W组
// 不排除任何 B， 因为B在All中明确指定
```

```c#
//覆盖另一个使用WriteGroup的系统

// 如果系统在查询中 通过WriteGroup筛选，可以使用自己的系统覆盖该系统并写出那些组件
// 要覆盖系统，要将自己的组件添加到其他系统要写入的组件的WriteGroup中
// 由于WriteGroup筛选排除了查询中没有明确要求的WriteGroup中的任何组件
// 因此其他系统忽略具有组件的任何实体

// 例如：如果要通过指定旋转角度和轴 设置实体的方向
// 可以创建一个组件和一个系统
// 将角度和轴 的值转换为 四元数 ，并写入Unity.Transforms.Rotation组件
// 为了防止Unity.Transforms 系统更新Rotation
// 无论是否拥有其他组件，都可以将组件放在Rotation组中
using System;
using Unity.Collections;
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;

[Serializable]
[WriteGroup(typeof(Rotation))]
public struct RotationAngleAxis : IComponentData {
    public float Angle;
    public float3 Axis;
}

// 这样，可以用 RotationAngleAxis 组件更新实体，不会发生争用
using Unity.Burst;
using Unity.Entities;
using Unity.Jobs;
using Unity.Collections;
using Unity.Mathematics;
using Unity.Transforms;

public class RotationAngleAxisSystem : SystemBase {
    protected override void OnUpdate() {
        Entities.ForEach((ref Rotation destination, in RotationAngleAxis source)=>{
            destination.Value
                = quaternion.AxisAngle(math.normalize(source.Axis), source.Angle);
        }).ScheduleParallel();
    }
}
```

```c#
// 拓展使用WriteGroup的另一个系统
// 拓展 而不是 覆盖
// 或者运行 将来的系统 覆盖或拓展
// 那么 可以在自己的系统上启用WriteGroup筛选
// 默认情况 两个系统都不会处理组件的任何组合，必须显示查询和处理每个组合

// 前面 W A B
// 如果 新的 C 添加到 W组
// 则新系统查询 C 可能也带有 A B
// 如果新系统也开启了筛选，那就不会带A B了
// 就必须显示查询有意义的每个组合

var query = new EntityQueryDesc
{
    All = new ComponentType[] {
        ComponentType.ReadOnly<C>(),
        ComponentType.ReadWrite<W>()
    },
    Any = new ComponentType[] {
        ComponentType.ReadOnly<A>(),
        ComponentType.ReadOnly<B>()
    },
    Options = EntityQueryOptions.FilterWriteGroup
};
```

#### 版本号

```c#
//优化策略
//版本号是32位带符号整数，一直增加，有整数溢出是C#定义的行为
bool VersionBIsMoreRecent = (VersionB - VersionA) > 0;
// 通常无法保证版本号增加多少
```

```
EntityId.Version
一个EntityId 有 索引 版本号
由于ECS 回收索引， EntityManager每次实体销毁时，版本号增加
如果EntityId 中查找版本号不匹配EntityManager,
意味着引用的实体不再存在
例如 知道EntityId 调用 ComponentDataFromEntity.Exists

World.Version
ECS 每次创建或者销毁System时，都会增加World的版本号

EntityDataManager.GlobalVersion
在每个job component system更新前增加

System.LastSystemVersion
每次EntityDataManager.GlobalVersion更新后取的值

Chunk.ChangeVersion
EntityDataManager.GlobalVersion上次访问组件数组时的值
共享组件的版本号 没有作用
WithChangeFilter在Entities.ForEach构造中使用，ECS会比较
块的ChangeVersion和System.LastSystemVersion
并且它仅处理其组件数组 在系统上一次开始运行后 可写的块
例如：如果保证自上一帧以来，一组单位的Version未变
则可以跳过检查这些单位是否更新

EntityManager.m_ComponentTypeOrderVersion[]
对于每种非共享组件类型，涉及该类型的迭代器无效时，ECS会增加版本号
任何可能修改该类型的数组的东西
例如：如果特定组件标识的静态对象和每个块的边界框
则仅当组件的类型顺序版本发生更改时，才需要更新边界框

SharedComponentDatamanager.m_SharedComponentVersion[]
存储在引用共享组件的块中的实体发生任何结构更改时，增加版本号
例如：保留每个共享组件的实体计数，则可以使用该版本号仅在相应版本号发生更改时重做每个计数
```

#### Job 拓展

```
Unity C# 作业系统可以多线程运行代码 提供调度，并行处理，多线程安全
jobs系统是Unity的核心，提供用于创建和运行job的通用接口和类 不论是否用ECS

IJob 创建一个可以在作业系统计划程序确定的任何线程或核心上运行的作业
IJobParallelFor 创建一个可以在多个线程上并行运行的作业，以处理NativeContainer的元素
IJobExtensions 提供扩展方法以运行IJobs
IJobParallelForExtensions 提供扩展方法以运行IJobParallelFor作业
JobHandle 访问预定作业的句柄 可以使用JobHandle实例来指定作业之间的依赖关系
```

#### 通用 Jobs

```c#
//普通的c#中，可以使用继承和接口使一段代码可以处理多种类型
// void foo(IBlendable a) {}

// HPC#中，不能使用托管类型，或者虚拟方法，因此 泛型是唯一选择
void foo<T>(T a) where T : struct, IBlendable {}

//作业必须用HPC#
[BurstCompile()]
public struct BlendJob<T> : IJob where T : struct, IBlendable {
    public NativeRefreence<T> blendable;

    public void Execute() {
        var val = blendable.Value;
        val.Blend();
        blendable.Value = val;
    }
}

// Burst安排通用job
[assembly: RegisterGenericJobType(typeof(MyJob<int, float>))]
// 没有注册的job会引发异常

// 自动注册具体job
var job = new MyJob<int, float>();

//间接实例化具体job，不会自动注册
void makeJob<T>() {
    new MyJob<T, float>().Schedule();
}
void foo(){
    makeJob<int>(); //不行
}

//作为返回类型或者out是可以注册的
MyJob<T, float> makeJob<T>() {
    var j = new MyJob<T, float>();
    j.Schedule();
    return j;
}
void foo() {
    makeJob<int>(); //可以
}

//多级间接注册，可以
MyJob<T, float> makeJob<T>() {
    var j = MyJob<T, float>();
    j.Schedule();
    return j;
}
void foo<T>() {
    makeJob<T>();
}
var bar() {
    foo<int>(); //可以
}

//将通用job嵌套在另一个类或者结构里
struct BlendJobWrapper<T> where T : struct, IBlendable {
    public T blendable;

    [BurstCompile()]
    public struct BlendJob : IJob {
        public T blendable;
        public void Execute() {}
    }

    public JobHandle Schedule(JobHandle dep = new JobHandle()) {
        return new BlendJob { blendable = blendable }.Schedule(dep);
    }
}
```

```c#
//示例 job化 排序
public unsafe static JobHandle Sort<T, U>(T* array, int length, U comp, JobHandle deps)
    where T : unmanged
    where U : IComparer<T>
{
    if (length == 0)
        return inputDeps;

    var segmentSortJob = new SegmentSort<T, U> { Data = array, Comp = comp, Length = length, SegmentWith = 1024 };
    var segmentSortMergeJob = new SegmentSortMerge<T, U> { Data = array, Comp = comp, Length = length, SegmentWith = 1024 }

    var segmentCount = (length + 1023) / 1024;
    var workerSegmentCount = segmentCount / math.max(1, JobsUtility.MaxJobThreadCount);
    var handle = segmentSortJob.Schedule(segmentCount, workerSegmentCount, deps);
    return segmentSortMergeJob.Schedule(segmentSortJobHandle);
}
// 请注意，排序分为两个作业：第一个将数组拆分为多个子节，然后分别（并行）对它们进行排序；
// 第二个等待第一个，然后将这些排序的子节合并为最终的排序结果。

// 但是，当前的定义，方法不会自动注册这两个泛型作业，可以用out参数
public unsafe static JobHandle Sort<T, U>(T* array, int length, U comp, JobHandle deps
        out SegmentSort<T, U> segmentSortJob, out SegmentSortMerge<T, U> segmentSortMergeJob)
    where T : unmanaged
    where U : IComparer<T>
{
    if (length == 0)
        return inputDeps;

    segmentSortJob = new SegmentSort<T, U> { Data = array, Comp = comp, Length = length, SegmentWidth = 1024 };
    segmentSortMergeJob = new SegmentSortMerge<T, U> { Data = array, Comp = comp, Length = length, SegmentWidth = 1024 };

    var segmentCount = (length + 1023) / 1024;
    var workerSegmentCount = segmentCount / math.max(1, JobsUtility.MaxJobThreadCount);
    var handle = segmentSortJob.Schedule(segmentCount, workerSegmentCount, deps);
    return segmentSortMergeJob.Schedule(segmentSortJobHandle);
}
// 虽然可以，但是太丑了

// 将两个作业类型包装
unsafe struct SortJob<T, U> :
    where T : unamanged
    where U : IComparer<T>
{
    public T* data;
    public U comparer;
    public int length;

    unsafe struct SegmentSort : IJobParalleFor {
        [NativeDisableUnsafePtrRestriction]
        public T* data;
        public U comp;
        public int length;
        public int segmentWidth;

        public void Execute(int index) {...}
    }

    unsafe struct SegmentSortMerge : IJob {
        [NativeDisableUnsafePtrRestriction]
        public T* data;
        public U comp;
        public int length;
        public int segmentWidth;

        public void Execute() {...}
    }

    public JobHandle Schedule(JobHandle dep = new JobHandle()) {
        if (length == 0)
            return inputDeps;
        var segmentSortJob = new SegmentSort<T, U> { Data = array, Comp = comp, Length = length, SegmentWidth = 1024 };
        var segmentSortMergeJob = new SegmentSortMerge<T, U> { Data = array, Comp = comp, Length = length, SegmentWidth = 1024 };

        var segmentCount = (length + 1023) / 1024;
        var workerSegmentCount = segmentCount / math.max(1, JobsUtility.MaxJobThreadCount);
        var handle = segmentSortJob.Schedule(segmentCount, workerSegmentCount, deps);
        return segmentSortMergeJob.Schedule(segmentSortJobHandle);
    }
}
// 这种安排中，Sort用户无需调用方法，创建实例SortJob并调用Schedule
// 也能自动 注册 SegmentSort SegmentSortMerge
```

### 建立游戏玩法

```
Unity.Transforms 提供用于定义世界空间变换 3D对象层次结构以及管理他们的系统的组件
Unity.Hybrid.Renderer 提供组件和系统以在Unity运行时中呈现ECS实体

无需定义自己的MonoBehaviour来存储实例数据和实现自定义游戏逻辑
可以定义ECS组件，在运行时存储数据，并为自定义逻辑编写系统
```

```
GameObject 转换
各种转换系统会识别MonoBehaviour组件，转换为基于ECS的组件
例如：Unity.Transforms 转换系统 检查 UnityEngine.Transform 添加ECS组件 如LocalToWorld

可以实现 IConvertGameObjectToEntity MonoBehaviour组件，实现自定义转换

ConvertToEntity组件，或者 是 SubScene的一部分，
ECS转换代码将转换GameObject
任何一种情况下， Unity.Transforms 和 Unity.Hybrid.Render 提供的转换系统都将处理GameObject或
Scene Asset 及其任何子 GameObject

用ConvertToEntity转换GameObject和用SubScene转换 的区别
ECS序列化并将它从转换的SubScene生成的实体数据保存到磁盘上
可以在运行时，非常快速地加载 或 流式传输 此序列化数据
ECS始终在运行时，将具有ConvertToEntity MonoBehaviours的GameObjects转换

最佳实践：使用标准的MonoBehaviours进行创作，并使用IConvertGameObjectToEntity将这些创作组件的值
应用于 IComponentData 结构 以供运行时使用。
通常，最方便编写的数据布局不是运行时最有效的数据布局

可以使用IConvertGameObjectToEntity来自定义SubScene中任何GameObject
具有ConvertToEntity MonoBehaviour的GameObject或具有MonoBehaviour的GameObject的子代的自定义转换

注意：：基于DOTS的应用程序的创作工作流是活跃开发的领域。大致轮廓以及到位，但是还会有许多变化
```

```c#
// 生成创作组件
// Unity可以自动为简单的运行时ECS组件生成创作组件
// Unity生成创作组件时，可以将包含ECS组件的添加脚本直接添加到编辑器中的GameObject上。
// 然后，使用检查器窗口设置组件的初始值

// Unity自动生成MonoBehaviour类，该类包含公共字段，并提供一个Conversion方法
// IComponentData
[GenerateAuthoringComponent]
public struct RotationSpeed_ForEach : IComponentData {
    public float RadiansPerSecond;
}

// 1  单个C# 文件中，只有一个组件可以包含 GenerateAuthoringComponent
//    并且不能有MonoBehaviour
// 2  ECS仅 反映 公共字段，并且与组件中指定的名词相同
// 3  ECS将IComponentData中的Entity类型的字段反映为它生成的
//    MonoBehaviour中的GameObject类型的字段
//    ECS将这些字段的GameObject或Prefab转换为引用的Prefab
// 4  IComponentData的Entity类型的字段在生成MonoBehaviour中反映为GameObject类型的字段
//    分配给这些字段的GameObject或Prefab将转换为参考的预制体
```

```c#
// IBufferElementData
[GenerateAuthoringComponent]
public struct IntBufferElement : IBufferElementData
{
    public int Value;
}
// 将生成一个 名为 IntBufferElementAuthoring的类
// 并公开一个 public List<int>
// 转换过程中 此列表转换为 DynamicBuffer<IntBufferElement>然后添加到转换的实体中

// 注意：IBufferElementData 对于包含两个或者更多字段的类型，无法自动生成创作组件
// 无法为具有显示布局的类型自动生成创作组件
```

### ECS深潜 老旧 看看思想就行了

https://rams3s.github.io/blog/2019-01-09-ecs-deep-dive/  

```
OOP -> A GameObject is a container.
ECS -> An entity is a key.

entity 只是一个id，唯一目的是将组件逻辑的分组在一起
component 只保存数据
system 负责处理/转换存储在component中的数据，为此遍历同类Component

好处：
内存/缓存友好
--仅从内存读取使用的内容
--定期的内存访问有助于硬件预读
--紧闭打包的数据更好地利用缓存
--没有多余的填充
利用多核并行
--System明确依赖相关Component
自动矢量化代码
--同时处理Component Group
```

```
Hybrid ECS
GameObjectEntity 在OnEnable时，自动创建Entity并注册到EntityManager
ComponentDataWrapper 编辑器可以编辑

[Serializable]
public struct RotSpeed : IComponentData {
    public float Value;
}
public class RotSpeedComponent : ComponentDataWrapper<RotationSpeed>{}
```

```
Archetype
组合
ECS + DoD -> An archetype is a unique set of arrays.
每次向实体中添加或者删除组件时，都必须将其移动到另一个原型中
这是有成本的，尤其是如果非常频繁地对许多实体执行此操作时
设计代码要考虑到这一事实

EntityArchetype archetype = EntityManager.CreateArchetype(
    typeof(Position),
    typeof(Rotation),
    typeof(LinearMovement));
var entity = EntityManager.CreateEntity(archetype);

实例化Prefab相当于框架自动管理这些原型
向组件添加删除组件时，该框架负责在原型之间移动实体
```

```
Chunk
Archetypes are made of chunks.
数组由于删除，插入困难，不适用原型
原型使用16kb块的链接列表

System
A system is a data transform.

Group
A group is a query on archetypes.
系统不能立即在原型上工作，每个组都是对原型的查询。
```

```c#
class PositionToRigidbodySystem : ComponentSystem {
    ComponentGroup m_Group;
    protected override void OnCreateManager(int capacity) {
        m_Group = GetComponentGroup(new EntityArchetypeQuery{
            All = new[] {
                ComponentType.Create<Position>(),
                ComponentType.Create<RigidBody>()
            }
        });
    }
    protected override void OnUpdate() {
        var positionTypeReadOnly = GetArchetypeChunkComponentType<Position>(true);
        var rigidbodyTypeRW = GetArchetypeChunkComponentType<Rigidbody>();

        NativeArray<ArchetypeChunk> chunks =
            m_Group.CreateArchetypeChunkArray(Allocator.TempJob);

        for (var chunkIndex = 0; chunkIndex < chunks.Length; chunkIndex++) {
            ArchetypeChunk chunk = chunks[chunkIndex];
            NativeArray<Position> rotations = chunk.GetNativeArray(positionTypeReadOnly);
            NativeArray<Rigidbody> rigidbodies = chunk.GetNativeArray(rigidbodyTypeRW);

            for (var i = 0; i < chunk.Count; i++) {
                rigidbodies[i].position = positions[i].Value;
            }
        }

        chunks.Dispose();
    }
}
```

```
减法组件 Subtractive Component
比如，波坏系统不需要处理具有无敌组件的实体，不论他们拥有什么其他组件
减法组件可以指定哪些组件不能进入结果集

标签组件 Tag Component
更精确的查询
namespace Sample.Boids {
    [Serializable]
    public struct BoidTarget : IComponentData{}

    [UnityEngine.DisallowMultipleComponent]
    public class BoidTargetComponent : ComponentDataWrapper<BoidTarget>{}
}

共享组件 Shared components are for grouping.
维持实体之间的严格秩序是昂贵的，却不一定合乎需要或者有用
比如共享网格和材质
每当添加共享组件到实体，实体都将按块分组，给定块中的所有实体
由此可知：共享组件的值不应该经常更改，
因为他们需要将所有组件数据从更改后的实体记忆到另一个块中
还有：一个块可以存储多个共享组件
使用多个共享组件时请一定要小心，不仔细进行操作，可能导致块的占用率低，从而浪费内存
因为共享组件的值的每种组合最终都将位于不同的块中
Debugger可以帮助发现这种问题
好例子：
public struct MeshInstanceRenderer : ISharedComponentData {
    public Mesh mesh;
    public Material material;
    public ShadowCastingMode castShadows;
    public bool receiveShadows;
}
```

```
世界
worlds are for isolation.
世界是孤立的，每个World包含一个EntityManager和一组ComponentSystems
一个世界的系统，实体，原型不适用于另一个世界

通常，使用一个World进行仿真，另一个进行渲染

在Megacity演示中，世界用于流式传输。一旦准备好一个新世界，它就会迅速合并，并且活动并渲染该世界。
```

```
反应系统

希望知道何时，实体首次被系统看到，执行初始设置
某个实体何时离开同一个系统，实体组执行清理
系统可能对合适更改组件感兴趣

“那里有一个，那里有很多。”
对于常规更新是正确的，对于低频率事件，也是。
因此我们要避免，每次实体增删组件时，组件值更新时，都将调用回调。

检测新的和删除的实体

不好的方法：
扫描整个实体列表，与上一次更新的实体列表比较

一种替代方案：
利用标签组件检测新的实体。在实体更改状态以供新系统处理时，
添加一个标签组件，并在首次处理该组件时将其删除。
但是这不好，又容易出错，你必须知道要添加哪个标签组件。
检测实体的去除也困难

更好的方法：
系统状态组件SSC ISystemStateComponent
public struct LocalToWorld : ISystemStateComponentData {
    public float4x4 Value;
}
case 0: 
Position/Rotation/Scale存在LocalToWorld不存在，
这是一个新的实体，TransformSystem可以执行此实体所需的任何初始设置，
然后将LocalToWorld组件添加到其中
case 1:
Position/Rotation/Scale存在LocalToWorld也存在，
这是一个跟踪的实体，没什么特别的
case 2:
Position/Rotation/Scale不存在LocalToWorld存在，
这是一个删除的实体，TransformSystem可以进行内部清理并删除系统状态组件本身

SSC很好地解决了添加/删除检测问题，但是，如何跟踪组件值的更改呢？
ECS框架无法为我们提供每个实体的检测，因为这将需要大量记录，而且效率低。
但是提供了每块可能变化的指示。

每个块中，每个数组带有一个版本号。
只读数组的版本号是在访问时不会被修改
但是，系统访问读写数组时，其版本号将分配给EntityManager.GlobalSystemVersion

有了版本号，我们手动遍历块，可以确定是否已经修改某些组件的值
var chunkRotationsChanged = ChangeVersionUtility.DidAddOrChange(
    chunk.GetComponentVersion(rotationType), lastSystemersion);
```

```
组件系统
Component System
在主线程上运行，不推荐

作业组件系统
Job Component System
在主线程上安排工作线程上执行的作业，在工作线程上运行

同步点
Sync Point
混合CS 和 JCS时应格外小心：所有结构更改都具有硬性同步点
例如，实体创建销毁，组件增加删除
无论何时发生这些事件之一，（在主线程上）
JCS计划的所有作业都必须在执行该更改之前完成，这回导致巨大的停顿

！！！坚持使用JCS，不要将它们和CS混合使用，除非你知道自己在做什么
public class RotationSpeedSystem : JobComponentSystem {
    [BurstCompile]
    struct RotationSpeedRotation : IJobProcessComponentData<Rotation, RotationSpeed> {
        public float dt;
        public void Execute(ref Rotation rotation, [ReadOnly]ref RotationSpeed speed) {
            rotation.Value = math.mul(
                math.normalize(rotation.Value),
                quaternion.axisAngle(math.up(), speed.Value * dt));
        }
    }

    protected override JobHandle OnUpdate(JobHandle inputDeps) {
        var job = new RotationSpeedRotation() {dt = Time.deltaTime};
        return job.Schedule(this, 64, inputDeps);
    }
}

也可以使用遍历作业中的块 IJobChunk
```

## [<主页](https://www.wangdekui.com/)