## [<主页](https://www.wangdekui.com/)


# DoTween
```c#
using DG.Tweening;
```

[Ease参考](https://easings.net/cn)

---

## 类型

[Tweener]() 控制动画的值

[Sequence]() 控制Tweener

[Tween]() 代表上面两种

[Nested Tween]() Sequence内部嵌套的Tween


---

## 前缀

[DO]() 所有Tween方法的前缀

```c#
transform.DOMoveY(100, 1);
transform.DORestart();
DOTween.Play();
```

[Set]() Tween对象的设置，除了Form

```c#
myTween.SetLoops(4, LoopType.Yoyo).SetSpeedBased();
```

[On]() Tween对象的回调

```c#
myTween.OnStart(myStartFunction).OnComplete(myCompleteFunction);
```

---

## 初始化

使用DOTween提供的global settings初始化Tween对象

可以在初始化使用SetCapacity，设置最大容量，等同于 DOTween.SetTweensCapacity

> static DOTween.Init(bool recycleAllByDefault = false, bool useSafeMode = true,  LogBehaviour logBehaviour = LogBehaviour.ErrorsOnly)  
`DOTween.Init();`  
`DOTween.Init(true, true, LogBehaviour.Verbose).SetCapacity(200, 10);`  

> recyleAllByDefault
>> 如果是true，则对Tween使用对象池，但是要注意解除绑定引用  
`.OnKill(() => myTweenReference = null);`  
可以修改静态属性 DOTween.defaultRecyclable  
可以为每个Tween设置特别的回收行为 SetRecyclable

> useSefaMode
>> 自动处理野指针，慢更安全。依赖于环境，在ios或者win10都可能失效

> logBehaviour
>> DOTween记录报错的等级

---

## 创建Tweener

Tweener是DOTween的工作单元，获取一个属性/字段并将值，动态变为目标值。

DOTween支持的类型:
**float**
**double**
**int**
**uint**
**long**
**ulong**
**Vector2/3/4**
**Quaternion**
**Rect**
**RectOffset**
**Color**
**string**

---

### 通用方法  
>`DOTween.To(() => myVector, x => myVector = x, new Vector3(3, 4, 8), 1);`  
`DOTween.To(() => myFloat, x => myFloat = x, 52, 1);`  
这是最灵活的方式，允许操作所有值，不管public private static dynamic  
快捷方式就是包装了通用方法  
和快捷方法一样，**通用方法有From版本**。
>> static DOTween.To(getter , setter, to, float duration)  
>>> getter  
A delegate that returns the value of the property to tween.   
Can be written as a lambda like this: ()=> myValue  
where myValue is the name of the property to tween.
>>> setter
A delegate that sets the value of the property to tween.  
Can be written as a lambda like this: x=> myValue = x  
where myValue is the name of the property to tween.
>>> to  
The end value to reach.  
>>> duration  
The duration of the tween.

---

### 快捷方法  
> DOTween对一些Unity对象如 Transform Rigidbody Material实现了快捷方法  
可以直接在对象上调用DOxxx方法，自动将对象作为Tween的对象
>> `transform.DOMove(new Vector3(2, 3, 4), 1);`  
`rigidbody.DOMove(new Vector3(2, 3, 4), 1);`  
`material.DOColor(Color.green, 1);`  

> 除另有说明外，每个快捷方法还有FROM版本。  
**特别重要** 当使用FROM时 目标对象将立刻被Set到FROM的位置  
是在代码执行的那一刻 而不是在Tween开始的那一刻  
>> `transform.DOMove(new Vector3(2, 3, 4), 1).From();`  
`rigidbody.DOMove(new Vector3(2, 3, 4), 1).From();`  
`material.DOColor(Color.green, 1).From();`  

---


### 其他通用方法

除非另有说明，这些方法也有FROM版本，只需要将From链接到Tweener，就会取代默认的To方式。

> static DOTween.Punch(getter, setter, Vector3 direction, float duration, int vibrato, floa elasticity);  
`// Punch upwards a Vector3 called myVector in 1 second`  
`DOTween.Punch(() => myVector, x => myVector = x, Vector3.up, 1);`  
>> 没有FROM版本  
向传入的Vector3方向，像弹簧一样弹出去又回来  
>>> vabrato 震动幅度  
elasticity 0到1 弹回时，向后弹 0代表不弹 1代表弹回一个身位

> static DOTween.Shake(getter, setter, float duration, float/Vector3 strength, int vibrato, float randomness, bool ignoreZAxis);  
`// Shake a Vector3 called myVector in 1 second`  
`DOTween.Shake(() => myVector, x => myVector = x, 1, 5, 10, 45, false`  
>> 没有FROM版本  
向传入的Vector3方向，摇晃  
>>> strength 震动强度， 可以使用Vector3而不是float代表每个轴的强度  
randomness 0到360 高于90要小心 表示多少震动是随机的  设为0将沿着单个方向晃动，并表现得像随机打击  
ignoreZAxis 如果是true 只沿着X轴和Y轴摇动  如果你使用Vector3 则不可用

> static DOTween.ToAlpha(getter, setter, float to, float duration);  
`// Tween the alpha of a color called myColor to 0 in 1 second`  
`DOTween.ToAlpha(() => myColor, x => myColor = x, 0, 1);`  
>> 改变颜色的alpha值

> static DOTween.ToArray(getter, setter, flaot to, float duration);  
`// Tween a Vector3 between 3 values, for 1 second each`  
`Vector3[] endValues = new[]{new Vector3(1,0,1),new Vector3(2,0,2),new Vector3(1,4,1)};`  
`float[] durations = new[] {1,1,1};`  
`DOTween.ToArray(() => myVector, x => myVector = x, endValues, durations);`  
>> 将Vector3缓动到给定的最终值 在每个段之间而不是整体上应用Tween

> static DOTween.ToAxis(getter, setter, float to, float duration, AxisConstraint axis);  
`// Tween the X of a Vector3 called myVector to 3 in 1 second`  
`DOTween.ToAxis(() => myVector, x => myVector = x, 3, 1);`  
`// Same as above, but tween the Y axis`  
`DOTween.ToAxis(() => myVector, x => myVector = x, 3, 1, AxisConstraint.Y);`  
>> 将一个轴从当前值缓动到给定值

> static DOTween.To(setter, float startValue, float endValue, float duration);  
`DOTween.To(MyMethod, 0, 12, 0.5f);`  
`// Where MyMethod is a function that accepts a float parameter`  
`// (which will be the result of the virtual tween)`  
` `  
`// Alternatively, with a lambda`  
`DOTween.To(x => someProperty = x, 0, 12, 0.5f);`  
>> 传入函数共享Tween的值

---

## 创建Sequence

序列可以包含Tweener和序列

可以任意嵌套，对深度没有任何限制

不必是一个一个的序列，可以使用Insert方法重叠

一个Tweener或者Sequence只能被**一个**Sequence嵌套

主Sequence将控制它的所有嵌套单元 嵌套的序列不再受人控制

**!重要** 不要使用空的Sequence

---

> 获取一个Sequence 保存引用  
>> static DOTween.Sequence()  
`Sequence mySequence = DOTween.Sequence();`  

> 添加 tween interval callback  
>> [!所有方法都应当在Sequence开始前应用，一旦Sequence开始，就确定了  
所有嵌套的Tweener 和 Sequence 必须被初始化好，一旦开始就改不了]()
Delay 和 Loop 非无限循环 可以在 嵌套Tween内工作
>>> Append(Tween tween)   
`mySequence.Append(transform.DOMoveX(45, 1));`  

>>> AppendCallback(TweenCallback callback)  
`mySequence.AppendCallback(MyCallback);`  

>>> AppendInterval(float interval)  
`mySequence.AppendInterval(interal);//添加间隔`  

>>> Insert(float atPosition, Tween tween)
`mySequence.Insert(1, transform.DOMoveX(45, 1));`  

>>> InsertCallback(float atPosition, TweenCallback callback)  
`mySequence.InsertCallback(1, MyCallback);`  

>>> Join(Tween tween)  
`// The rotation tween will be played together with the movement tween`  
`mySequence.Append(transform.DOMoveX(45, 1));`  
`mySequence.Join(transform.DORotate(new Vector3(0, 180, 0), 1));//同时进行`  

>> Prepend(Tween tween)  
`mySequence.Prepend(transform.DOMoveX(45, 1));`  

>> PrependCallback(TweenCallback callback)
`mySequence.PrependCallback(MyCallback);`  

>> PrependInterval(float interval)  
`mySequence.PrependInterval(interval);`  

[小技巧:可以只创建回调序列，用作计时器]()

---

```c#
// 创建一个序列
// Grab a free Sequence to use
Sequence mySequence = DOTween.Sequence();
// Add a movement tween at the beginning
mySequence.Append(transform.DOMoveX(45, 1));
// Add a rotation tween as soon as the previous one is finished
mySequence.Append(transform.DORotate(new Vector3(0,180,0), 1));
// Delay the whole Sequence by 1 second
mySequence.PrependInterval(1);
// Insert a scale tween for the whole duration of the Sequence
mySequence.Insert(0, transform.DOScale(new Vector3(3,3,3), mySequence.Duration()));
```

```c#
// 同上
Sequence mySequence = DOTween.Sequence();
mySequence.Append(transform.DOMoveX(45, 1))
  .Append(transform.DORotate(new Vector3(0,180,0), 1))
  .PrependInterval(1)
  .Insert(0, transform.DOScale(new Vector3(3,3,3), mySequence.Duration()));
```

---

## 设置 选项 回调

### 全局设置

---

> static LogBehaviour DOTween.logBehaviour  
>> Default: ErrorsOnly  
LogBehaviour.ErrorsOnly  error
LogBehaviour.Default  error warning
LogBehaviour.Verbose  all

> static bool DOTween.maxSmoothUnscaledTime  
>> Default: 0.15f  
如果 useSmoothDeltaTime == true 在与时间无关的的Tween中，被认为是最大时间

> static bool DOTween.nestedTweenFailureBehaviour  
>> Default: NestedTweenFailureBehaviour.TryToPreserveSequence  
在安全模式处于活动状态且嵌套在Sequence中的补间失败的情况下，行为会发生。  

> static bool DOTween.onWillLog<LogType, object>  
>> Default: false  
用于拦截日志  

> static bool DOTween.showUnityEditorReport  
>> 在UnityEditor跑完DOTween后，获得调试信息，多少个Tweener Sequence  

> static bool DOTween.timeScale  
>> Default: 1  
全局时间缩放  

> static bool DOTween.useSafeMode  
>> Default: true  
如果是 false 要自己手动管理 tween 对象  
要注意ios 和 windows下.net 的策略  

> static bool DOTween.useSmoothDeltaTime  
>> Default: false  
如果是 true DOTween会对UpdateType.Normal和UpdateType.Late使用Time.smoothDeltaTime  
动画更加流畅  

> static DOTween.SetTweensCapacity(int maxTweeners, int maxSequences)
>> Default: 200 Tweener ， 50 Sequence  
"c++ 使用 reserve 来避免不必要的重新分配"  
`DOTween.SetTweenCapacity(2000, 100);`  

---

> static bool DOTween.defaultAutoKill  
>> Default: TRUE  

> static bool DOTween.defaultAutoPlay  
>> Default: AutoPlay.All  

> static float DOTween.defaultEaseOvershootOrAmplitude  
>> Default: 1.70158f  
默认的 overshoot amplitude  

> static float DOTween.defaultEasePeriod  
>> Default: 0  
ease 的周期  

> static Ease DOTween.defaultEaseType  
>> Default: Ease.OutQuad  

> static LoopType DOTween.defaultLoopType  
>> Default: LoopType.Restart  
默认循环类型应用于所有新创建的**涉及循环**的补间

> static bool DOTween.defaultRecyclable  
>> Default: false  

> static bool DOTween.defaultTimeScaleIndependent  
>> Default: false  
设置默认情况下是否应考虑Unity的timeScale

> static UpdateType DOTween.defaultUpdateType  
>> Default: UpdateType.Normal  

---

### Tweener Sequence 设置

> float timeScale  
>> Default: 1  
`myTween.timeScale = 0.5f;`  
[!提示：可以通过另一个补间对该值进行补间，以实现平滑的慢动作效果]()


### 下面的设置可以运用到所有Tween对象。 运行时，除了SetLoops 和 SetAs ，也都可使用

> SetAs(Tween tween \ TweenParams tweenParams)  
>> id ease loops delay timeScale callbacks / TweenParams对象  
不要拷贝特定的SetOptions的设置的参数，它们每次都需要手动应用  
`transform.DOMoveX(4, 1).SetAs(myOtherTween);`  

> SetAutoKill(bool autoKillOnCompletion = true)
>> 如果你需要手动控制Tween对象  
`transform.DOMoveX(4, 1).SetAutoKill(false);`  

> SetEase(Ease easeType \ AnimationCurve animCurve \ EaseFunction customEase)  
>> Sequence默认是EaseType.Linear，如果赋值为其他类型，Sequence整体的时间线会改变，而不是每个嵌套对象的Ease被改变  
注意：Back和Elastic缓动（意味着低于或超出起始值和结束值的任何缓动）均不适用于路径  
>> overshoot过冲  
>>> Eventual overshoot to use with Back ease (default is 1.70158), or number of flashes to use with Flash ease.  
配合 Back Flash 使用
>> amplitude振幅 
>>> Eventual amplitude to use with Elastic ease (default is 1.70158).  
配合 Elastic 使用
>> peirod周期  
>>> Eventual period to use with Elastic ease (default is 0), or power to use with Flash ease.  
配合 Elastic Flash 使用  
`transform.DOMoveX(4, 1).SetEase(Ease.InOutQuint);`  
`transform.DOMoveX(4, 1).SetEase(myAnimationCurve);`  
`transform.DOMoveX(4, 1).SetEase(MyEaseFunction);`  
>> 特殊 Ease  
>> Flash InFlash OutFlash InOutFlash  
>>> overshoot  
>>>> Indicates the total number of flashes to apply. An even number will end the tween on the starting value, while an odd one will end it on the end value.  
>>> period  
>>>> Indicates the power in time of the ease, and must be between -1 and 1.
0 is balanced, 1 fully weakens the ease in time, -1 starts the ease fully weakened and gives it power towards the end.  
`image.DOColor(myColor, 0.7f).SetEase(Ease.Flash, 16, 1);//闪电效果`  
`image.DOColor(myColor, 0.7f).SetEase(Ease.Flash, 15, 1);//奇数`  
>> 附加： EaseFactory  
>>> **EaseFactory.StopMotion** is an extra layer you can add to your easings, making them behave as if they were playing in stop-motion (practically, they will simulate the given FPS while tweening). It can be applied as a wrapper to any ease.  
EaseFactory.Stopotio(int fps, Ease\AnimationCurve\EaseFunction ease)  
`transform.DOMoveX(4, 1).SetEase(EaseFactory.StopMoton(5, Ease.InOutQuint));`  

> SetId(object id)  
>> 设置id int string 会更快  
`transform.DOMoveX(4, 1).SetId("supertween");`  

> SetLink(GameObject target, LinkBehaviour linkBehaviour = linkehaviour.KillOnDestroy)
>> 将tween的生命周期链接到GameObject  
注意：添加到Sequence中无效  
>> target  
>>> The link target (unrelated to the target set via SetTarget).  
和SetTarget设置的目标无关  
>> linkBehaviour  
>>> The behaviour to use (LinkBehaviour.KillOnDestroy is always evaluated—and set by default—even if you choose another one).  
>> `transform.DOMoveX(4, 1).SetLink(aGameObject, LinkBehaviour.PauseOnDisableRestartOnEnable);`  

> SetLoops(int loops, LoopType loopType = LoopType.Restart)  
开始了就确定了，再设置无效 在Sequence中，无限循环无效 -1 无限循环  
>> LoopType.Restart 重新启动  
LoopType.Yoyo 来回晃荡  
LoopType.Incremental  Each time a loop ends the difference between its endValue and its startValue will be added to the endValue, thus creating tweens that increase their values with each loop cycle. 仅适用于Tweener
`transform.DOMoveX(4, 1).SetLoops(3, LoopType.Yoyo);`  

> SetRecyclable(bool recyclable)  
如果不设置 使用DOTween初始化的默认值  
>> recyclable  
如果是真 用完会回收 否则被销毁  
`transform.DOMoveX(4, 1).SetRecyclable(true);`  

> SetRelative(bool isRelative = true)  
如果设为真 所有endValue变成beginValue + endValue  
如果是Sequence 所有嵌套子对象都会变成这样  
[！重要 对FROM没有影响，因为当设置为FROM时，要选是否是相对的]()  
[如果Tween已经开始，则没有效果]()  
`transform.DOMoveX(4, 1).SetRelative();`  

> SetTarget(object target)  
Sets the target for the tween (which can then be used as a filter with DOTween's static methods). This is useful when using the generic tween method, while shortcuts automatically set this to the target of your shortcut.  
>> 谨慎使用 如果只想用id SetId就好  
`DOTween.To(()=> myInstance.aFloat, (x)=> myInstance.aFloat = x, 2.5f, 1).SetTarget(myInstance);`  

> SetUpdate(UpdateType updateType, bool isIndependentupdate = false)  
>> UpdateType.Normal  
>>> 每帧更新 在Update中  
>> UpdateType.Late  
>>> 每帧更新 在LateUpdate中  
>> UpdateType.Fixed  
>>> 使用 Fixedupdate 更新  
>> UpdateType.Manual  
>>> 由 DOTween.ManualUpdate 手动更新  
>> isIndependentUpdate  
如果是true 忽略Unity的Time.timeScale  
[！注意: timeScale可能是0， 此时FixedUpdate无法运行]()



## [<主页](https://www.wangdekui.com/)