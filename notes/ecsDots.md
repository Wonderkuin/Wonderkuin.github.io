### [<主页](https://www.wangdekui.com/)

[第二部分](#second)

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
        var jobHandle = Entities.ForEach((Entity entity, int entityInQueryIndex, ref PositionAndRotation posAndRot) => {
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
    public void Convert(Entity entity, EntityManager dstManager, GameObjectConversionSystem conversionSystem) {
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

#### 总结



<div id="second"></div>

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
        float4(s.xyz, c.x) * c.yxxy * c.zzyz + s.yxxy * s.zzyz * float4(c.xyz, s.x) * float4(1f, -1f, -1f, 1f);
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

    public void Convert(Entity entity, EntityManager dstManager, GameObjectConversionSystem conversionSystem) {
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
        Graphics.DrawMesh(m_MeshFilter.mesh, meshRenderer.transform.localToWorldMatrix, m_Material, 0);
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
public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, BatchCullingContext cullingContext) {
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
public JobHandle OnPerformCulling(BatchRendererGroup rendererGroup, BatchCullingContext cullingContext) {
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

## [<主页](https://www.wangdekui.com/)