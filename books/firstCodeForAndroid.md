# Android 第一行代码

---

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

#### 显式Intent
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
intent.setData(Uri.parse("http://..."));
startActivity(intent);
```

#### data标签，表明能响应什么数据
android:scheme 协议 如http  
android:host 主机名 网站名称
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

```java
Bitmap bitmap = Bitmap.createBitmap(100, 100, Bitmap.Config.ARGB_8888);
ImageView iv = (ImageView)findViewById(R.id.image);
iv.setImageBitmap(bitmap);
```

#### Dragable抽象接口

StateListDrawable选择第一个符合当前状态标准的项  

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
        android:drawable="@drawable/button_pressed" />
    <item android:state_focused="true"
        android:drawable="@drawable/button_focused" />
    <item android:state_hovered="true"
        android:drawable="@drawable/button_hovered" />
    <item android:drawable="@drawable/button_normal>
</selector>
```

其他：Canvas SurfaceView TextureView  

#### WebView
基于Webkit的HTML渲染引擎，支持V8 Javascript解释器  
需要INTERNET权限  
```xml
<WebView
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

```Java
WebView webView = (WebView)findViewById(R.id.webView);
webView.loadUrl("http://...");

// JavaScript Flash支持
WebSettings webSettings = webView.getSettings();
webSettings.setjavaScriptEnabled(true);
webSettings.setPluginState(WebSettings.PluginState.ON);

//缩放控制 拿捏缩放
webSettings.setSupportZoom(true);
webSettings.setBuildInZoomControls(true);

//重写URL加载
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        view.loadUrl(url);
        return true;//停止事件，阻止浏览器打开
    }
});
```

### 复用UI

```
<include>标签，包含另一个布局
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <include layout="@layout/sub_layout">    
</LinearLayout>
```

```
局部搜索  
View.findViewById  
整个View搜索  
Activity.findViewById  

<merge>标签 子布局
merge标签会被去掉  
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <ImageView />
    <TextView />
</merge>
```

```
ViewStub  
像<include>一样，但是不会扩张的，直到发送指定的请求才会扩张  
能加速UI的绘制速度  
```

```xml
<ViewStub
    android:id="@+id/view_stub"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inflatedId="@+id/sub"
    android:layout="@layout/sub"
/>
```

```java
ViewStub vs = (ViewStub) findViewById(R.id.view_stub);
vs.setVisibility(View.VISIBLE);
View v = vs.inflate();
```

#### 样式

```xml
<!--不好的写法-->
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textColor="#FF0000"
/>
```

```xml
<!--制作一个样式-->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="RedText">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:textColor">#FF0000</item>
    </style>
</resources>
```

使用样式  
```xml
<TextViwe
    style="@style/RedText"
    android:text="@string/hello"
/>
```

样式继承  
```xml
<stype name="GreenText" parent="@android:style/TextAppearance">
    <item name="android:textColor">#FF0000</item>
</style>
```

继承自己的样式，简便写法  
```xml
<style name="RedText.Small">
    <item name="android:textSize">8dip</item>
</style>
```

#### 主题

样式只会应用到一个视图上，不应用到子视图上  
主题可以在activity或者application上应用  
```xml
<activity
    android:name=".ExampleActivity"
    android:theme="@android:style/Theme.Holo"
/>
```

#### Fragment
class是一个Java类，也可以不写，在运行时添加fragment  

```xml
<FrameLayout xmlns:android="http:schemas.android.com/apk.res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment class="com.example.ExampleFragment"
        android:id="@+id/example"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

一个Fragment的生命周期与Activity类似，重写onCreateView  
```java
public class SimpleTextFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        TextView tv = new TextView(getActivity());
        tv.setText("Hello");
        return tv;
    }
}
```

类比onCreate  
onAttach第一次与活动联系时调用  
onCreate初始化，此时host活动可能还没完成onCreate  
onCreateView创建视图，fragment不一定需要UI组件  
onActivityCreated当host活动完成了onCreate调用后调用的  
类比onDestroy  
onDestroyView视图与fragment分离  
onDestroy不再调用  
onDetach从host活动卸下  

```java
public class SimpleTextFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.task_list, null);
    }
}
```

#### fragment 事务

更改UI中现有的fragment，需要使用事务，通过FragmentManager执行  
```java
FragmentManager fm = getFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
ExampleFragment fragment = new ExampleFragment();
ft.add(R.id.fragment_container, fragment);
ft.commit();

// 查找
//fm.findFragmentById(R.id.frag);
//fm.findFragmentByTag("tag");

//fragment 回退栈
ft.addToBackStack(null);//tasks a string name argument, not used here
ft.commit();

// 弹出最近的事务
//ft.popBackStackI();
```

### 导航和数据

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/search"
        android:title="Search"
        android:icon="@android:drawable/ic_menu_search"
        android:showAsAction="ifRoom|withText"
    />
    <!--可以给标题栏加组件，比如搜索，在menu的三个点的左边-->
    <item
        android:id="@+id/search"
        android:title="Search"
        android:icon="@android:drawable/ic_menu_search"
        android:showAsAction="ifRoom|collapseActionView"
        android:actionViewClass="android.widget.SearchView"
    />
    <!--Action Provider 实现微信右上角功能，自定义menu布局-->
    <item
        android:id="@+id/share"
        android:title="@string/share"
        android:showAsAction="ifRoom"
        android:actionProviderClass="android.widget.ShareActionProvider"
    />
</menu>
```

#### 标签导航

```java
public void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    final ActionBar bar = getActionBar();
    bar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);

    Tab t = bar.newtab();
    t.setText("Tab Text");
    t.setTableListener(this);
    bar.addTab(bar.newTab());
}

public void onTabSelected(ActionBar.Tab tab, FragmentTransaction ft) {
    ft.replace(R.id.content, fragment, null);
}

public void onTabUnselected(ActionBar.Tab tab, FragmentTransaction ft) {
    ft.remove(fragment);
}

public void onTabReselected(ActionBar.Tab tab, FragmentTransaction ft) {
}
```

#### TabWidget ？ 略
#### Adapter ViewHolder

```java
public class TimeListAdapter extends CursorAdapter {
    private static class ViewHolder {
        int nameIndex;
        int timeIndex;
        TextViwe name;
        TextViwe time;
    }

    public TimeListAdapter(Context context, Cursor c, int flags) {
        super(context, c, flags);
    }

    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        ViweHolder holder = (ViewHolder)view.getTag();
        holder.name.setText(cursor.getString(holder.nameIndex));
        holder.time.setText(cursor.getString(holder.timeIndex));
    }
    @Override
    public View newView(Context context, Cursor cursor, ViewGroup parent) {
        View view = LayoutInflater.from(context).inflate(R.layout.time_row, null);
        ViewHolder holder = new ViewHolder();
        holder.name = (TextView) view.findViewById(R.id.task_name);
        holder.time = (TextView) view.findViweById(R.id.task_time);
        holder.nameIndex = cursor.getColumnIndexThrow(TaskProvider.Task.NAME);
        holder.timeIndex = cursor.getColumnIndexThrow(TaskProvider.Task.DATE);
        view.setTag(holder);
        return view;
    }
}
```

#### loader 略

## 处理手势操作

单个手指操作  
hello文本跟随手指在屏幕上移动  

```java
public class TouchExample extends View {
    private Paint mPaint;
    private float mFontSize;
    private float dx;
    private float dy;
    public TouchExample(Context context) {
        super(context);
        mFontSize = 16 * getResources().getDisplayMetrics().density;
        mPaint = new Paint();
        mPaint.setColor(Color.WHITE);
        mPaint.setTextSize(mFontSize);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        String text = "Hello";
        canvas.drawText(text, dx, dy, mParint);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        dx = event.getX();
        dy = event.getY();
        invalidate();
        //应当返回true占用事件，防止基类处理，否则，只能收到ACTION_DOWN
        return true;
    }
}

public calss GestureActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TouchExample view = new TouchExample(this);
        setContextView(view);
    }
}
```

#### MotionEvent

Action列表  
ACTION_DOWN  
ACTION_POINTER_DOWN  
ACTION_POINTER_UP  
ACTION_MOVE  
ACTION_UP  
ACTION_CANCEL  

需要增加新的类放置触摸点  

```java
final static int MAX_POINTERS = 5;
private Pointer[] mPointers = new Pointer[MAX_POINTERS];
class Pointer {
    float x = 0;
    float y = 0;
    int index = -1;
    int id = -1;
}

public TouchExample(Context context) {
    for (int i = 0; i < MAX_POINTERS; i++) {
        mPointers[i] = new Pointer();
    }
}

// 记录手指
@Override
public boolean onTouchEvent(MotionEvent event) {
    int pointerCount = Math.min(event.getPointerCount(), MAX_POINTERS);
    switch (event.getAction() & MotionEvent.ACTION_MASK) {
        case MotionEvent.ACTION_DOWN:
        case MotionEvent.ACTION_POINTER_DOWN:
        case MotionEvent.ACTION_MOVE:
            // Clear previous pointers
            for (int id = 0; id < MAX_POINTERS; id++)
                mPointers[id].index = -1;
            // Now fill in the current pointers
            for (int i = 0; i < pointerCount; i++ ) {
                int id = event.getPointerId(i);
                Pointer pointer = mPointers[id];
                pointer.index = i;
                pointer.id = id;
                pointer.x = event.getX(i);
                pointer.y = event.getY(i);
            }
            invalidate();
            break;
        case MotionEvent.ACTION_CANCEL:
            for (int i = 0; i < pointCount; i++) {
                int id = event.getPointerId(i);
                mPointers[id].index = -1;
            }
            invalidate();
            break;
    }
    return true;
}

// 展示每个触摸点的索引值和id
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    for (Pointer p : mPointers) {
        if (p.index != -1) {
            String text = "Index: " + p.index + " ID: " + p.id;
            canvas.drawText(text, p.x, p.y, mPaint);
        }
    }
}
```

#### GestureDetector 方便的类 响应手势操作
点击 双击 滑动 速动fling  
拿捏缩放 ScaleGestureDetector  
```java
// 双击屏幕缩放文本，再次双击恢复大小
public class TouchExample extends View {
    private float mSacle = 1.0f;
    private GestureDetector mGestureDetector;
    private ScaleGestureDetector mScaleGestureDetector;
    public TouchExample(Context context) {
        super(context);
        mGestureDetector = new GestureDetector(context, new ZoomGesture());
        mScaleGestureDetector = new GestureDetector(context, new ScaleGesture());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mGestureDetector.onTouchEvent(event);
        mScaleGestureDetector.onTouchEvent(event);
    }
}

public class ZoomGesture extends GestureDetector.SompleOnGestureListener {
    private boolean normal = true;
    @Override
    public boolean onDoubleTap(MotionEvent e) {
        mScale = normal ? 3f : 1f;
        mPaint.setTextSize(mScale * mFontSize);
        normal = !normal;
        invalidate();
        return true;
    }
}

public class ScaleGesture extends ScaleGestureDetector.SimpleOnScaleGestureListener {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        mScale *= detector.getScaleFactor();
        mPaint.setTextSize(mScale * mFontSize);
        invalidate();
        return true;
    }
}
```

### 动画

#### Drawable动画

动画球例子  

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:andoird="http://schemas.android.com/apk/res/android"
    android:shape="oval" >
    <solid android:color="FFFFFF"/>
    <size android:height="100dp" android:width="100dp" />
</shape>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:visible="true"
    android:oneshot="true">
    <item android:drawable="@drawable/white_circle"
        android:duration="250"/>
    <item android:drawable="@drawable/gray_circle"
        android:duration="250"/>
    <item android:drawable="@drawable/black_circle"
        android:duration="250"/>
</animation-list>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/image_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="center"
    android:src="@drawable/animation"
/>
```

```java
public class AnimationExampleActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstanceState);
        setContentView(R.layout.main);
        ImageView iv = (ImageView) findViewById(R.id.image_view);
        iv.setOnTouchListener(new OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                ImageView iv = (ImageView) v;
                AnimationDrawable ad = (AnimationDrawable) iv.getDrawable();
                ad.stop();
                ad.start();
                return true;
            }
        });
    }
}
```

#### 视图动画框架

/res/anim/文件夹下放置  
Translate 移动视图  
Scale 改变视图的尺寸  
Rotate 翻转视图  
Alpha  改变视图的透明度  

```xml
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:toYDelta="25%"
    android:toXDelta="25%"
/>
```

```java
TranslateAnimation anim = 
    new TranslateAnimation(
        TranslateAnimation.RELATIVE_TO_SELF, 0.0f,
        TranslateAnimation.RELATIVE_TO_SELF, 0.25f,
        TranslateAnimation.RELATIVE_TO_SELF, 0.0f,
        TranslateAnimation.RELATIVE_TO_SELF, 0.25f);
anim.setDuration(500);
```

组合  
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="500"
        android:toYDelta="25%"
        android:toXDelta="25%"
        />
    <alpha
        android:dutation="500"
        android:startOffset="500"
        android:fromAlpha="1.0"
        android:toAlpha="0.0"
    />
</set>
```
设置Ease  
accelerator  
decelerate  
overshoot  
bounce  

```xml
android:interpolator="@android:anim/accelerate_interpolator"
```

使用  

```java
TextViwe tv = (TextView)findViewById(R.id.text);
Animation animation = Animationutils.loadAnimation(this, R.anim.slide);
animation.setAnimationListener(new AnimationListener(){
    @Override
    public void onAnimationStart(Animation animation) {}
    @Override
    public void onAnimationRepeat(Animation animation) {}
    @Overrid
    public void onAnimationEnd(Animation animation) {
        tv.setVisibility(View.INVISIBLE);
    }
});
tv.startAnimation(animation);
```

#### 属性动画 略

### 自定义视图

调用invalidate()强制绘制  

```java
public class CrossView extends View {
    public CrossView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(calculateMeasure(widthMeasureSpec), calculateMeasure(heightMeasureSpec));
    }

    private static final int DEFAULT_SIZE = 100;
    private int calculateMeasure(int meausreSpec) {
        int result = (int) (DEFAULT_SIZE * getResources().getDisplayMetrics().density);
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSPec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else if (specMode == MeasureSpec.AT_MOST) {
            result = Math.min(result, specSize);
        }
        return result;
    }

    private Paint mPaint;
    public CrossView(Context context, AttributeSet attrs) {
        super(context);
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setColor(0xFFFFFFFF);
    }

    float[] mPoints = {
        0.5f, 0f, 0.5f, 1f,
        0f, 0.5f, 1f, 0.5f
    };

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.save();
        int scale = getWidth();
        canvas.scale(scale, scale);
        canvas.drawLines(mPoints, mPaint);
        canvas.restore();
    }
}
```

```xml
<com.example.CrossView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
/>
```

如果是内部类  
com.example.CustomViewActivity$CrossView

#### 自定义属性

res/values/目录  
attr 必须包括name和format  
已经定义过format不需要再定义  
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <attr name="test" format="string" />
    <declare-styleable name="cross">
        <attr name="android:color" />
        <attr name="rotation" format="string" />
        <attr name="test" />
    </declare-styleable>

    <attr name="enum_attr">
        <enum name="value1" value="1" />
        <enum name="value2" value="2" />
    </attr>
    <!--可以进行拼接-->
    <com.example.Foo example:flag_attr="flag1|flag2" />
    <attr name="flag_attr">
        <flag name="flag1" value="0x01" />
        <flag name="flag2" value="0x02" />
    </attr>
</resources>
```

使用  
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:example="http://schemas.android.com/apk/res/com.example">
    <com.example.CrossView
        example:rotation="30"
    />
</LinearLayout>
```

---