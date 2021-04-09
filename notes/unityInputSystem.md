### [<主页](/index.html)

# InputSystem

### 简介 安装

```markdown
UnityEngine.InputSystem包

Unity2019.1+ .Net4+

Edit>Project Settings>Player
    Active Input Handling
```

### 快速入门

```c#
// 直接从设备输入
var gamepad = Gamepad.current;
if (gamepad != null) {
    if (gamepad.rightTrigger.wasPressedThisFrame)
        ;// todo
    Vector2 move = gamepad.leftStick.ReadValue();
}
//Keyboard.current
//Mouse.current
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
InputSystemUIInputModule


```

---

## [<主页](/index.html)