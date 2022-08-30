# Unity DOTS 旧的参考

---

#### 概念

DOTS : Data-Oriented Tech Stack  
由  
ECS : Entity Component System  
Job System  
Burst Compiler  
组成  

#### 高性能C#

High Performance C#  

标识  
Allocator.Persistent 持久化  
Allocator.Temp 临时  
Allocator.TempJob  
Allocator.Invalid  
Allocator.None  
Allocator.AudioKernel  

数据结构  
NativeArray  
NativeList  
NativeQueue  
NativeHashMap  
NativeSlice  
NativeMultiHashMap  
NativeStream  

```c#
// 扩展数组需要新建
NativeArray<Vector3> nativeArray = new NativeArray<Vector3>(5, Allocator.Temp);
for (int i = 0; i < nativeArray.Length; i++)
    nativeArray[i] = Vector3.one * i;
nativeArray[2] = Vector3.one;
nativeArray.Dispose();
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

#### Burst Compiler

导航菜单->Job->Burst->Open Inspector  

应用程序在手机安装时才进行AOT编译  
Project Settings  
Burst AOT Settings  

```c#
//强制使用同步编译，牺牲浮点数精度提高效率
[BurstCompile(CompileSynchronously = true)]
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast)]
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast, 
    FloatPrecision = FloatPrecision.High)]

[BurstCompile]//启用Burst
struct Job {}
[BurstDiscard]//如果代码包含堆内存对象，关闭Burst
struct void Method(){}
```

#### 自定义 ComponentSystemGroups

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

#### Version
```c#
entity.Version
System.LastSystemVersion
Chunk.ChangeVersion[]
EntityManager.m_ComponentTypeOrderVersion[]
SharedComponentDataManager.m_SharedComponentVersion[]
```

#### Entity自己渲染烘焙贴图

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
FrozenRenderSceneTag 是场景冻结，这个元素不会被运行时修改  
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
选Convert And Inject Gamebject不删除原始对象  
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