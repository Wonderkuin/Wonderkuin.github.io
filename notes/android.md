### [<主页](https://www.wangdekui.com/)

# Android

## 简介

#### 四大组件
Activity 活动 看到的东西都放在活动中  
Service 服务 退出应用也可运行  
BroadcastReceiver 广播接收器 允许应用接收来自各处的广播消息  
ContentProvider 内容提供器 应用程序之间共享数据，读取联系人  

#### 其他
系统自带控件 自定义控件  
SQLite数据库 轻量级 速度快 嵌入式 关系型 标准SQL 封装好的API  
多媒体 音乐 视频 录音 拍照 闹铃  
地理位置 GPS定位  

#### 环境配置
手动安装jdk  
安装好android studio，配置好国内代理  
自动安装sdk ndk gradle  

#### 新建项目
Application name: test  安装后显示的名称  
Company Domain: geb.com  
Package name: com.geb.test或者自定义 Android系统根据包名区分不同应用程序  
Minimum SDK: 最低api  
Templete: 空模板  
Android Virtual Device  

## 项目大概

### 配置文件 AndroidManifest
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.test.quanxian">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Quanxian">
        <activity android:name=".MainActivity"
            android:label="定义标题">
            <intent-filter>
                <!--注册主活动-->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### 活动 Activity

AppCompatActivity 最低向下兼容到 Android 2.1  

```java
package com.test.quanxian;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 设置View
        setContentView(R.layout.activity_main);
    }
}
```

### 布局

android.support  
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"/>
</RelativeLayout>
```

androidx  
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### 资源 R
R.string  
R.drawable  
R.mipmap  
R.layout  

string  
```xml
<resources>
    <string name="app_name">HelloWorld</string>
</resources>
```

id  
```xml
<Button android:id="@+id/button_1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Button 1"
    />
```

#### 使用按钮

```java
Button button1 = (Button)findViewById((R.id.button_1));
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Toast.makeText(MainActivity.this, "按钮1", Toast.LENGTH_SHORT).show();
    }
});
```

#### Menu菜单

应用程序三个点，下拉的菜单  

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="Add"/>
    <item
        android:id="@+id/remove_item"
        android:title="Remove"/>
</menu>
```

```java
//显示，返回false就不显示
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main, menu);
    return true;
}

//消息回调
@Override
public boolean onOptionsItemSelected(@NonNull MenuItem item) {
    switch (item.getItemId())
    {
        case R.id.add_item:
            Toast.makeText(this, "增加", Toast.LENGTH_SHORT).show();
            break;
        case R.id.remove_item:
            finish();//销毁活动
            break;
        default:
    }
    return true;
}
```

### Intent 在活动间跳转
启动活动  
启动服务  
发送广播  

#### 显示Intent
栈 此时结束SecondActivity，回到MainActivity  

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
startActivity(intent);
```

#### 隐式Intent
不指定某个Activity，先设置intent-filter  
action和category同时匹配，则相应Intent  
```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.test.quanxian.ACTION_START"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.MY_CATEGORY"/>
    </intent-filter>
</activity>
```

```java
Intent intent = new Intent("com.test.quanxian.ACTION_START");
intent.addCategory("android.intent.category.MY_CATEGORY");
startActivity(intent);
```

#### 隐式Intent跳转给别的应用打开
```java
//系统内置动作
//android.intent.action.VIEW
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://www.baidu.com"));
startActivity(intent);
```

#### data标签，表明能响应什么数据
android:scheme 协议 如http  
android:host 主机名 如www.baidu.com  
android:port 端口 紧跟在主机名后  
android:path 端口后的部分 域名后的内容  
android:mimeType 可以处理的数据类型 允许通配符  

只需要指定 android:scheme 为http，就可以相应所有http协议的内容了，指定过多，限制更严格  

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:scheme="http"/>
</intent-filter>
```

http外 geo地理位置 tel拨打电话  
```java
Intent intent = new Intent(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:10000"));
startActivity(intent);
```

#### Intent 存储数据

```java
String data = "Hello";
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
intent.putExtra("extra_data", data);
startActivity(intent);
```

```java
// onCreate
Intent intent = getIntent();
String data = intent.getStringExtra("extra_data");
Log.d("SecondActivity", data);
```

#### 返回数据给上一个活动

```java
// 从Main到Second
// 请求码要唯一值
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
startActivityForResult(intent, 1);

// 接受返回结果
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    switch(requestCode) {
        case 1:
            if (resultCode == RESULT_OK) {
                String returnedData = data.getStringExtra("data_return");
                Log.d("Main", returnedData);
            }
            break;
        default:
    }
}
```

构建传递数据的Intent  
调用setResult方法  
```java
//onclick 点击按钮返回
Intent intent = new Intenet();
intent.putExtra("data_return", "Hello Main");
setResult(RESULT_OK, intent);//RESULT_CANCELED
finish();
```

### Activity的生命周期

运行 暂停 停止 销毁  
onCreate 首次创建  
onStart 由不可见变为可见  
onResume 位于栈顶，准备和用户交互时调用  
onPause 系统转到其他Activity，此时应该 迅速 释放资源保存数据  
onStop 不可见时，和pause的区别为：对话框不执行onStop  
onRestart 由停止变为运行前调用  
onDestroy 销毁  


## [<主页](https://www.wangdekui.com/)