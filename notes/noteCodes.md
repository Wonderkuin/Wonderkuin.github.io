# C#语法记录

```c#
// IDisposable接口
public class ActiveSet : IDisposable {
	~ActiveSet() {
		Dispose();
	}

	public void Dispose() {
	}
}

// using 语法糖
using(var as = new ActiveSet()) { }

// 相当于
ActiveSet activeSet = null;
try {
	activeSet s = new ActiveSet();
} finally {
	if (activeSet != null) {
		mc.Dispose();
	}
}

//类似于c++ ostream 栈上自动close
```

```c#
// 只读list
internal List<int> m_list = new List<int>();

public ReadOnlyList<int> GetList {
	get{ return new ReadOnlyList<int>(m_list); }
}
```

```c#
// ? ??
int index = Source?.Index ?? 0;
```

```c#
// public new
public new void Func() {
	// 当基类和派生类都有Func时，派生类用new，屏蔽基类方法不会warning
}

// new public
new public class Clazz{
	// 嵌套类隐藏基类中同名的类，消除警告
}
```

```c#
// 静态构造函数 无法保证次序
public static class LSFSettingsManager
{
	public const string SETTINGS_NAME = "";
	static LSFSettingsManager()
	{
		LSFSettings settings = Resources.Load<LSFSettings>(SETTINGS_NAME);
	}
}
```

```c#
// 关于点乘的用法
Vector3 worldDeltaPosition = agent.nextPosition - transform.position;
// 将worldDeltaPosition映射到本地空间
float dx = Vector3.Dot(transform.right, worldDeltaPosition);
float dy = Vectro3.Dot(transform.forward, worldDeltaPosition);
// 得到本地空间的下一个位置变化增量
Vector2 deltaPosition = new Vector2(dx, dy);
```