# DOTween
### [官方文档](http://dotween.demigiant.com/documentation.php)
### [Ease参考](https://easings.net/)

---

## 类型

```
[Tweener]() 控制动画的值
[Sequence]() 控制Tweener
[Tween]() 代表上面两种
[Nested Tween]() Sequence内部嵌套的Tween
```

---

## 前缀

```c#
[DO]() 所有Tween方法的前缀
transform.DOMoveY(100, 1);
transform.DORestart();
DOTween.Play();
```

```c#
[Set]() Tween对象的设置，除了Form
myTween.SetLoops(4, LoopType.Yoyo).SetSpeedBased();
```

```c#
[On]() Tween对象的回调
myTween.OnStart(myStartFunction).OnComplete(myCompleteFunction);
```

---

## 初始化

```
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
```

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

> AudioMixer
>> DOSetFloat(string floatName, float to, float duration)

> AudioSource
>> DOFade(float to, float duration)  
DOPitch(float to, float duration)  

> Camera
>> DOAspect(float to, float duration)  
DOColor(float to, float duration)  
DOFarClipPlane(float to, float duration)  
DONearClipPlane(float to, float duration)  
DOOrthoSize(float to, float duration)  
DOPixelRect(Rect to, float duration)  
DORect(Rect to, float duration)  
DOShakePosition(float duration, float/Vector3 strength, int vibrato, float randomness, bool fadeOut) 没有FROM版本  
DOShakeRotation(float duration, float/Vector3 strength, int vibrato, float randomness, bool fadeOut) 没有FROM版本  

> Light  
>> DOColor(Color to, float duration)  
DOIntensity(float to, float duration) 强度  
DoShadowStrength(float to, float duration)  
DOBlendableColor(Color to, float duration) 允许多个DOBlendableColor同时工作 不像DOColor那样被替换  

> LineRenderer
>> DOColor(Color2 startValue, Color2 endValue, float duration)  
将目标的颜色更改为给定的颜色  
注意，此方法还需要插入start颜色，因为LineRenderers无法获得startColor  
Color2是一种特殊的DOTween结构 允许在一个变量中存储两种颜色  
`myLineRenderer.DOColor(new Color2(Color.white, Color.white), new Color2(Color.green, Color.black), 1);`  

> Material  
>> DOColor(Color to, float duration)  
DOColor(Color to, string property, float duration)  
``// Tween the specular value of a material``  
``myMaterial.DOColor(Color.green, "_SpecColor", 1);``  
DOColor(Color to, int propertyID, float duration)  
DOFade(float to, float duration)  
DOFade(float to, string preperty, float duration)  
DOFade(float to, int prepertyID, float duration)  
DOFloat(float to, string preperty, float duration)  
DOFloat(float to, int prepertyID, float duration)  
DOGradientColor(Gradient to, float duration)  
``GradientColor不使用Alpha``  
DOGradientColor(Gradient to, string property, float duration)  
DOGradientColor(Gradient to, int propertyID, float duration)  
DOOffset(Vector2 to, float duration)  
DOOffset(Vector2 to, string property, float duration)  
DOOffset(Vector2 to, int propertyID, float duration)  
DOTiling(Vector2 to, float suration)  
DOTiling(Vector2 to, string property, float duration)  
DOTiling(Vector2 to, int propertyID, float duration)  
DOVector(Vector4 to, string property, float duration)  
DOVector(Vector4 to, int propertyID, float duration)  
DOBlendableColor(Color to, float suration)  
DOBlendableColor(Color to, string property, float duration)  
``// Tween the specular value of a material``  
``myMaterial.DOBlendableColor(Color.green, "_SpecColor", 1);``  
DOBlendableColor(Color to, int propertyID, float duration)  

> Rigidbody  
>> DOMove(Vector3 to, float duration, bool snapping)  
DOMoveX/Y/X(float to, float duration, bool snapping)  
DOJump(Vector3 endValue, float jumpPower, int numJumps, float duration, bool snapping)  
`Y轴有跳跃效果`  
`返回一个Sequence而不是Tweener，SetSpeedBase无效`  
`jumpPower 跳跃的最大高度=jumpPower+offsetOfY`  
DORotate(Vector3 to, float duration, RotateMode mode)  
`需要Vector3而不是Quaternion，可以用myQuaternion.eulerAngles`  
`Fast 默认值 旋转以最短路线进行，角度不超过360°`  
`FastBeyond360 旋转将超过360°`  
`WorldAxisAdd  使用世界坐标轴和高级精度模式（如使用transform.Rotate（Space.World）时）将给定旋转添加到变换中。 在这种模式下，最终值始终被视为相对值`  
`LocalAxisAdd 将给定的旋转添加到变换的局部轴上（例如，在Unity的编辑器中启用“本地”开关或使用transform.Rotate（Space.Self）旋转对象时）。 在这种模式下，最终值始终被视为相对值`  
DOLookAt(Vector3 towards, float duration, AxisConstraint axisConstraint = AxisConstraint.None, Vector3 up = Vector3.up)  
>> ProOnly  
DOSpiral(float duration, Vector3 axis = null, SpiralMode mode = spiralMode.Expand, float speed = 1, float frequency = 10, float depth = 0, bool snapping = false)  
`transform.DOSpiral(3, Vector3.forward, SpiralMode.ExpandThenContract, 1, 10);`  
`螺旋升降`  

> RigidBody2D   
>> DOMove(Vector3 to, float duration, bool snapping)  
DOMoveX/Y(float to, float duration, bool snapping)  
DOJump(Vector3 endValue, float jumpPower, int numJumps, float duartion, bool snapping)  
DORotate(float toAngle, float duration)  

SpriteRenderer
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  
DOGradientColor(Gradient to, float duration)  
DOBlendableColor(Color to, float duration)  

TrailRenderer
> DOResize(float toStartWidth, float toEndWidth, float duration)  
DOTime(float to, float duration)  

Transform
> Move
>> DOMove(Vector3 to, float duration, bool snapping)  
>
>> DOMoveX/Y/Z(float to, float duration, bool snapping)  
>
>> DOLocalMove(Vector3 to, float duration, bool snapping)  
>
>> DOLocalMoveX/Y/Z(float to, float duration, bool snapping)  
>
>> DOJump(Vector3 endValue, float jumpPower, int numJumps, float duration, bool snapping)  
>
>> DOLocalJump(Vector3 endValue, float jumpPower, int numJumps, float duration, bool snapping) 

> Rotate
>> DORotate(Vector3 to, float duration, RotateMode mode)  
当在某些极限情况下仅沿X轴进行小旋转时，目标将摆动到位置。 如果发生这种情况，请改用DORotateQuaternion  
mode Fast FastBeyond360 WorldAxisAdd LocalAxisAdd  
>
>> DORotateQuaternion(Quaternion to, float duration)  
此方法特殊，不支持LoopType.Incremental循环  
>
>> DOLocalRotate(Vector3 to, float duraton, RotateMode mode)  
>
>> DOLocalRotateQuaternion(Quaternion to, float duration)  
>
>> DOLookAt(Vector3 towards, float duration, AxisConstraint axisContraint = AxisConstraint.None, Vector3 up = Vector3.up)  

> Scale  
>> DOScale(float/Vector3 to, float duration)  
>
>> DOScaleX/Y/Z(float to, float duration)  

> Punch no FROM
>> DOPunchPosition(Vector3 punch, float duration, int vibrato, float elasticity, bool snapping)  
>
>> DOPunchRotation(Vector3 punch, float duration, int vibrato, float elasticity)  
>
>> DOPunchScale(Vector3 punch, float duration, int vibrato, float elasticity)  

> Shake no FROM  
>> DOShakePosition(float duration, float/Vector3 strength, int vibrato, float randomness, bool snapping, bool fadeOut)  
>
>> DOShakeRotation(float duration, float/Vector3 strength, int vibrato, float randomness, bool fadeOut)  
>
>> DOShakeScale(float duration, float/Vector3 strength, int vibrato, float randomness, bool fadeOut)  

> Path no FROM  
>> DOPath(Vector3[] waypoints, float duration, PathType path = Linear, PathMode = Full3D, int resolution = 10, Color gizmoColor = null)  
可以SetOptions和SetLookAt  
pathType Linear CatmullRom CubicBezier  
pathMode Ignore 3D side-scroller2D top-down2D  
resolution 曲线点 默认为10 一般5就够了  
立方贝塞尔路径  
CubicBezier路径航路点必须为三的倍数，其中每三组代表：1）路径航路点； 2）IN控制点（前一个航路点上的控制点）； 3）OUT控制点（航路点上的控制点） 新的航点）。 请记住，第一个航路点始终是自动添加的，并由目标的当前位置确定（并且没有控制点）  
![CubicBezier](http://dotween.demigiant.com/_imgs/content/dotween_path_cubicBezier.png)  
>
>> DOLocalPath(Vector3[] waypoints, float duration, PathType path = Linear, PathMode = Full3D, int resolution = 10, Color gizmoColor = null)

> Blendable tweens
>> DOBlendableMoveBy(Vector3 by, float duration, bool snapping)  
`// Tween a target by moving it by 3,3,0`  
`// while blending another move by -3,0,0 that will loop 3 times`  
`// (using the default OutQuad ease)`  
`transform.DOBlendableMoveBy(new Vector3(3, 3, 0), 3);`  
`transform.DOBlendableMoveBy(new Vector3(-3, 0, 0), 1f).SetLoops(3, LoopType.Yoyo);`  
>
>> DOBlendableLocalMoveBy(Vector3 by, float duration, bool snapping)  
>
>> DOBlendableRotateBy(Vector3 by, float duration, RotateMode mode)  
>
>> DOBlendableLocalRotateBy(Vector3 by, float duration, RotateMode mode)  
>
>> DOBlendableScaleBy(Vector3 by, float duration)  

> ProOnly Spiral no FROM  
>> DOSpiral(float duration, Vector3 axis = null, SpiralMode mode = SpiralMode.Expand, float speed = 1, float frequency = 10, float depth = 0, bool snapping = false)  
`transform.DOSpiral(3, Vector3.forward, SpiralMode.ExpandThenContract, 1, 10);`  

Tween
> DOTimeScale(float toTimeScale, float duration)  
---

#### UGUI

CanvasGroup
> DOFade(float to, float duration)  

Graphic
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  
DOBlendableColor(Color to, float duration)

Image
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  
DOFillAmount(float to, float duration)  
DOGradientColor(Gradient to, float duration)  
DOBlendableColor(Color to, float duration)  

LayoutElement
> DOFlexibleSize(Vector2 to, float duration, bool snapping)  
DOMinSize(Vector2 to, float duration, bool snapping)  
DOPreferredSize(Vector2 to, float duration, bool snapping)  

Outline
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  

RectTransform
> DOAnchorMax(Vextor2 to, float duration, bool snapping)  
DOAnchorMin(Vextor2 to, float duration, bool snapping)  
DOAnchorPos(Vextor2 to, float duration, bool snapping)  
DOAnchorPosX/Y(float to, float duration, bool snapping)  
DOAnchorPos3D(Vector3 to, float duration, bool snapping)  
DOAnchorPos3DX/Y/Z(float to, float duration, bool snapping)  
DOJumpAnchorPos(Vector2 endValue, float jumpPower, int numJumps, float duration, bool snapping)  
DoPivot(Vector2 to, float duration)  
DOPivotX/Y(float to, float duration)  
DOPunchAnchorPos(Vector2 punch, float duration, int vibrato, float elasticity, bool snapping)  
DOShakeAnchorPos(Vector2 duration, float/Vector3 strength, int vibrato, float randomness, bool snapping, bool fadeOut)  
DOSizeDelta(Vector2 to, float duration, bool snapping)  

ScrollRect  
> DONormalizedPos(Vector2 to, float duration, bool snapping)  
DOHorizontalNormalizedPos(float to, float duration, bool snapping)  
DOVerticalPos(float to, float duration, bool snapping)  

Slider
> DOValue(float to, float duration, bool snapping = false)  

Text
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  
DOText(string to, float duration, bool = richTextEnabled = true, ScrambleMode scrambleMode = ScrambleMode.None, string scrambleChars = null) 打字机效果
DOBlendableColor(Color to, float duration)  

#### Pro only 2D Toolkit shortcuts

tk2dBaseSprite
> DOScale(Vector3 to, float duration)  
DOScaleX/Y/Z(float to, float duration)  
DOColor(Color to, float duration)  
DOFade(float to, float duration)  

tk2dSlicedSprite
> DOScale(Vector2 to, float duration)  
DOScaleX/Y(float to, float duration)  

tk2dTextMesh
> DOColor(Color to, float duration)  
DOFade(float to, float duration)  
DOText(string to, float duration, bool richTexEnabled = true, ScambleMode scrambleMode = ScrambleMode.None, string scrambleChars = null)  

#### Pro only TextMesh Pro shortcuts

> DOScale(float to, float duration)  
DOColor(Color to, float duration)  
DOFaceColor(Color to, float duration)  
DOFaceFade(float to, float duration)  
DOFade(float to, float duration)  
DOFontSize(float to, float duration)  
DOGlowColor(Color to, float duration)  
DOMaxVisibleCharacters(int to, float duration)  
DOOutlineColor(Color to, float duration)  
DOText(string to, float duration, bool richTextEnabled = true, ScrambleMode scrambleMode = ScrambleMode.None, string scrambleChars = null)  

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


#### 下面的设置可以运用到所有Tween对象。 运行时，除了SetLoops 和 SetAs ，也都可使用

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
`transform.DOMoveX(4, 1).SetUpdate(UpdateType.Late, true);`  
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

#### 下面是回调

> OnComplete(TweenCallback callback)  
完成时被触发 包括所有循环都完成时   
`transform.DOMoveX(4, 1).SetComplete(MyCallback);`  

> OnKill(TweenCallback callback)  
销毁时触发  
`transform.DOMoveX(4, 1).OnKill(MyCallback);`  

> OnPlay(TweenCallback callback)  
第一次Play 从暂停开始Play 都会触发  
`transform.DOMoveX(4, 1).OnPlay(MyCallback);`  

> OnPause(TweenCallback callback)  
从播放变成暂停时触发 如果autoKill，完成时也触发  
`transform.DOMoveX(4, 1).OnPause(MyCallback);`  

> OnRewind(TweenCallback callback)  
Tween的Rewinded 调用Rewind函数 倒着播放到起始状态  
注意: Rewinding一个已经rewinded的Tween不会触发  
`transform.DOMoveX(4, 1).OnRewind(MyCallback);`  

> OnStart(TweenCallback callback)  
开始时  
`transform.DOMoveX(4, 1).OnStart(MyCallback);`  

> OnStepComplete(TweenCallback callback)  
每次循环完成时  
`transform.DOMoveX(4, 1).OnStepComplete(MyCallback);`  

> OnUpdate(TweenCallback callback)  
每次Update  
`transform.DOMoveX(4, 1).OnUpdate(MyCallback);`  

> OnUpdate(TweenCallback callback)  
每次Update  
`transform.DOMoveX(4, 1).OnUpdate(MyCallback);`  

> OnWayPointChange(TweenCallback<int> callback)  
when a path tween's current waypoint changes  
这是一个特殊的回调，它需要接受int类型的参数 它将是新更改的航点索引  
`void Start() {`  
`   transform.DOPath(waypoints, 1).OnWaypointChange(MyCallback);`  
`}`  
`void MyCallback(int waypointIndex) {`  
`   Debug.Log("Waypoint index changed to " + waypointIndex);`  
`}`  

#### 嵌套的Tween仍然能正常使用回调  
#### 可用lambda  
`// Callback without parameters`  
`transform.DOMoveX(4, 1).OnComplete(MyCallback);`  
`// Callback with parameters`  
`transform.DOMoveX(4, 1).OnComplete(() => MyCallback(someParam, someOtherParam));`  

### Tweener 特别的函数和设置

#### 对Sequence无效  在运行时，除了SetEase 其他无效

> From(bool isRelative = false)  
>> 必须在其他设置之前调用，除了Tween Specific Options  
isRelative 如果是真 使用 currentValue+endValue 作为endValue  
`// Regular TO tween`  
`transform.DOMoveX(2, 1);`  
`// FROM tween`  
`transform.DOMoveX(2, 1).From();`  
`// FROM tween but with relative FROM value`  
`transform.DOMoveX(2, 1).From(true);`  

> From(T fromValue, bool setImmediately = true)  
>> 必须在其他设置之前调用，除了Tween Specific Options  
直接设置初始值  
fromValue 开始的值  
setImmediately 如果是true，目标直接设置到初始位置，否则等带Tween开始  

> SetDelay(floa delay)  
>> 设置延迟  开始后再设置就无效了  
如果是Sequence并真正延迟， 只是在Sequence开头添加一个间隔 和PrependInterval相同    
`transform.DOMoveX(4, 1).SetDelay(1);`  

> SetSpeedBased(bool isSpeedBased = true)  
>> 使用速度进行Tween，如果要匀速，设置Ease.Linear  
如果开始了，没用 Sequence，也没用  
`transform.DOMoveX(4, 1).SetSpeedBased();`  

#### 注意

> 一些Tweener有特殊选项，具体取决于要Tween的类型  
如果有特定的选项，SetOptions方法能调用，否则不可用  
注意，通常通用方式创建补间时才可用 快捷方法中已经包含了相同的选项  
记住SetOptions必须在补间创建功能之后立即链接，否则它将不再可用    

#### Tween Specific Options

> Color tween => SetOptions(bool alphaOnly)  
仅仅tween alpha
`DOTween.To(()=> myColor, x=> myColor = x, new Color(1,1,1,0), 1).SetOptions(true);`  

> float tween => SetOptions(bool snapping)  
顺滑地使用整数  
`DOTween.To(()=> myFloat, x=> myFloat = x, 45, 1).SetOptions(true);`  

> Quaternion tween => SetOptions(bool useShortest360Route)  
默认true 旋转采用最短路径 且旋转角度不会超过360°  
FALSE 将充分考虑轮换  
如果Tween设置为FROM 则始终为FALSE  
`DOTween.To(()=> myQuaternion, x=> myQuaternion = x, new Vector3(0,180,0), 1).SetOptions(true);`  

> Rect tween => SetOptions(bool snapping)  
顺滑地使用整数  
`DOTween.To(()=> myRect, x=> myRect = x, new Rect(0,0,10,10), 1).SetOptions(true);`  

> String tween => SetOptions(bool richTextEnabled, ScrambleMode scrambleMode = ScrambleMode.None, string scrambleChars = null)  
>> richTextEnabled  
默认是true 正确解释富文本 否则被视为普通文本  
>> scramble  
随机顺序字符拼成字符串  
None 默认的  
All Uppercase Lowercase Numerals 使用Scrambling  
Custom 自定义  
>> scrambleChars  
自定义加扰的字符 使用尽可能多的字符,最少10个 DOTween使用快速加扰模式，使用更多字符可获得更好的结果  
`DOTween.To(()=> myString, x=> myString = x, "hello world", 1).SetOptions(true, ScrambleMode.All);`  

> Vector2/3/4 tween => SetOptions(AxisConstraint constraint, bool snapping)  
>> constraint  
只tween给定的轴  
>> snapping  
如果是true 顺滑地使用整数  
`DOTween.To(()=> myVector, x=> myVector = x, new Vector3(2,2,2), 1).SetOptions(AxisConstraint.Y, true);`  

> Vector3Array tween => SetOptions(bool snapping)  
`DOTween.ToArray(()=> myVector, x=> myVector = x, myEndValues, myDurations).SetOptions(true);`  

#### DOPath Specific Options

> Path tween => SetOptions(bool closePath, AxisConstraint lockPosition = AxisConstraint.None, AxisConstraint lockRotation = AxisConstraint.None)  
>> closePath  
如果True path自动封闭  
>> lockPosition   
`AxisConstrain.X | AxisConstraint.Y`  
>> lockRotation  
同上  
`transform.DOPath(path, 4f).SetOptions(true, AxisConstraint.X);`    

> Path tween => SetLookAt(Vector3 lookAtPosition/lookAtTarget/lookAhead, Vector3 forwardDirection, Vector3 up = Vector3.up)
>> lookAtPosition  位置  
>> lookAtTarget  Transform  
>> lookAhead  百分比 0 到 1  
默认向前看  
>> up  
定义向上方向  
`transform.DOPath(path, 4f).SetLookAt(new Vector3(2,1,3));`  
`transform.DOPath(path, 4f).SetLookAt(someOtherTransform);`  
`transform.DOPath(path, 4f).SetLookAt(0.01f);`  


### TweenParams

可以存储设置，交给多个Tweener  
可以创建新的，可以Clear已有的并重新设置  
使用SetAs方法  
```c#
// Store settings for an infinite looping tween with elastic ease
TweenParams tParms = new TweenParams().SetLoops(-1).SetEase(Ease.OutElastic);
// Apply them to a couple of tweens
transformA.DOMoveX(15, 1).SetAs(tParms);
transformB.DOMoveY(10, 1).SetAs(tParms);
```

#### 可以串联
`transform.DOMoveX(45, 1).SetDelay(2).SetEase(Ease.OutQuad).OnComplete(MyCallback);`  

## 控制 Tween

#### 有三种方法，有相同名字 除了快捷方法有些带有DO前缀

> DOTween 静态方法  
>> 默认返回整数，代表能操作的tween的数量  
方法有些带有All后缀，能操作所有  
方法可以传入SetId设置的Id  
`// Pauses all tweens`  
`DOTween.PauseAll();`  
`// Pauses all tweens that have "badoom" as an id`  
`DOTween.Pause("badoom");`  
`// Pauses all tweens that have someTransform as a target`  
`DOTween.Pause(someTransform);`  

> Tween 方法  
>> `myTween.Pause();`  

> 常规对象调用  
>> `transform.DOPause();`  

### Control 方法

#### 重要 要在动画结束后使用这些方法 必须禁用autoKill 否则Tween将在完成时自动终止

> CompleteAll/Complete(bool withCallbacks = false)  
>> 直接结束 到end位置  无限循环的tween无效  
withCallbacks 只适用于Sequence，如果是true，触发callbacks  

> FlipAll/Flip()  
>> 翻转向（如果向前移动，则向后，反之亦然）  

> GotoAll/Goto(float to, bool andPlay = false)  
>> 发送到指定位置  
>> to
>>> 到达的时间位置（如果高于整个补间持续时间，则补间将仅到达其结尾）  
>> andPlay  如果true，到达指定位置后Play 否则暂停  

> KillAll/Kill(bool complete = true, params object[] idsOrTargetsToExclude)  
>> 完成时直接Kill，除非使用SetAutoKill(false)阻止  
这个方法可以快速释放  

> PauseAll/Pause()  

> PlayAll/Play()  

> PlayBackwardsAll/PlayBackwards()  

> PlayForwardAll/PlayForward()  

> RestartAll/Restart(bool includeDelay = true, float changeDelayTo = -1)  
>> includeDelay  
>>> 如果是真，tween包含最终的延时  
>> changeDelayTo  
>>> 修改延时  

> RewindAll/Rewind(bool includeDelay = true)  

> SmoothRewindALl/SmoothRewind()  
“平滑后退”将补间动画设置到其起始位置（而不是跳转到其起始位置）  
跳过所有经过的循环（LoopType.Incremental除外），同时保持动画流畅  
如果调用仍在等待延迟发生的补间，则只需将延迟设置为0并暂停补间即可  
注意：平滑倒带的补间将改变其播放方向  

> TogglePauseAll/TogglePause()  
Plays the tween if it was paused, pauses it if it was playing.

### 特殊 Control 方法

> 通用方法
>> ForceInit()  
如果想从tweener获取初始化之前不可用的数据 如PathLength  

> 特殊  
>> GotoWaypoint(int waypointIndex, bool andPlay = false)  
只有Path tweener能用  并且必须是Linear  
将Path tweener发送到指定的 index下单path  
注意 调用此方法后，lookAt方向可能会错误  
并且可能需要手动设置 因为它依赖于平滑的路径移动 并且不适用于包含剧烈的方向变化的跳转  
>>> waypointIndex  
>>>> 跳到第导航点 如果大于最后一个，就算最后一个  
>>> andPlay  
>>>> 如果是真 到达后play 否则pause  
>> `myPathTween.GotoWaypoint(2);`  

## 从 Tween 取出数据

### 静态方法

> 小心垃圾增长  
>> static List<Tween> PausedTweens()   
>> static List<Tween> PlayingTweens()  
>> static List<Tween> TweensById(object id, bool playingOnly = false)   
>>> 如果true 仅返回playing状态的Tween  
false 全部返回  
>> static List<Tween> TweensByTarget(object target, bool playingOnly = false)  
注意：Tween的目标是快捷方法创建时自动设置的，而不是在使用通用方法时设置的  
注意：DOTweenAnimation视觉编辑器会将其gameObject分配为目标  
>> static bool IsTweening(object idOrTarget, bool alsoCheckIfPlaying = false)  
默认false 此时只要目标处于活动状态，就返回真  
否则必须 还在播放状态 才返回真  
`transform.DOMoveX(45, 1); // transform is automatically added as the tween target`  
`DOTween.IsTweening(transform); // Returns TRUE`  
>> static int TotalPlayingTweens()  
返回active和playing状态的Tween 即使在delay播放状态 也被认为是playing  
`int totalPlaying = DOTween.TotalPlayingTweens();`  

### Instance 方法 Tween Tweener Sequence

> float fullPosition  
Gets Sets 时间位置 包括循环 不包括延时 这不是方法  
> int CompletedLoops()  
`int completedLoops = myTween.CompletedLoops();`  
> float Delay()  
`float eventualDelay = myTween.Delay();`  
> float Duration(bool includeLoops = true)  
delays excluded, loops included if includeLoops is TRUE  
注意：使用SpeedBased，Tween开始时将重新计算  
`float loopCycleDuration = myTween.Duration(false);`  
`float fullDuration = myTween.Duration();`  
> float Elapsed(bool includeLoops = true)  
返回已经过去的时间  
delays excluded, loops included if includeLoops is TRUE  
includeLoops 为true 返回每个循环以来的完整时间 否则当前循环周期的时间  
`float loopCycleElapsed = myTween.Elapsed(false);`  
`float fullElapsed = myTween.Elapsed();`  
> float ElapsedDirectionalPercentage()  
根据单个循环返回此补间的已用百分比 0到1  不包括延迟  
计算最终的向后Yoyo循环为1到0，而不是0到1  
> float ElapsedPercentage(bool includeLoops = true)  
includeLoops 为true 包括循环 延迟始终不包含  
`float loopCycleElapsedPerc = myTween.ElapsedPercentage(false);`  
`float fullElapsedPerc = myTween.ElapsedPercentage();`  
> bool IsActive()  
`bool isActive = myTween.IsActive();`  
> bool IsBackwards()  
`bool isBackwards = myTween.IsBackwards();`  
> bool IsComplete()  
`bool isComplete = myTween.IsComplete();`  
>> bool IsInitialized()  
`bool isInitialized = myTween.IsInitialized();`  
>> bool IsPlaying()  
`bool isPlaying = myTween.IsPlaying();`  
>> int Loops()  
已经过的循环次数  
`int totLoops = myTween.Loops();`  

### Instance 方法 Path tweens

>> Vector3 PathGetPoint(float pathPercentage)  
根据给定的路径百分比返回路径上的一个点  
如果 不是Path Tween 无效 未初始化 返回Vector3.zero  
Tween开始后 pro版本的TweenEditor创建后 会直接初始化 否则  
通过调用ForceInit强制初始化路径  
`Vector3 myPathMidPoint = myTween.PathGetPoint(0.5f);`  

>> Vector3[] PathGetDrawPoints(int subdivisionsXSegment = 10)  
返回可用于绘制路径的点数组  
如果 不是PathTween 无效 未初始化 返回NULL  
注意：新的数组  
Tween开始后 pro版本的TweenEditor创建后 会直接初始化 否则  
通过调用ForceInit强制初始化路径  
`Vector3[] myPathDrawPoints = myTween.PathGetDrawPoints();`  

> float pathLength()  
返回路径长度  
如果 不是Path Tween 无效 未初始化 返回-1  
Tween开始后 pro版本的TweenEditor创建后 会直接初始化 否则  
通过调用ForceInit强制初始化路径  
`float myPathLength = myTween.PathLength();`  

## 协程

### YieldInstructions 可以放在协程中
### CustomYieldInstruction 自定义的

> WaitForCompletion()  
>> `Tween 被杀死  完成`  
`IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1);`  
`  yield return myTween.WaitForCompletion();`  
`  // This log will happen after the tween has completed`  
`  Debug.Log("Tween completed!");`  
`}`  

> WaitForElapsedLoops(int elapsedLoops)  
Tween 被杀死  经历了给定的循环次数  
>> `IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1).SetLoops(4);`  
`  yield return myTween.WaitForElapsedLoops(2);`  
`  // This log will happen after the 2nd loop has finished`  
`  Debug.Log("Tween has looped twice!");`  
`}`  

> WaitForKill()  
Tween 被杀死  
>> `IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1);`  
`  yield return myTween.WaitForCompletion();`  
`  // This log will happen after the tween has been killed`  
`  Debug.Log("Tween killed!");`  
`}`  

> WaitForPosition(float position)  
Tween 被杀死 已到达指定位置（包括循环，不包括延迟）  
>> `IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1);`  
`  yield return myTween.WaitForPosition(0.3f);`  
`  // This log will happen after the tween has played for 0.3 seconds`  
`  Debug.Log("Tween has played for 0.3 seconds!");`  
`}`  

> WaitForRewind()  
Tween 被杀死 倒回    
>> `IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1).SetAutoKill(false).OnComplete(myTween.Rewind);`  
`  yield return myTween.WaitForRewind();`  
`  // This log will happen when the tween has been rewinded`  
`  Debug.Log("Tween rewinded!");`  
`}`  

> WaitForStart()  
Tween 被杀死  
started (meaning when the tween is set in a playing state the first time, after any eventual delay`  
>> `IEnumerator SomeCoroutine()`  
`{`  
`  Tween myTween = transform.DOMoveX(45, 1);`  
`  yield return myTween.WaitForStart();`  
`  // This log will happen when the tween starts`  
`  Debug.Log("Tween started!");`  
`}`  

## 其他方法

**DOTween 静态方法**

> static DOTween.Clear(bool destroy = false)  
杀死所有Tween 清除对象池 重置最大容量到default  
destroy true时，也会销毁DOTween的gameObject并重置初始化内容 下次需要重新初始化  

> static DOTween.ClearCachedTweens()  
清理对象池  

> static DOTween.Validate()  
> 验证所有活动补间并删除最终无效的补间 通常是因为其目标已被破坏  
这是一个昂贵的操作 谨慎使用
其实完全不需要使用它，特别是在安全模式为ON的情况下  

> static DOTween.ManualUpdate(float delatTIme, float unscaledDeltaTime)  
更新所有UpdateType.Manual的Tween  

### Instance methods  Tween Tweener Sequence

> ChangeEndValue(newEndValue, float duration = -1, bool snapStartValue = false)  
更改Tween的最终值并倒回（不暂停）  
对Sequence内部的Tweeners无效  
注意：不适用于仅将一个值从一个点动画化为另一个点（例如DOLookAt）  
注意：对于单个轴的快捷方法 DOMoveX/Y/Z DOScaleX/YZ等  必须传递完整的Vector2/3/4  
>> duration  
>>> 如果大于0 ，一样修改duration  
>> snapStartValue  
>>> 如果是true，将当前值设置为初始值  

> ChangeStartValue(newStartValue, float duration = -1)  
>> 更改初始值并倒回 此外同上

> ChangeValues(newStartValue, newEndValue, float duration = -1)  
>> 更改初始值和最终值并倒回 此外同上  

## 编辑器方法

在编辑器预览Tween

> sattic DOTweenEditorPreview.PrepareTweenForPreview(bool clearCallbacks = true, bool preventAutoKill = true, bool andPlay = true)  
要设置UpdateType为Manual  
clearCallbacks  建议true  
preventAutoKill 如果是true 将阻止自动销毁  
andPlay true则立刻play  

> static DOTweenEditorPreview.Start(Action onPreviewUpdate = null)  
在编辑器中启动循环 在playMode期间无效  
在调用此方法之前，必须通过DOTweenEditorPreview.PrepareTweenForPreview将Tween添加到预览  

> static DOTweenEditorPreview.Stop()  
停止更新 释放所有回调  

## 虚方法  

不可用于Sequence  

> static Tweener DOVirtual.Float(float from, flaot to, float duration, TweenCalback<float> onVirtualUpdate)  
Tweens a virtual float.  
You can add regular settings to the generated tween, but do not use SetUpdate or you will overwrite the onVirtualUpdate parameter.  
**from** The value to start from.  
**to** The value to tween to.  
**duration** The duration of the tween.  
**onVirtualUpdate** A callback which must accept a parameter of type float, called at each update.  

> static Tweener DOVirtual.EaseValue(float from, float to, float lifetimePercentage, Ease easeType \ AnimationCurve animCurve)  
Returns a value based on the given ease and lifetime percentage (0 to 1).  
Comes with various overloads to allow an AnimationCurve ease or custom overshoot/period/amplitude.  
**from** The value to start from when lifetimePercentage is 0.  
**to** The value to reach when lifetimePercentage is 1.  
**lifetimePercentage** The time percentage (0 to 1) at which the value should be taken.  
**easeType** The type of ease.  

> static Tween DOVirtual.DelayedCall(float delay, TweenCallback callback, bool ignoreTimeScale = true)  
Fires the given callback after the given time.  
Returns a Tween so you can eventually store it and pause/kill/etc it.  
**delay** Callback delay.  
**callback** Callback to fire when the delay has expired.  
**ignoreTimeScale** If TRUE (default) ignores Unity's timeScale.  
`// Example 1: calling another method after 1 second`  
`DOVritual.DelayedCall(1, MyOtherMethodName);`  
`// Example 2: using a lambda to throw a log after 1 second`  
`DOVirtual.DelayedCall(1, ()=> Debug.Log("Hello world");`  

---