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

# Android UI

## UI基础

dp 密度无关像素  
sp 缩放像素，字体，文字大小  

### layout 属性

id  

layout_height:wrap_content  
layout_width:match_parent  
layout_margin  
padding  
gravity:left center right 对于本身而言，内容显示在控件的什么位置 默认在左  
orientation:vertical horizantal  
layout_weight:0 1 4 可以设置layout_height:0dp，用weight控制高度  
layout_gravity 和gravity不同，设置该元素在父对象的位置

文本视图TextView  
text  
textAppearance  
textSize  

RelativeLayout属性  
与父视图对齐，把视图固定在ReltativeLayout的边角 值为true 或者 false  
layout_alignParentTop  
layout_alignParentBottom  
layout_alignParentRight  
layout_alignParentLeft  
放置视图位于水平或者竖直中心，或者直接放在中央 值为true 或者 false  
layout_centerHorizontal  
layout_centerVertical  
layout_centerInParent  
与另一个视图对齐  
layout_alignTop  
layout_alignBottom  
layout_alignRight  
layout_alignLeft  
视图的所有边缘都与指定的视图对齐，进行视图的重叠，完全匹配 值为id  
layout_alignBaseline
相对另一个视图放置视图 值为id  
layout_above  
layout_below  
layout_leftOf  
layout_rightOf  

GridLayout示例  
父 columnCount="3"  
子1 layout_column="1" layout_row="1"  
子2 layout_column="1" layout_row="2"  
子3 layout_column="2" layout_row="3"  
子4 layout_column="2" layout_row="3"  

Space视图 空 占位  
layout_width="58dp"  
layout_height="1dp"  
layout_column="0"  
layout_gravity="fill_horizontal"  
layout_row="0"  

#### layout 类型

FrameLayout 不排布子视图  
TableLayout 展示表格数据，每一行包含在TableRow中，不能指定layout_width    
LinearLayout 线性布局  
RelativeLayout 相对布局 创建复杂UI 注意：不能有循环依赖  
GridLayout  网格布局，比TableLayout性能好，可以拖放到布局编辑器设计  


### 显示列表 ListView

```xml
<ListView android:layout_width="match_parent"
    android:layout_height="match_parent" android:id="@+id/list">
</ListView>
```

一个ListActivity将绑定到一个含有ListView的默认视图。  
不必在活动的onCreate中调用setContentView，默认已经设置为ListView  
内容为何？  
默认的android.R.layout.simple_list_item_1  
默认的android.R.layout.two_line_list_item  

特殊属性：android:entries 不需要编程填充列表，使用静态值  

自定义布局  
```xml
<!--time_row.xml-->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:andoird="http://schemas.android.com/apk/res/android"
    android:id="@+id/time_row" android:orientation="hoziaontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center" android:paddingLeft="10dp"
    android:paddingRight="10dp" android:paddingBottom="20dp"
    android:paddingTop="20dp">
    <TextView android:id="@+id/lap_name"
        android:layout_height="wrap_content"
        android:text="Lap 1" android:layout_weight="1"
        android:layout_width="0dp"/>
    <TextView android:id="@+id/lap_time"
        android:layout_height="wrap_content"
        android:text="00:00:00" android:layout_weight="1"
        android:layout_width="0dp" android:gravity="right"/>
</LinearLayout>
```

动态生成列表，需要建立一个列表适配器 ListAdapter  
Adapter类被用于往UI的视图上绑定数据  
需要重写 getView方法  
Loader类 往列表适配器中加载数据  

```java
public class TiemListAdapter extends ArrayAdapter<long> {
    public TimeListAdapter(Content content, int textViewResourceId) {
        super(context, textViewResourceId);
    }
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View view = convertView;
        if (view == null) {
            View = LayoutInflater.from(getContext()).inflate(R.layout.time_row, null);
        }
        long time = getItem(position);
        TextView name = (TextView)view.findViewById(R.id.lap_name);
        String taskString = getContext().getResources().getString(R.id.task_name);
        TextView lapTime = (TextView)view.findViewById(R.id.lap_time);
        lapTime.setText(DateUtils.formatElapsedTime(time));
        return view;
    }
}
```

```java
setContentView(R.layout.main);
TextView counter = (TextView)findViewById(R.id.counter);
counter.setText(DateUtils.formatElapsedTime(0));
if (mTimeListAdapter == null)
    mTimeListAdapter = new TimeListAdapter(this, 0);
ListView list = (ListView)findViewById(R.id.time_list);
list.setAdapter(mTimeListAdapter);
```

#### Activity和UI

Activity代表用户界面，所有与用户有关的交互都通过活动发生  
onPause方法里面，应该保存用户数据  
从竖屏变为横屏，会创建新的活动  
onSaveInstanceState在活动被销毁并且可能被重新建立时调用  
可以保存列表滚动位置  

更新UI需要在主线程或者ui线程  
Activity.runOnUiThread  
View.post View.postDelayed View.postInvalidate  
AsyncTask  
消息处理程序  


```java
@Override
protected void onSaveInstanceState(Bundle outState) {
    ListView list = (ListView) findViewById(R.id.time_list);
    int pos = list.getFirstVisiblePosition();
    outState.putInt("first_position", pos);
    super.onSaveInstanceState(outState);
}
```

### 防止应用程序 无响应

安卓系统的回调函数都是由主线程完成的，包括活动和服务回调函数，时间处理程序，按键监听程序等  
不要在这些回调函数中执行任何阻塞操作。请使用AsyncTask  

#### Handler 和消息队列
运行长时间任务后，通知UI刷新  
```java
private Handler mHandler = new Handler() {
    public void handleMessage(Message msg) {
        long current = System.currentTimeMillis();
        mTime += current - mStart;
        mStart = current;
        TextView counter = (TextView)TimeTrackerActivity.this.findViewById(R.id.counter);
        counter.setText(DateUtils.formatElapsedTime(mTime/1000));
        mHandler.sendEmptyMessageDelayed(0, 250);
    };
};

private void startTimer() {
    mStart = System.currentTimeMillis();
    mHandler.removeMessage(0);
    mHandler.sendEmptyMessage(0);
}

private void stopTimer() {
    mHandler.removeMessage(0);
}

private void resetTimer() {
    stopTimer();
    if (mTimeListAdapter != null)
        mTimeListAdapter.add(mTime/1000);
    mTime = 0;
}

private boolean isTimerRunning() {
    return mHandler.hasMessages(0);
}
```

#### AsyncTask

一次性任务比较常用  

```java
private class DownloadFilesTask extends Asynctask<URL, Integer, Long> {
    protected Long doInBackground(URL... urls) {
        while(true) {
            // Do some work
            publishProgress((int)((i / (float)count)*100));
        }
        return result;
    }
    protected void onProgressUpdate(Integer... progress) {
        setProgressPercent(progress[0]);
    }
    protected void onPostExecute(Long result) {
        showDialog("Result is " + result);
    }
}
```

#### 资源限定符

朝向 屏幕尺寸 屏幕分辨率的布局限定符  

屏幕朝向 Screenorientation port纵向 land横向  
屏幕像素密度 Screen pixel density ldpi mdpi hdpi xhdpi nodpi  
屏幕尺寸 Screen size small normal large xlarge  
最小宽度 可用宽度 可用高度 sw320dp sw720dp w720dp h720dp  
API版本 v6 v14  
等等  

记住  
在布局中始终使用dp  
尽可能使用wrap_content和match_parent  
为每个密度提供可选的图像资源，确保图像在所有的屏幕尺寸上都合适  
为任何伸缩后会发生扭曲的图像使用9-patch补丁图形  

#### TOAST
可以setGravity改变默认展示位置  
可以重写toast默认布局，创建自定义toast  
```java
Context context = getApplicationContext();
CharSequence text = "toast";
int duration = Toast.LENGTH_SHORT;
Toast toast = Toast.makeText(context, text, duration);
toast.show();
```

#### 状态栏通知
状态栏展示图标  
文字  
标题和消息，可选  
PendingIntent用户点击时触发  
```java
int icon = R.drawable.icon;
CharSequence ticker = "Hello World!";
long now = System.currentTimeMillis();
Notification notification = new Notification(icon, ticker, now);

NotificationManager nm = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
Context context = getApplicationContext();
CharSequence message = "Hello World!";
Intent intent = new Intent(this, Example.class);
String title = "Hello World!";
String message = "This is a message.";
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
notification.setLatestEventInfo(context, title, message, pendingIntent);
nm.notify(ID, notification);
```

#### 对话框
AlertDialog 包含Accept和Cancel  
ProgressDialog进度条  
继承DialogFragment类并实现onCreateView或onCreateDialog  

```java
public class ConfirmClearDialogFragment extends DialogFragment {
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        return new AlertDialog.Builder(getActivity())
            .setTitle(R.string.confirm_clear_all_title)
            .setMessage(R.string.confirm_clear_all_message)
            //.setPositiveButton(R.string.ok, null)
            // 点击事件，关闭对话框并清空列表适配器
            .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    dialog.dismiss();
                    mAdapter.clear();
                }
            })
            .setNegativeButton(R.string.cancel, null)//消除对话框
            .create();
    }
}
```

```java
FragmentManager fm = getSupportFragmentManager();
if (fm.findFragmentByTag("dialog") == null) {
    ConfirmClearDialogFragment frag = ConfirmClearDialogFragment.newInstance(mTimeListAdapter);
    frag.show(fm, "dialog");
}
```

#### 事件

点击事件略  

```java
View view = findViewById(R.id.my_view);
view.setOnLongClickListener(new View.OnLongClickListener() {
    @Override
    public boolean onLongClick(View v) {
        Toast.makeText(TimeTrackerActivity.this, "Long pressed!", Toast.LENGTH_LONG).show();
        return false;//返回真会停止传递事件
    }
})
```
键盘支持  
聚焦Focus Event  
关键Key Events  

#### Meun菜单

用户不一定看得到菜单里面的功能  

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/clear_all"
        android:title="@string/clear_all"/>
</menu>
```

见上面Menu菜单  
onOptionsItemSelected中可以创建弹窗询问是否执行操作  
```java
FragmentManager fm = GetSupportFragmentManager();
if (fm.findFramentByTag("dialog") == null) {
    ConfirmClearDialogFragment frag = ConfirmClearDialogFragment.newInstance(mTimeListAdapter);
    frag.show(fm, "dialog");
}
```

#### 上下文菜单

相当于Windows桌面右键，为视图设置上下文菜单的Linstener并且实现onCreateContextMenu方法  
并且实现onContextItemSelected方法处理用户的选择，通过View.setOnCreateContextMenuListener或者  
Activity.registerForContextMenu注册上下文菜单linstener  

```java
TextViwe tv = (TextView)findViewById(R.id.text);
tv.setOnCreateContextMenuListener(this);
// Alternatively, use-Activity.registerForContextMenu(tv);
@Override
public void onCreateContextMenu(ContextMenu menu, View v, ContextMenuInfo menuInfo) {
    super.onCreateContextMenu(menu, v, menuInfo);
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.context_menu, menu);
}
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch(item.getItemId()) {
        case R.id.item1:
            return true;
        case R.id.item2:
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

#### 服务 Service 执行后台任务

```java
public class TimerService extends Service {
    private static final String TAG = "TimerService";
    public static int TIMER_NOTIFICATION = 0;
    private NotificationManager nNM = null;
    private Notification mNotification = null;
    private long mStart = 0;
    private long mTime = 0;
    public class LocalBinder extends Binder {
        TimerService getService() {
            return TimerService.this;
        }
    }
    private final IBinder mBinder = new LocalBinder();
    private Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            long current = System.currentTimeMillis();
            mTime += current - mStart;
            mStart = current;
            updateTime(mTime);
            mHandler.sendEmptyMessageDelayed(0, 250);
        };
    };
    @Override
    public void onCreate() {
        Log.i(TAG, "onCrate");
        nNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // Show notification when we start the timer
        showNotification();
        mStart = System.currentTimeMillis();
        // Only a single message type, 0
        mHandler.removeMessage(0);
        mHandler.sendEmptyMessage(0);
        // Keep restarting until we stop the service
        return START_STICKY;
    }
    @Override
    public void onDestroy() {
        // Cancel the ongoing notification
        nNM.cancel(TIMER_NOTIFICATION);
        mHandler.removeMessages(0);
    }
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    public void stop() {
        mHandler.removeMessage(0);
        stopSelf();
        mNM.cancel(TIMER_NOTIFICATION);
    }
    public boolean isStopped() {
        return !mHandler.hasMessages(0);
    }
    public void reset() {
        stop();
        timerStopped(mTime);
        mTime = 0;
    }
    private void updateTime(long time) {
        Intent intent = new Intent(TimeTrackerActivity.ACTION_TIME_UPDATE);
        intent.putExtra("time", time);
        sendBroadcast(intent);
    }
    private void timerStopped(long time) {
        // Broadcast timer stopped
        Intent intent = new Intent(TimeTrackerActivity.ACTION_TIMER_FINISHED);
        intent.putExtra("time", time);
        sendBroadcast(intent);
    }

    private void updateNotification(long time) {
        String title = getResources().getString(R.string.running_timer_notification_title);
        String message = DateUtils.formatElapsedTime(time/1000);
        Context context = getApplicationContext();
        Intent intent = new Intent(context, TimeTrackerActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);
        mNotification.setLatestEventInfo(context, title, message, pendingIntent);
        mNM.notify(TIMER_NOTIFICATION, mNotification);
    }
}
```

```java
// TimeTrackerActivity onCreate
IntentFilter filter = new intentFilter();
filter.addAction(ACTION_TIME_UPDATE);
filter.addAction(ACTION_TIMER_FINISHED);
registerReceiver(mTimeReceiver, filter);
```

## 视图框架

#### TextView EditText
```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="mathch_parent"
    android:orientation="vertical">
    <TextView
        android:id="@+id/counter"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:padding="10dp"
        android:text="@string/sample_time"
        android:textAppearance="?android:attr/textApperanceLarge"
        android:textSize="50sp" />
    <Button
        android:id="@+id/start_stop"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="30dp"
        android:text="@string/start" />
    <TextView
        android:id="@+id/task_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:layout_weight="1" "填满整个画面"
        android:text="@string/task_name"
        android:textSize="20dp" />
    <EditText
        android:id="@+id/task_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:hint="@string/task_name" "预设文本，用户输入时消失"
        android:gravity="top|left"
        android:calendarViewShown="false"
        android:layout_margin="10dp"
        android:textSize="24dp" />
    <!--android:inputType="phone" 数字类型 -->
</LinearLayout>
```

#### 按钮
四态 normal hovered focused pressed  
布尔值按钮 下拉列表按钮Spinner  

```xml
<Button
    android:id="@+id/finished"
    android:layout_width="match_content"
    android:layout_height="wrap_content"
    android:text="@string/finished"
/>
```

#### ScrollView
FrameLayout的一个子类  
在ScrollView中不能使用ListView  
fillViewport非常常用  
```xml
<ScrollView
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:fillViewport="true"
    android:layout_weight="1" >
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:text="测试"/>
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Button"/>
    </LinearLayout>
</ScrollView>
```

#### ImageView

缩放类型  
center 中心位置显示，无缩放  
centerCrop 中心位置缩放图片，使x和y的尺寸都大于或者等于视图，同时保持图像的比例，超出视图大小的部分被切割掉  
centerInside 缩放图片并保持原有的图片宽高比，保持图片在视图里面。如果图片已经比视图小，则与center类型相同  
fitCenter 缩放图片，保持原有宽高比，使其在图片内部。只要有一个轴与视图一致，缩放结果将置于视图的中心  
fitStart 左上角，其余和fitCenter相同  
fitEnd 右下角，其余和fitCenter相同  
fitXY 对图片同时进行xy轴的缩放使其恰好符合视图的尺寸，不保持原有宽高比  
matrix 使用提供的矩阵缩放图片 setImageMatrix 可以进行旋转等变换  

```xml
<ImageView
    android:id="@+id/image"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="center"
    android:src="@drawable/icon"
/>
```

## [<主页](https://www.wangdekui.com/)