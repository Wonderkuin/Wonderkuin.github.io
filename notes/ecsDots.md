### [<主页](https://www.wangdekui.com/)

[指南第二部分](#index_second)
[文档与博客](#index_documents)

### 宏
不用托管的IComponentData  
UNITY_DISABLE_MANAGED_COMPONENTS  
Hybrid Renderer V2 URP或者DHRP开启  
ENABLE_HYBRID_RENDERER_V2  

#### 包安装
com.unity.entities  
Entities  
Jobs  
Burst  

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

```
Chunk component data
块组件，增加不会影响原始块中的其余实体

```

### ECS深潜

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