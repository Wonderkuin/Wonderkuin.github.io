# Vs Code 权威指南

---

```
unity vscode配置
0 隐藏meta
    "files.exclude" : {
        "**/*.meta" : true
    },
1 智能提示
Assembly-CSharp.csproj看
    <TargetFrameworkVersion>v4.7.1</TargetFrameworkVersion>
下载 对应 .net framework
2 UnityEngine.UI
先用vs studio打开工程，生成csproj
再用vscode打开工程
3 调试
Debugger for Unity
先运行项目再创建launch.json
```

```
快捷键

Ctrl Shift P
打开命令面板
	:open settings 打开设置

Ctrl ,
快速打开设置

Ctrl +/-
字体缩放

```

# 第一章 如何学习vscode

```
会用 Google
会用 Stack Overflow

Visual Studio Code 官网文档
Github仓库有Issues 和 Wiki

学会提问，清晰地描述问题
```

# 第二章 vscode简介

```
优点：
跨平台
智能提示
代码调试
内置Git支持

性能
插件
开源
```

# 第三章 核心组件

```
Electron 原名Atom Shell 由Github开发
以Nodejs作为运行时，以Chromium作为渲染引擎
使开发者可以使用HTML CSS Js开发跨平台桌面GUI

AtomShell的名字让人误解，Atom和vsCode没关系
Skype GithubDesktop Slack都用这个框架


Monaco Editor
vscode最核心组件 是一个基于浏览器的代码编辑器
包含 智能提示 代码验证 语法高亮 代码片段 代码格式化
代码跳转 键盘快捷键 文件比较

Gitee Web IDE， Eclipse Che ， Eclipse Theia都用这个


TypeScript
设计目标就是开发大型应用，解决开发者在使用js时的痛点
功能：类型批注 类型推断 类型擦除 接口 枚举 Mixin
泛型编程 命名空间 元组

由于TS 带来了类型支持，IDE轻松智能提示
编写代码时，通过TSLint ESLint进行静态检查
编译时 可以通过TS编译器进行类型检查
持续集成或者持续部署时，可以轻松进行代码检查，提取发现错误

声明文件 .d.ts 就是头文件


语言服务器 Langeuage Server
提供了自动补全 定义跳转 代码格式化等 编程语言相关功能

LSP Language Server Protocol是 编辑器/IDE和 LSP的一种协议
通过Json-Rpc传输消息，可以让不同IDE方便嵌入各种编程语言

问题：
语言服务器通常由原生编程语言实现，但vscode是nodejs运行时的

语言功能经常消耗资源密集，比如，为了验证一个文件，LSP
需要解析大量文件，生成抽象语法树 ，进行静态分析
这些操作消耗大量CPU和内存，vscode不想影响性能

集成不同语言工具和不同的编辑器需要大量工作。
语言工具的角度：需要适应代码编辑器的不同API
代码编辑器角度：无法从不同语言工具获取统一API
在M个编辑器中支持N中语言，工作量是MxN :(

为了解决这些问题，微软设计了LSP，把语言工具和代码编辑器的通信标准化，
语言服务器可以用任何语言编写，运行在自己的进程中， 不影响代码编辑器性能
通过LSP与代码编辑器进行通信。
工作量变成M+N
LSP已经标准化，众多主流语言都有了LSP支持。


Debug Adapter Protocol
和LSP异曲同工，DAP是一个居于Json的协议，抽象了开发工具与调试工具的通信
需求：
各种类型的断点
变量查看
多线程以及多线程支持
调用堆栈
表达式监控
调试控制台

是一个进行开源开发的协议


xterm.js
集成终端Integrated Terminal
基于一个被广泛使用的开源项目Xterm.js进行开发的
是一个使用TypeScript开发的前端组件，把完整的终端功能带入了浏览器
主要功能：
终端应用，bash vim tmux
高性能 速度快，支持GPU加速渲染
丰富的Unicode支持，支持Emoji表情符合，输入法编辑器 CJK字符
自包含：不需要额外的依赖
可访问性 accessibility 支持屏幕阅读器
其他功能：链接支持 主题 插件 完整的API文档

并不是一个直接下载便可以使用的终端应用
它是一个前端组件，可以与bash这样的进程连接
让用户通过Xterm.js交互

支持主流浏览器
```

# 第四章 安装与配置

```
略
```

# 第五章 快速入门

```
设置

用户设置 User Settings
	全局范围的设置，会应用到所有vscode
工作区设置 Workspace Settings
	设置被保存至相应的工作区中，只会对相应的工作区生效
	工作区设置会覆盖用户设置

通过关键词搜索设置
	format格式化
	wordwrap自动换行

默认使用json或者ui配置
	"workbench.settings.editor": "ui"
UI能直接打开Json编辑

系统Json在 %APPDATA%\Code\User\settings.json
工作区json在根目录.vscode文件夹下面

特定语言的设置
Ctrl+Shift+P打开命令面板
Preferences:Configure Language Specific Settings

设置与安全
指定可执行程序的路径，只能通过用户设置进行定义
不能通过工作区设置进行定义
git:path
terminal.integrated.shell.linux
terminal.integrated.shellArgs.linux
terminal.integrated.shell.osx
terminal.integrated.shellArgs.osx
terminal.integrated.shell.windows
terminal.integrated.shellArgs.windows
terminal.external.windowsExec
terminal.external.osxExec
terminal.external.linuxExec


常用设置
控制编辑器自动格式化粘贴的内容
"editor.formatOnPaste" : true
在保存文件后进行代码格式化
"edit006Fr.formatOnSave" : true
改变字体大小
//编辑区域
"editor.fontSize" : 18,
//集成终端
"terminal.integrated.fontSize" : 14,
//输出窗口
"[Log]" : {
	"editor.fontSize" : 15
}

```

---