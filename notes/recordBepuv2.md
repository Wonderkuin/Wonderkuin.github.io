# A different kind of C#
[too early for me!](https://www.bepuentertainment.com/blog/2018/8/30/modern-speedy-c-the-bepuphysics-v2-perspective)

---

# 前言

```
托管语言经常遇到性能问题
BEPUphysics v2 比 v1 性能提高了10倍以上
看看用到了什么技术
```

---

# 值类型

```c#
// Value Types

// 严格的OOP对于性能就是一条痛苦之路，这适用于任何语言，甚至C++
// 相对CPU 内存非常慢，每次触及新内存都会触发CPU停顿
// 大型惯用的OOP程序往往会形成一个巨大的引用网络，需要不连贯且通常未缓存的内存访问
// 无论你的编译器多好，都会使你的CPU大部分时间处于空闲状态
// 作者引用了性能分析结果 Intel VTune已经有免费许可证了! 
// CPI 每条指令的周期数 DRAM Bound 等待主内存请求的周期百分比 

RawList<SolverUpdateable> solverUpdateables = new RawList<SolverUpdateable>();
public abstract class SolverUpdateable;
// 引用类型实际上是一个指针数组，这些引用所指向的数据不太可能是有序且连续的
// 无论这个数组如何遍历，访问仍然会跳过整个内存并遭受大量缓存未命中

struct Snoot
{
	public int A;
	public float B;
	public long C;
}

public static void Boop()
{
	var snoots = new Snoot[4];
	for (int i = 0; i < snoots.Length; ++i)
	{
		ref var snoot = ref snoots[i];
		var value = i+1;
		snoot.A = value;
		snoot.B = value;
		snoot.C = value;
	}

	ref var snootBytes = ref Unsafe.As<Snoot, byte>(ref snoots[0]);
	for (int i = 0; i < snoots.Length; ++i)
	{
		Console.Write($"Snoot {i} bytes: ");
		for (int j = 0; j < Unsafe.SizeOf<Snoot>(); ++j)
		{
			Console.Write($"{Unsafe.Add(ref snootBytes, i * Unsafe.SizeOf<Snoot>() + j}, ");
		}
		Console.WriteLine();
	}
}
//Snoot 0 bytes: 1, 0, 0, 0, 0, 0, 128, 63, 1, 0, 0, 0, 0, 0, 0, 0,
//Snoot 1 bytes: 2, 0, 0, 0, 0, 0, 0, 64, 2, 0, 0, 0, 0, 0, 0, 0,
//Snoot 2 bytes: 3, 0, 0, 0, 0, 0, 64, 64, 3, 0, 0, 0, 0, 0, 0, 0,
//Snoot 3 bytes: 4, 0, 0, 0, 0, 0, 128, 64, 4, 0, 0, 0, 0, 0, 0, 0,

// v2 represents a group of constraints
// v2中表示的一种约束
public struct TypeBatch
{
	public RawBuffer BodyReferences;
	public RawBuffer PrestepData;
	public RawBuffer AccumulatedImpulses;
	public RawBuffer Projection;
	public Buffer<int> IndexToHandle;
	public ConstraintCount;
	public int TypeId;
}

//这里的一个重要注意事项是约束的属性——主体引用、前步数据等——被拆分到不同的数组中。
//并非约束处理的每个阶段都需要访问每个约束属性。
//例如，求解迭代根本不需要 PrestepData；尝试将前步数据与其余属性捆绑在一起只会在求解过程中浪费缓存行中的宝贵空间。
//这有助于内存带宽。

//但是，TypeBatch 中根本没有处理逻辑。是原始数据。无类型，甚至 - RawBuffer 只是指一个字节块。它是如何使用的？

//正如名称所暗示的那样，每个 TypeBatch 仅包含一种类型的约束。TypeBatches 是...处理...由 TypeProcessor 处理。
//TypeProcessor 是为了理解单一类型的约束而构建的，并且没有自己的状态；它们可以很容易地实现为静态函数的函数指针。在求解器中，您会发现：

public TypeProcessor[] TypeProcessors;

//当求解器遇到 TypeId 为 3 的 TypeBatch 时，它可以进行快速数组查找以找到知道如何处理该 TypeBatch 数据的 TypeProcessor。

//（旁注：每个虚拟调用的成本在整个类型批处理中分摊。
//执行类型批处理往往需要花费数百纳秒的绝对最小值，因此为间接添加 2-4 纳秒几乎无关紧要。
//比较这个到在数组中有一堆 SolverUpdateable 引用并对每个实例进行虚拟调用的 OOPy 方法。
//与数据缓存未命中相比没什么大不了的，但仍然可以避免毫无意义的开销。）

//最后，这可以使用如下所示的紧密代码循环对连续的数据数组进行爆破：

[MethodImpl(MethodImplOptions.AggressiveInlining)]
public void Solve(ref BodyVelocities velocityA, ref BodyVelocities velocityB, ref AngularHingeProjection projection, ref Vector2Wide accumulatedImpulse)
{
    Vector3Wide.Subtract(velocityA.Angular, velocityB.Angular, out var difference);
    Matrix2x3Wide.TransformByTransposeWithoutOverlap(difference, projection.VelocityToImpulseA, out var csi);
    Vector2Wide.Scale(accumulatedImpulse, projection.SoftnessImpulseScale, out var softnessContribution);
    Vector2Wide.Add(softnessContribution, csi, out csi);
    Vector2Wide.Subtract(projection.BiasImpulse, csi, out csi);
    Vector2Wide.Add(accumulatedImpulse, csi, out accumulatedImpulse);
    ApplyImpulse(ref velocityA.Angular, ref velocityB.Angular, ref projection, ref csi);
}

//这是纯数学，没有分支，也没有约束缓存未命中。
//虽然 GPU 在计算吞吐量方面受到关注，但 CPU 在这方面仍然非常出色，尤其是在指令级并行性方面。
//上面的每个操作都是使用 SIMD 指令在多个约束条件下同时执行的（稍后我将详细介绍）。

//在那段代码中隐藏了一些缓存未命中，但它们是求解器算法的本质所必需的——体速度不能与约束数据一起存储，
//因为它们之间没有一对一的映射，所以它们必须是不连贯地聚集和分散。
//（旁注：这些缓存未命中可以部分缓解；库尝试按访问模式对约束进行分组。）

//概括
//值类型对于 C# 来说并不陌生，对于一般的编程当然也不陌生，但控制内存访问模式对于性能来说绝对是至关重要的。
//没有这一点，所有其他优化尝试实际上都将无关紧要。
```

---

# JIT

```c#
/*
JIT 并不真正需要恶魔般的授权

在 C# 中编写有关性能的系列文章而不提及 Just-In-Time (JIT) 编译器会有点愚蠢。
与提前为特定平台 (AOT) 预编译程序集的离线工具链不同，许多 C# 应用程序在最终用户的设备上按需编译。
虽然这在理论上确实为 JIT 提供了更多关于目标系统的知识，但它也限制了可用于编译的时间。
大多数用户不会容忍 45 秒的启动时间，即使它确实使之后的一切运行速度提高了 30%。 

值得一提的是，有AOT编译路径，部分平台需要AOT。
Mono 历来提供了这样的路径，.NET Native 用于 UWP 应用程序，并且较新的CoreRT正在稳步前进。
AOT 并不总是意味着深度离线优化，但放宽时间限制至少在理论上是有帮助的。
还有正在进行的分层编译工作，最终可能会导致更高的优化层。 

一个普遍的担忧是，运行当今任何 JIT 质量的编译器都将导致较差的优化，从而使 C# 成为真正高性能代码的死胡同。
JIT 确实无法像离线流程那样进行深度优化，这在各种用例中都会体现出来。
*/

[MethodImpl(MethodImplOptions.AggressiveInlining)]
public void Solve(ref BodyVelocities velocityA, ref BodyVelocities velocityB, ref BallSocketProjection projection, ref Vector3Wide accumulatedImpulse)
{
    Vector3Wide.Subtract(velocityA.Linear, velocityB.Linear, out var csv);
    Vector3Wide.CrossWithoutOverlap(velocityA.Angular, projection.OffsetA, out var angularCSV);
    Vector3Wide.Add(csv, angularCSV, out csv);
    Vector3Wide.CrossWithoutOverlap(projection.OffsetB, velocityB.Angular, out angularCSV);
    Vector3Wide.Add(csv, angularCSV, out csv);
    Vector3Wide.Subtract(projection.BiasVelocity, csv, out csv);

    Symmetric3x3Wide.TransformWithoutOverlap(csv, projection.EffectiveMass, out var csi);
    Vector3Wide.Scale(accumulatedImpulse, projection.SoftnessImpulseScale, out var softness);
    Vector3Wide.Subtract(csi, softness, out csi);
    Vector3Wide.Add(accumulatedImpulse, csi, out accumulatedImpulse);

    ApplyImpulse(ref velocityA, ref velocityB, ref projection, ref csi);
}

/*
这是一个非常紧凑的循环中的一大堆数学。
正是在这种情况下，您可能期望更好的优化器提供显着的胜利。
在不剧透的情况下，我可以告诉你，JIT可以 用这里生成的程序集做得更好。

现在想象一下微软的某个人（或者甚至是你，毕竟它是开源的！）在狂热的梦想中获得超自然知识，并用他们的灵魂来授权 RyuJIT。
RyuJIT v666 受到下面深不可测的黑暗的不利祝福，以某种方式使您的 CPU 在 0 个周期内执行所有指令。
仅作用于寄存器的指令是完全免费的，具有无限的吞吐量，唯一剩余的成本是等待来自缓存和内存的数据。

如果我们假设唯一的 瓶颈是内存带宽，那么在影响之前加速最多大约是 1.25 倍，之后是 1.55 倍。
换句话说，帧时间将下降到不超过 24-30 毫秒。
实际上，由内存延迟引起的停顿会阻止实现那些理想的加速。

RyuJIT v666，以及所有地球优化编译器的扩展，都不能大大加快这种模拟的速度。
即使我用 C 重写了它，期望超过百分之几也是不明智的。
此外，鉴于在大多数情况下计算的改进速度比内存快，新的处理器往往从恶魔中获益更少。

当然，并非每个模拟都受内存带宽限制。涉及网格等复杂对撞机的模拟往往会为魔术编译器提供更多工作空间。
它永远不会那么 令人印象深刻。

那么，JIT 生成的程序集会更好吗？绝对的，而且它正在 迅速好转。
对于非常特定类型的代码，有时是否会出现严重问题，尤其是在不受内存瓶颈限制的情况下？是的。

但是它足以创建许多复杂的快速应用程序吗？是的！
*/

// 看不懂
// 总之 JIT在生成高效代码方面做的相当不错
```

---

# 注意你的step

```c#
// 三种不同的struct

struct VFloat
{
	public float X;
	public float Y;
	public float Z;
}

struct VNumerics3
{
	public Vector<float> X;
	public Vector<float> Y;
	public Vector<float> Z;
}

struct VAvx
{
	public Vector256<float> X;
	public Vector256<float> Y;
	public Vector256<float> Z;
}

// 手动内联

// VFloat
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static void Add(in VFloat a, in VFloat b, out VFloat result)
{
  result.X = a.X + b.X;
  result.Y = a.Y + b.Y;
  result.Z = a.Z + b.Z;
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static VFloat operator +(in VFloat a, in VFloat b)
{
  VFloat result;
  result.X = a.X + b.X;
  result.Y = a.Y + b.Y;
  result.Z = a.Z + b.Z;
  return result;
}


for (int j = 0; j < innerIterationCount; ++j)
{
  ref var value = ref Unsafe.Add(ref baseValue, j);
  var r0 = value + value;
  var r1 = value + value;
  var r2 = value + value;
  var r3 = value + value;
  var i0 = r0 + r1;
  var i1 = r2 + r3;
  var i2 = i0 + i1;
  accumulator += i2;
}

//从历史上看，即使函数被内联，对值类型使用运算符也意味着对参数和返回值进行大量复制。
//（在操作员参数上允许“输入”对此有一点帮助，至少在 JIT 无法在没有帮助的情况下消除副本的情况下。）

//作为补偿，许多具有任何程度性能敏感度的 C# 库（如 XNA 及其后代）都提供了 ref/out 重载。
//这有帮助，但不能有效地使用运算符总是会损害可读性。
//让 refs 遍布你的代码也不是太好，但在大多数情况下，参数（不需要调用站点修饰）使我们免于这种情况。

//但是为了获得最佳性能，您必须咬紧牙关并手动内联。这是一个无法维护的混乱的秘诀，但它稍微快了一点！


//手动内联时 Vector{T} 和 AVX 都比 VFloat 慢，但考虑到一半的添加被优化掉，这是预期的。
//不幸的是，相对于手动内联实现，看起来即使是非运算符函数也会受到影响。

//手动内联时，8 宽 AVX 也比 4 宽 Vector{T} 快一点。
//在 3770K 上，相关的 4 宽指令和 8 宽指令具有相同的吞吐量，因此预计会非常接近。
//边际减速源于 Vector{T} 实现使用额外的 vmovupd 指令加载输入值。
//手动缓存局部变量中的值实际上有所帮助。

//略
```

---

# 善待GC

```c#
//理想情况下，这意味着不收集任何垃圾
//GC 善意最常见的形式是重用对实例的引用，而不是简单地按需实例化新的实例。 


//这有几个问题：

//池化实例对分代垃圾收集起作用。Gen0 系列非常便宜。
//更高的世代更昂贵，并且只有在它们活着的情况下才会将分配提升到更高的世代。
//池的全部意义在于使分配保持活动状态，因此如果最终收集这些池中的实例，则成本会更高。

//GC 必须以某种方式跟踪所有活着的引用。
//从概念上讲，这是一个图遍历，其中节点是跟踪的分配，边是这些分配之间的引用。
//图中的节点和边越多，GC 要做的工作就越多。
//池化直接增加了这个图的大小。即使在应用程序被拆除之前从不收集池化实例，任何其他垃圾收集也会受到堆复杂性增加的影响。


//换句话说，使用 GC 跟踪的实例池是承诺永远不会导致 GC，或者在 GC 最终发生时可能会出现长时间的故障。

非托管型没有GC跟踪

struct Unmanaged
{
  public float Q;
  public float P;
}
unsafe static void ItsOkayGCJustTakeANap(int elementCount)
{
  var pointer = (Unmanaged*)Marshal.AllocHGlobal(elementCount * sizeof(Unmanaged));
  for (int i = 0; i < elementCount; ++i)
  {
    pointer[i] = new Unmanaged { Q = 3, P = i };
  }
  Marshal.FreeHGlobal(new System.IntPtr(pointer));
}

//如果您真的愿意，您可以自由地用 C# 编写基本的 C。万岁？

//BEPUphysics v2 使用的 BEPUutilities 库实现了自己的内存管理。
//目前它不是基于AllocHGlobal构建的——而是从大对象堆中分配大块，然后从它们中进行子分配——但这是一个相当小的实现细节。

//BEPUutilities BufferPool 类是库中大部分内存的来源。
//它是一个简单的 2 次幂存储桶缓冲区分配器，以浪费一些空间为代价提供快速分配和释放。
//如果您浏览源代码，您可能会遇到各种 Buffer{T} 实例；所有这些都来自 BufferPool。


pool.Take<int>(128, out var buffer);
for (int i = 0; i < 128; ++i)
{
  buffer[i] = i;
}
pool.Return(ref buffer);


//如果不提及Span{T} 和朋友，任何有关 C# 中内存管理的博客文章都是不完整的。它提供了对任何类型内存的清晰抽象。
//BEPUphysics 根本不使用 Span{T}。
//这部分是因为 v2 在 Span{T} 准备好之前就开始了开发，部分是因为几乎所有的内存都知道是不受管理的。
//如果您是该库的用户，您可能仍然会发现在与不是基于非托管内存假设构建的系统进行互操作时，跨度非常有用。


//概括
//您也可以体验手动内存管理的乐趣。让自己沉浸在内存泄漏和访问冲突中。
//来吧，不要害怕；构建您自己的自定义分配器，因为您可以.

//或者不要那样做，而是坚持使用 Span{T} 之类的东西。
//但至少考虑让你的垃圾 GC 休息一下，并尝试使用值类型的大缓冲区而不是十亿个对象实例。
//将它与大量的批处理混合在一起，即使它有时看起来有点像 1995 年的代码，您也会获得具有惊人简单性的效率秘诀。
```

---

# 泛型滥用 Generics Abuse

```c#
//假设您有一个函数，它围绕用户提供的其他一些逻辑块执行一些复杂的逻辑。
//例如，一个函数接受一个主体和约束列表并将约束应用于主体，如下所示：
public struct Body
{
  public float X;
  public float Y;
  public float Otherness;
  public float Ineffability;
}

interface IConstraint
{
  int A { get; }
  int B { get; }
  void Apply(ref Body a, ref Body b);
}

struct IneffabilityConstraint : IConstraint
{
  public int A { get; set; }

  public int B { get; set; }

  public void Apply(ref Body a, ref Body b)
  {
    a.Ineffability = a.Ineffability + b.Otherness * (b.X * b.X + b.Y * b.Y);
  }
}

static void ApplyConstraintsThroughInterface(Span<Body> bodies, Span<IConstraint> constraints)
{
  for (int i = 0; i < constraints.Length; ++i)
  {
    var constraint = constraints[i];
    constraint.Apply(ref bodies[constraint.A], ref bodies[constraint.B]);
  }
}

//循环获取主体引用并将它们交给相关约束。这里没什么特别的。它有效，相当简单。
//而且它比它需要的要慢得多。


//约束范围包含 IConstraint 类型的元素。
//即使我们在上面只有一个 IConstraint 实现，想象一下如果这是一个库 - 可能有任意数量的其他 IConstraint 实现，并且约束范围可能包含一堆不同的类型。
//编译器将调用正确实现的责任推给执行时间。

//在没有任何其他帮助的情况下，它几乎可以保证在最终程序集中中有一些沉重的无法内联的函数调用。


/// Boxing 装箱

//我们可以在 span 中存储任何 IConstraint 实现的引用类型实例，因为它只存储引用。
//但是值类型实例不会被 GC 直接跟踪，并且默认情况下没有引用。
//而且，由于我们谈论的是具有固定步长的连续数组，我们不能只是将任意大小的值类型实例推入其中。

//为了使一切正常，值类型实例被“装箱”到引用类型实例中。然后将该新实例的引用存储在数组中。

//换句话说，将值类型插入形状类似于引用类型的槽中会自动为您创建引用类型实例。它有效，但现在你给了可怜的 GC 更多的工作要做，这很粗鲁。

//当然，您可以只坚持引用类型，但随后您就会拥有一个非常复杂的引用网络供 GC 跟踪。老实说，这也太不体贴了。


/// 专攻速度

//理想情况下，我们可以消除虚拟调用和任何其他形式的最后一秒间接调用，允许内联并避免任何装箱。

//如果我们愿意将传入的数据组织成相同类型的批次，这是可能的。这为我们正在使用的特定类型专门化整个循环打开了大门，至少在概念上是这样。这看起来如何？

static void ApplyConstraintsWithGenericsAbuse<TConstraint>(Span<Body> bodies, Span<TConstraint> constraints) where TConstraint : IConstraint
{
  for (int i = 0; i < constraints.Length; ++i)
  {
    ref var constraint = ref constraints[i];
    constraint.Apply(ref bodies[constraint.A], ref bodies[constraint.B]);
  }
}

// 约束

// 这让一切变得不同。由于额外的类型意识，JIT 不仅可以消除虚拟调用，还可以内联包含的逻辑。

// 在比较这两种方法的微基准测试中，仅这一点就占了大约 4 倍的加速。每个虚拟调用具有更多逻辑的较大程序不会显示出太大的差异，但成本是真实的。


// 用class 替代 struce  没有约束好，但是也比原始的struct快30%~50%

//当涉及到泛型时，JIT 以不同的方式处理值类型和引用类型。
//JIT 利用了这样一个事实，即所有引用类型都能够共享相同的实现，即使这意味着无法内联特定的实现。
//（此行为可能会发生变化；据我所知，可能已经存在某些引用类型是专门化的情况。）

//但即使 JIT 想要，它也不能有效地跨值类型共享实现。
//它们的大小不同，除非装箱，否则没有间接方法表。
//因此，它不是将每个值类型装箱以共享相同的代码，而是选择输出类型专用的实现。

//这真的很有用。



//这种编译器行为在引擎中的所有地方都（ab）使用。您可以将其从简单的事情（如高效回调）一直延伸到几乎模板地狱。
public unsafe interface INarrowPhaseCallbacks
{
  void Initialize(Simulation simulation);

  bool AllowContactGeneration(int workerIndex, CollidableReference a, CollidableReference b);

  bool ConfigureContactManifold(int workerIndex, CollidablePair pair, ConvexContactManifold* manifold, out PairMaterialProperties pairMaterial);

  bool ConfigureContactManifold(int workerIndex, CollidablePair pair, NonconvexContactManifold* manifold, out PairMaterialProperties pairMaterial);

  bool AllowContactGeneration(int workerIndex, CollidablePair pair, int childIndexA, int childIndexB);

  bool ConfigureContactManifold(int workerIndex, CollidablePair pair, int childIndexA, int childIndexB, ConvexContactManifold* manifold);  

  void Dispose();
}

//另一种情况：想要枚举通过约束连接到给定物体的一组物体？没问题。
public void EnumerateConnectedBodies<TEnumerator>(int bodyHandle, ref TEnumerator enumerator) where TEnumerator : IForEach<int>


//像上面这样的例子还有很多，特别是在光线/查询过滤和处理方面。我们可以比回调更进一步吗？当然！这是上述玩具“约束”示例的真实扩展：
public abstract class TwoBodyTypeProcessor<TPrestepData, TProjection, TAccumulatedImpulse, TConstraintFunctions>
  : TypeProcessor<TwoBodyReferences, TPrestepData, TProjection, TAccumulatedImpulse>
  where TConstraintFunctions : struct, IConstraintFunctions<TPrestepData, TProjection, TAccumulatedImpulse>

//在没有将太多垃圾邮件复制到这篇博文中的情况下，这个通用定义提供了足够的信息来创建由所有两个主体约束共享的许多函数。
public unsafe override void SolveIteration(ref TypeBatch typeBatch, ref Buffer<BodyVelocity> bodyVelocities, int startBundle, int exclusiveEndBundle)
{
  ref var bodyReferencesBase = ref Unsafe.AsRef<TwoBodyReferences>(typeBatch.BodyReferences.Memory);
  ref var accumulatedImpulsesBase = ref Unsafe.AsRef<TAccumulatedImpulse>(typeBatch.AccumulatedImpulses.Memory);
  ref var projectionBase = ref Unsafe.AsRef<TProjection>(typeBatch.Projection.Memory);
  var function = default(TConstraintFunctions);
  for (int i = startBundle; i < exclusiveEndBundle; ++i)
  {
    ref var projection = ref Unsafe.Add(ref projectionBase, i);
    ref var accumulatedImpulses = ref Unsafe.Add(ref accumulatedImpulsesBase, i);
    ref var bodyReferences = ref Unsafe.Add(ref bodyReferencesBase, i);
    int count = GetCountInBundle(ref typeBatch, i);
    Bodies.GatherVelocities(ref bodyVelocities, ref bodyReferences, count, out var wsvA, out var wsvB);
    function.Solve(ref wsvA, ref wsvB, ref projection, ref accumulatedImpulses);
    Bodies.ScatterVelocities(ref wsvA, ref wsvB, ref bodyVelocities, ref bodyReferences, count);
  }
}


//包含原始无类型缓冲区的 TypeBatch 在传递给知道如何解释它的 TypeProcessor 时被赋予意义。
//单独的类型函数不必担心重新解释无类型内存或收集速度；这一切都由共享实现处理，没有虚拟调用开销。

//我们还能走得更远吗？当然！让我们看看ConvexCollisionTask，它以矢量化的方式处理成批的碰撞对。
//正如您现在可能期望的那样，它有一些重要的泛型：

public class ConvexCollisionTask<TShapeA, TShapeWideA, TShapeB, TShapeWideB, TPair, TPairWide, TManifoldWide, TPairTester> : CollisionTask
  where TShapeA : struct, IShape where TShapeB : struct, IShape
  where TShapeWideA : struct, IShapeWide<TShapeA> where TShapeWideB : struct, IShapeWide<TShapeB>
  where TPair : struct, ICollisionPair<TPair>
  where TPairWide : struct, ICollisionPairWide<TShapeA, TShapeWideA, TShapeB, TShapeWideB, TPair, TPairWide>
  where TPairTester : struct, IPairTester<TShapeWideA, TShapeWideB, TManifoldWide>
  where TManifoldWide : IContactManifoldWide

//换句话说，这要求输入是两个可矢量化的形状和一个能够处理这些形状类型的函数。
//但是实际的批处理执行比仅仅内联用户提供的函数更有趣。查看循环体：
if (pairWide.OrientationCount == 2)
{
  defaultPairTester.Test(
    ref pairWide.GetShapeA(ref pairWide),
    ref pairWide.GetShapeB(ref pairWide),
    ref pairWide.GetSpeculativeMargin(ref pairWide),
    ref pairWide.GetOffsetB(ref pairWide),
    ref pairWide.GetOrientationA(ref pairWide),
    ref pairWide.GetOrientationB(ref pairWide),
    countInBundle,
    out manifoldWide);
}
else if (pairWide.OrientationCount == 1)
{
  //Note that, in the event that there is only one orientation, it belongs to the second shape.
  //The only shape that doesn't need orientation is a sphere, and it will be in slot A by convention.
    //注意，如果只有一个方向，则属于第二个形状。
  //唯一不需要方向的形状是球体，按照惯例它会在插槽A中。
  Debug.Assert(typeof(TShapeWideA) == typeof(SphereWide));
  defaultPairTester.Test(
    ref pairWide.GetShapeA(ref pairWide),
    ref pairWide.GetShapeB(ref pairWide),
    ref pairWide.GetSpeculativeMargin(ref pairWide),
    ref pairWide.GetOffsetB(ref pairWide),
    ref pairWide.GetOrientationB(ref pairWide),
    countInBundle,
    out manifoldWide);
}
else
{
  Debug.Assert(pairWide.OrientationCount == 0);
  Debug.Assert(typeof(TShapeWideA) == typeof(SphereWide) && typeof(TShapeWideB) == typeof(SphereWide), "No orientation implies a special case involving two spheres.");
  //Really, this could be made into a direct special case, but eh.
  //真的，这可以变成一个直接的特例，但是呃。
  defaultPairTester.Test(
    ref pairWide.GetShapeA(ref pairWide),
    ref pairWide.GetShapeB(ref pairWide),
    ref pairWide.GetSpeculativeMargin(ref pairWide),
    ref pairWide.GetOffsetB(ref pairWide),
    countInBundle,
    out manifoldWide);
}

//Flip back any contacts associated with pairs which had to be flipped for shape order.
//翻转与必须翻转形状顺序的对关联的任何接触。
if (pairWide.HasFlipMask)
{
  manifoldWide.ApplyFlipMask(ref pairWide.GetOffsetB(ref pairWide), pairWide.GetFlipMask(ref pairWide));
}


//概括
//C# 泛型可能不是图灵完备的元编程语言，但它们仍然可以做一些非常有用的事情，而不仅仅是拥有特定类型的列表。通过对编译器的了解，您可以零开销地表达各种简单或复杂的抽象。

//你也可以制造可怕的混乱。
```

---

# 走得更远

```c#
//编写高效的 SIMD 加速代码可能很棘手。考虑单个交叉产品：
result.X = a.Y * b.Z - a.Z * b.Y;
result.Y = a.Z * b.X - a.X * b.Z;
result.Z = a.X * b.Y - a.Y * b.X;

//如果您想尝试脑筋急转弯，请想象您有可以同时处理 4 条车道数据的减法、乘法和车道洗牌操作。
//尝试为一对输入 3d 向量提供表达相同数学结果所需的最少运算次数。

//然后，将其与标量版本进行比较。在不破坏太多的情况下，使用 4-wide 操作不会快 4 倍。

//任何时候您尝试对标量代码进行严格矢量化时，都会出现这样的问题。
//您很少能从 128 位操作中受益，更不用说更新的 256 位或 512 位操作了。
//只是没有足够的独立计算通道来使机器充满实际工作。

//幸运的是，大多数计算密集型程序都涉及对一堆实例多次执行同一件事。
//如果您假设这种批处理模型从头开始构建您的应用程序，它甚至不会比传统方法涉及更多的困难（尽管它确实有 不同的困难）。

//假设我们有大量的 2^20 操作要执行，每次计算：
result = dot(dot(cross(a, b), a) * b, dot(cross(c, d), c) * d)


//结构数组

//我们将从一个简单的标量实现和一个狭义的矢量化版本开始，作为比较的基础。它们看起来像这样：


public struct Vector3AOS
{
  public float X;
  public float Y;
  public float Z;
  ...
}

//v1
for (int i = 0; i < LaneCount; ++i)
{
  ref var lane = ref input[i];
  Vector3AOS.CrossScalar(lane.A, lane.B, out var axb);
  Vector3AOS.CrossScalar(lane.C, lane.D, out var cxd);
  var axbDotA = Vector3AOS.DotScalar(axb, lane.A);
  var cxdDotC = Vector3AOS.DotScalar(cxd, lane.C);
  Vector3AOS.ScaleScalar(lane.B, axbDotA, out var left);
  Vector3AOS.ScaleScalar(lane.D, cxdDotC, out var right);
  results[i] = Vector3AOS.DotScalar(left, right);
}

//v2
for (int i = 0; i < LaneCount; ++i)
{
  ref var lane = ref input[i];
  Vector3AOS.Cross3ShuffleSSE(lane.A, lane.B, out var axb);
  Vector3AOS.Cross3ShuffleSSE(lane.C, lane.D, out var cxd);
  Vector3AOS.DotSSE(axb, lane.A, out var axbDotA);
  Vector3AOS.DotSSE(cxd, lane.C, out var cxdDotC);
  Vector3AOS.ScaleSSE(lane.B, axbDotA, out var left);
  Vector3AOS.ScaleSSE(lane.D, cxdDotC, out var right);
  Vector3AOS.DotSSE(left, right, out var result);
  results[i] = Sse41.Extract(result, 0);
}

//它们在内存访问级别上都非常相似。
//从数组中加载相关的整个结构实例（因此是结构数组！），对它做一些工作，将它存储起来。
//在较低级别上，SSE 版本有点奇怪。
//例如，Vector3AOS.Cross3ShuffleSSE 函数并不像之前的叉积代码那样直观：

const byte yzxControl = (3 << 6) | (0 << 4) | (2 << 2) | 1;
var shuffleA = Sse.Shuffle(a, a, yzxControl);
var shuffleB = Sse.Shuffle(b, b, yzxControl);
result = Sse.Subtract(Sse.Multiply(a, shuffleB), Sse.Multiply(b, shuffleA));
result = Sse.Shuffle(result, result, yzxControl);


//这些内部函数调用中只有三个实际上在执行算术运算。 
//尽管很丑陋，但 SSE 变体即使在当前 alpha 状态的内部函数支持下也更快，速度提高了大约 25-30%。与 4 倍的速度相差甚远。
//为了做得更好，我们需要利用我们连续做一百万次同样的事情这一事实。



//数组的结构

//结构数组 (AOS) 按实例布置内存。
//每个结构都包含与一个实例相关联的所有数据。
//这非常直观和方便，但正如我们所见，它不是宽矢量化的自然表示。
//充其量，您必须在运行时将 AOS 内存布局转换为矢量化形式，也许遍历多个实例并创建矢量化的实例属性包。
//但这需要我们花在实际计算上的宝贵时间！

//如果我们只是将每个实例属性存储在它自己的大数组中会怎样？

Buffer<float> ax, ay, az, bx, by, bz, cx, cy, cz, dx, dy, dz, result;

public override void Execute()
{
  for (int i = 0; i < LaneCount; i += Vector<float>.Count)
  {
    ref var ax = ref Unsafe.As<float, Vector<float>>(ref this.ax[i]);
    ref var ay = ref Unsafe.As<float, Vector<float>>(ref this.ay[i]);
    ref var az = ref Unsafe.As<float, Vector<float>>(ref this.az[i]);
    ref var bx = ref Unsafe.As<float, Vector<float>>(ref this.bx[i]);
    ref var by = ref Unsafe.As<float, Vector<float>>(ref this.by[i]);
    ref var bz = ref Unsafe.As<float, Vector<float>>(ref this.bz[i]);
    ref var cx = ref Unsafe.As<float, Vector<float>>(ref this.cx[i]);
    ref var cy = ref Unsafe.As<float, Vector<float>>(ref this.cy[i]);
    ref var cz = ref Unsafe.As<float, Vector<float>>(ref this.cz[i]);
    ref var dx = ref Unsafe.As<float, Vector<float>>(ref this.dx[i]);
    ref var dy = ref Unsafe.As<float, Vector<float>>(ref this.dy[i]);
    ref var dz = ref Unsafe.As<float, Vector<float>>(ref this.dz[i]);

    Cross(ax, ay, az, bx, by, bz, out var axbx, out var axby, out var axbz);
    Cross(cx, cy, cz, dx, dy, dz, out var cxdx, out var cxdy, out var cxdz);
    Dot(axbx, axby, axbz, ax, ay, az, out var axbDotA);
    Dot(cxdx, cxdy, cxdz, cx, cy, cz, out var cxdDotC);
    Scale(bx, by, bz, axbDotA, out var leftx, out var lefty, out var leftz);
    Scale(bx, by, bz, cxdDotC, out var rightx, out var righty, out var rightz);
    Dot(leftx, lefty, leftz, rightx, righty, rightz, out Unsafe.As<float, Vector<float>>(ref result[i]));
  }
}

//结果证明这比原始标量版本快 2.1 倍。
//这是一个改进，即使没有我们想要的那么大。
//内存带宽开始阻塞一些周期，但这不是唯一的问题。

//使用 SOA 布局时的一个潜在问题是过多的预取地址流。
//即使每个访问流都是完全顺序的，跟踪一堆不同的内存位置可能会在内存系统的几个部分遇到瓶颈。
//上面的13个活跃地址流已经足够引人注目了；如果以相同的 SOA 布局表示，当前的Hinge约束仅针对投影数据就有47 个流。


//您还可以尝试更改循环以在移动到下一个通道之前在所有通道上执行每个单独的操作
//（即乘以所有ay * bz，将其存储在临时中，然后执行所有 az * by 并存储它，依此类推）。
//考虑到一次只有两三个在飞行，这确实会消除地址流的担忧。
//不幸的是，每个操作都会驱逐 L1、L2 以及许多消费者处理器上的一百万个实例，甚至所有 L3。
//没有数据重用是可能的，最终结果是一个巨大的内存带宽瓶颈，性能比朴素的标量版本差大约 3.5 倍。
//现在考虑如果循环被修改为一次处理一个缓存友好的属性块会是什么样子……



///将它们捆绑在一起：数组结构数组

//我们想要这样的东西：

//允许高效的向量化执行，
//无需 SPMD/SIMT 工具即可使用（参见后面的部分），
//使用起来感觉相当自然（保留标量逻辑），
//缓存友好，不需要宏观级别的平铺逻辑，并且
//不会滥用预取器。

//有一个选项可以满足所有这些要求：AOSOA，Array Of Structures Of Arrays。
//使用之前的捆绑结构结构布局，将数组长度限制为单个 SIMD 包的大小，
//将所有这些较小的包数组推入一个结构中，并将所有包含这些结构的包数组存储在一个数组中。
//或者，用一种不那么语法上的折磨的方式来表达：

public unsafe struct Vector3AOSOANumerics
{
  public Vector<float> X;
  public Vector<float> Y;
  public Vector<float> Z;
  ...
}

struct Input
{
  public Vector3AOSOANumerics A;
  public Vector3AOSOANumerics B;
  public Vector3AOSOANumerics C;
  public Vector3AOSOANumerics D;
}

Buffer<Input> input;
Buffer<Vector<float>> results;

public override void Execute()
{
  for (int i = 0; i < LaneCount / Vector<float>.Count; ++i)
  {
    ref var lane = ref input[i];
    Vector3AOSOANumerics.Cross(lane.A, lane.B, out var axb);
    Vector3AOSOANumerics.Cross(lane.C, lane.D, out var cxd);
    Vector3AOSOANumerics.Dot(axb, lane.A, out var axbDotA);
    Vector3AOSOANumerics.Dot(cxd, lane.C, out var cxdDotC);
    Vector3AOSOANumerics.Scale(lane.B, axbDotA, out var left);
    Vector3AOSOANumerics.Scale(lane.D, cxdDotC, out var right);
    Vector3AOSOANumerics.Dot(left, right, out results[i]);
  }
}

//在性能方面，它的运行速度大约是标量版本的 2.4 倍。
//不是 4 倍，但正如您现在可能猜到的那样，部分原因是内存带宽瓶颈。
//基准测试对于每个加载的字节根本没有足够的数学。
//（如果您将代码更改为仅在 L1 中工作，它会比标量版本快 3.24 倍。
//对于我的机器上使用的 4 种宽操作仍然不是完美的缩放，但更接近 - 并且可以通过额外的编译器改进来缩小差距.)

//AOSOA 布局几乎用于引擎的每个计算成本高的部分。
//所有约束和几乎所有凸碰撞检测例程都使用它。
//在大多数模拟中，这些广泛矢量化的代码路径占据了大部分执行时间。
//这是 v2 如此快速的一个重要原因。


///关于自动向量化和朋友

//“足够聪明的编译器”理论上可以采用第一个简单的标量实现并将其直接转换为完全矢量化的形式。
//不幸的是，CoreCLR JIT 目前不支持自动向量化，甚至离线优化编译器都在挣扎。
//C、C++、C# 和它们的所有朋友都是极其宽松的语言，它们根本无法为编译器以矢量化有时需要的极端方式安全地转换代码提供足够的保证。

//但是自动向量化和内在函数并不是处理事情的唯一方法。
//通过稍微限制语言自由度以给编译器额外的保证，您可以编写简单的标量代码，将其转换为（主要是）高效的矢量化代码。
//最常见的例子是像 HLSL 这样的 GPU 着色语言。现代 GPU 上执行的每个“线程”的行为很像 CPU 向量指令的单通道
//（尽管 GPU 倾向于使用更广泛的执行单元和不同的延迟隐藏策略，以及其他细微差别）。

//这种方法通常称为 SPMD（单程序多数据）或 SIMT（单指令多线程），它可以实现与手写内在函数非常接近的性能，而工作量要少得多。
//它在 CPU 领域不太常见，但仍然有ISPC和OpenCL等选项。
//（Unity 的Burst可能符合条件，但它仍在开发中，我不清楚它在自动向量化器到成熟 SPMD 转换的范围内究竟处于什么位置。）

//此外，即使在 HLSL 中，您仍然需要负责有效的内存布局。即使所有代码都已优化矢量化，随机访问散布在各处的一堆属性也会降低性能。


///一种尺寸并不适合所有人
//不是所有东西都需要使用 AOSOA 布局。
//即使性能很重要，AOOSA 也不会自动成为正确的选择。始终遵循内存访问模式。

//bepuphysics v2 中的一个例子是身体数据。
//所有这些都以常规的旧 AOS 格式存储，即使 PoseIntegrator 对其进行了一些重要的数学运算。
//为什么？因为 PoseIntegrator 不是一大块帧时间，它通常已经受内存带宽限制，
//并且碰撞检测和约束求解例程经常请求身体数据。
//将所有物体的速度存储在单个缓存行中可以最大限度地减少求解器必须接触的缓存行的数量。
```

---

# 总结

```
高山仰止，景行行止，虽不能至，而心向往之
```