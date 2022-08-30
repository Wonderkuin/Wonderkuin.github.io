# InputSystem

---

### 简介 安装

```markdown
UnityEngine.InputSystem包

Unity2019.1+ .Net4+

Edit>Project Settings>Player
    Active Input Handling
```

### 快速入门

```c#
// 直接从设备输入 不建议
var gamepad = Gamepad.current;
if (gamepad != null) {
    if (gamepad.rightTrigger.wasPressedThisFrame)
        ;// todo
    Vector2 move = gamepad.leftStick.ReadValue();
}
//Keyboard.current
//Mouse.current
var mouse = Mouse.current;
if (mouse != null)
    look = mouse.delta.ReadValue();
```

### PlayerInput 和 PlayerInputManager

```markdown
PlayerInput 自动处理 启用 和 禁用
多个PlayerInput使用相同的Action 这些组件自动创建Action的私有副本

回调
public void OnMove(InputAction.CallbackContext context) {
    var value = context.ReadValue<Vector2>();
}

PlayerInputManager
可以设置多人游戏
```

### UI support

```markdown
InputSystemUIInputModule 替代 StandaloneInputModule

指针式输入
point leftClick rightClick middleClick scrollWheel

导航型输入
导航类型的输入基于从动作中读取move动作来控制当前选择
submit ISubmitHandler Cancel ICancelHandler

履带式输入
来自跟踪设备 XR控制器 HMD 类似于指针类型的输入
区别在于 世界空间设备的位置和方向源自光线
trackedDevicePosition trackedDeviceOrientation
需要添加TrackedDeviceRaycaster到具有UI Canvas组件的GameObject
TrackedDeviceRaycaster 和 GraphicRaycaster共存
```

### MultiplayerEventSystem

```markdown
如果要让多个本地播放器与不同的控制器共享一个屏幕，这非常有用，那么每个播放器都可以控制自己的UI实例。
为此，您需要使用输入系统的MultiplayerEventSystem组件替换Unity中的EventSystem组件。
```

### 调试器

```markdown
非常有用
Visualizers示例，实时监视各种InputSystem元素的状态
```

### 设置

```markdown
创建输入资产

更新模式
    固定 动态 手动

滤噪
    InputControl.noise

补偿屏幕方向
    ScreenOrientation.Portrait 保持不变
    ScreenOrientation.PortraitUpsideDown 180度
    ScreenOrientation.LandscapeLeft 90度
    ScreenOrientation.LandscapeRight 270度
    会影响 Gyroscope GravitySensor AttitudeSensor Accelerometer LinearAccelerationSensor

支持的设备
    Gamepad Keyboard Mouse等
    移动应用可能只支持触摸
    平台游戏可能只支持手柄
    跨平台可能支持手柄 键鼠，不需要XR设备

    如果为空，没有限制
    如果包含一个或者多个，仅仅使用这些设备

覆盖编辑器
    在编辑器中，可能要使用设置不支持的设备，输入调试器可以设置
```

### Actions

```markdown
InputActionAsset 资产 包含一个或多个 操作图 以及一些可选的控制方案
InputActionMap 动作的命名集合
InputAction 动作

动作用 InputBinding 引用它们收集的输入
该名称在Action所属的ActionMap中必须是唯一的 InputAction.actionMap
每个动作还有唯一的id InputAction.id
即使重命名动作 id也不变

每个操作图都有一个名称 InputActionMap.name
该名称在该map所属的asset中唯一 InputActionMap.asset
id也唯一 InputActionMap.id

创建动作
    .inputactions资产编辑器可以创建动作
    嵌入MonoBehaviour组件中
    从Json手动加载
    直接在代码中创建

!!! 必须手动启用和禁用 MonoBehaviour中的 Acitons 和 ActionMap

从Json加载动作
略

用代码创建动作
// Create free-standing Actions.
var lookAction = new InputAction("look", binding: "<Gamepad>/leftStick");
var moveAction = new InputAction("move", binding: "<Gamepad>/rightStick");

lookAction.AddBinding("<Mouse>/delta");
moveAction.AddCompositeBinding("Dpad")
    .With("Up", "<Keyboard>/w")
    .With("Down", "<Keyboard>/s")
    .With("Left", "<Keyboard>/a")
    .With("Right", "<Keyboard>/d");

// Create an Action Map with Actions.
var map = new InputActionMap("Gameplay");
var lookAction = map.AddAction("look");
lookAction.AddBinding("<Gamepad>/leftStick");

// Create an Action Asset.
var asset = ScriptableObject.CreateInstance<InputActionAsset>();
var gameplayMap = new InputActionMap("gameplay");
asset.AddActionMap(gameplayMap);
var lookAction = gameplayMap.AddAction("look", "<Gamepad>/leftStick");

使用动作
// Enable a single action.
lookAction.Enable();

// Enable an en entire action map.
gameplayActions.Enable();


监听

Map
actionMap.actionTriggered +=
    context=>{}
InputSystem
InputSystem.onActionChange +=
    (obj, change)=>{}
Action
void Update()
{
    var moveDirection = moveAction.ReadValue<Vector2>();
    position += moveDirection * moveSpeed * Time.deltaTime;
}
按钮
void Update()
{
    if (InputAction.triggered)
        ;
}
```

#### Input Action Assets

```markdown
.inputactions 纯json格式

编辑器操作
左上角选模式，常用的输入器
添加 Map 添加 Action
Listen可以监听，模式中存在的输入器的键

Action模式
    Value 值
    Button 按键
    Pass Through 传递
        比如Move 传递 Vector2，此时绑定案件，不再用banding模式
        而是用

Press Only 按一下
Pass 长按
PressAndRelease 按下True，抬起False
```

---