##一. Android系统详解
###1. 分层的架构

- application：应用层(java)
- application framework ： 应用框架层(java+JNI)
- libraries和dalvik ： 函数库和虚拟机层(c/c++) 
- linux kernel：linux 内核驱动层(c)

###2. 两种虚拟机的不同
版权问题：
	
- jvm ： java虚拟机 sun
- dvm：  dalvik虚拟机  google。专门为移动设备定制，针对手机内存、CPU性能有限等情况做了优化。

区别：

1. 基于的架构不同，jvm 基于栈架构，栈是位于内存上的一个空间，执行指令操作，需要向cpu寻址；dvm 基于寄存器架构，寄存器是cpu的一个组成部分，执行指令操作无需寻址直接执行。
2. 执行文件的格式不同，jvm执行的是多个.class文件。 dvm执行的是一个.dex文件

###3. art模式（android runtime，用来代替dalvik虚拟机）
空间换时间的概念。

**art：**程序在安装时需要预编译读取，将代码转换为机器码。好处:程序运行时，无需时时转换，运行速度快；缺点：安装时间稍长，由于转换机器码，所以占用略高的存储空间。
###4. Android系统启动过程
1. Bootloader引导程序
2. Linux kernel内核启动
3. Android系统启动
	1. 启动入口：init进程（Linux系统中用户空间的第一个进程）
	2. 加载配置（init.rc）
	3. 启动孵化器（zygote）——此进程是Android系统启动关键服务的一个母进程
	4. system_init启动Native世界
	5. ServiceThread启动java世界

##二. Activity详解
###1. activity的4种状态

- running
- paused(如果内存实在不足，会被回收，会造成黑屏)
- stopped
- killed

###2. activity的生命周期
![](http://i.imgur.com/OTkyd1v.png)

- Activity启动
- onCreate()：activity被创建时回调
- onStart()：可见但是无法交互
- onResume()：前台可见，可与用户交互
- onPause()：可见但是无法交互
- onStop()：activity已被完全覆盖
- onDestroy()：activity正在被销毁，做一些资源释放和回收操作

>1. 新Activity生命周期方法在原Activity的`onPause()`方法执行后才回调。
>2. 若新Activity使用透明主题，则旧Activity不会回调onStop()方法。

###3. Activity异常生命周期
异常的生命周期是指Activity被系统回收或者当前设备的configuration发生变化（一般指横竖屏切换）从而导致Activity被销毁重建。

####异常一：相关configuration发生变化导致Activity被杀死。

在此时，会出现以下两种情况。

- 在被杀死Activity的`onPause()`方法后回调`onSaveInstanceState()`方法。
- 在重启Activity的`onStart()`方法后回调`onRestoreInstanceState()`方法。
>1. `onRestoreInstanceState()`和`onCreate()`都可以进行数据恢复，但推荐使用前者，因为此时Bundle一定有值。
>2. 每个view也有`onSaveInstanceState()`和`onRestoreInstanceState()`方法。所以android会自动帮我们恢复一定的数据。比如：文本框数据等。

####异常二：内存不足导致Activity被杀死（此时也可重建，如上方法）。

####解决方法：配置configChange属性

设置Activity的xml属性：`android:configChanges="orientation|keyboardHidden|screensize"`

>- orientation：屏幕方向发生变化（API<13）
>- keyboardHidden：虚拟键盘隐藏
>- screensize：当设备旋转时屏幕尺寸发生变化（API>13）
>
>当横竖切换时，Activity不重建，而是会回调`onConfigurationChanged()`方法。

###4. 进程优先级
前台进程&nbsp;&nbsp;&nbsp;&nbsp;可见进程&nbsp;&nbsp;&nbsp;&nbsp;服务进程&nbsp;&nbsp;&nbsp;&nbsp;后台进程&nbsp;&nbsp;&nbsp;&nbsp;空进程

1. 前台进程：Foreground Process
	
	- 处于前台正与用户交互的Activity
	- 与前台Activity绑定的service
	- 调用了`startForeground()`方法的Service
	- 正在执行`onReceive()`的BroadcastReceiver
	- 正在执行`onCreate()`、`onStart()`、`onDestory()`的Service

2. 可见进程：Visible Process

3. 服务进程：Service Process

4. 后台进程：Background Process

	- 若需要回收，按最近最少使用顺序进行回收。

5. 空进程：Empty Process
###5. Android启动模式
- standard
- singletop：栈顶复用
- singletask：栈内复用
- singleinstance：任务栈独享

###6. scheme跳转协议
android中的URL scheme是一种页面内跳转协议，是一种非常好的实现机制，通过定义自己的scheme协议，可非常方便跳转app中的各个界面。

- 服务器可以定制化告诉App跳转那个界面
- 可以通过通知栏消息定制化跳转界面
- 可以通过H5页面跳转页面等。

一个完整的URL Scheme协议格式由scheme、host、port、path、query组成，其结构如下所示：

	<scheme>://<host>:<port>/<path>?<query>
	xl://goods:8888/goodsDetail?goodsId=10011002

####使用方法：

1. 在AndroidManifest.xml中对activity标签增加<intent-filter>设置Scheme
	
	    <intent-filter><!--URL Scheme启动-->
                <!--必有项-->
                <action android:name="android.intent.action.VIEW" />
                <!--如果希望该应用可以通过浏览器的连接启动，则添加该项-->
                <category android:name="android.intent.category.BROWSABLE"/>
                <!--表示该页面可以被隐式调用，必须加上该项-->
                <category android:name="android.intent.category.DEFAULT"/>
                <!--协议部分-->
                <data android:scheme="jctscheme"
                    android:host="jct_activity"/>
        </intent-filter>

2. 使用URL启动Activity

		Uri data = Uri.parse("jctscheme://jct_activity");
        Intent intent = new Intent(Intent.ACTION_VIEW,data);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        try {
        	startActivityForResult(intent,RESULT_OK);
        }catch (Exception e){
            e.printStackTrace();
            Toast.makeText(MainActivity.this,"没有匹配的APP",Toast.LENGTH_SHORT).show();
        }

>- 在网页中调用`<a href="jctschemel://jct_activity">打开新的应用</a>`
>- 在JS中调用`window.location = "jctscheme://jct_activity";`

####如何判断URL Scheme是否有效？

	boolean checkUrlScheme(Intent intent){
    	PackageManager packageManager = getPackageManager();
    	List<ResolveInfo> activities = packageManager.queryIntentActivities(intent, 0);
    	return !activities.isEmpty();
	}



###7. Android应用冷启动时白色闪屏
####现象：
冷启动APP的时候，需要完成第一次绘制才会显相应的布局，如果处理比较耗时的代码（应用较大），则会产生白屏后再显示布局。 

####原因：
这是由于`style`的原因。新创建的应用默认主题为`@style/AppTheme`，而`AppTheme`默认的`parent`为`Theme.AppCompat.Light.DarkActionBar`。一直查看它的父类你会发现，在`Platform.AppCompat.Light`中设置了`<item name="android:windowBackground">@color/background_material_light</item>`
它的颜色为白色。如果activity加载时间较长，则会先显示应用的背景，故出现白屏。

####解决方法：
1. 把应用默认主题设置为透明主题。例如`android:theme="@android:style/Theme.Translucent.NoTitleBar`(需要修改mainActivity继承Activity，否则会报错)
2. 修改当前主题中的属性，在属性中添加背景透明`<item name="android:windowIsTranslucent">true</item>`
3. 修改当前主题中的属性，在属性中添加背景图片`<item name="android:windowBackground">@mipmap/logo</item>`，但是这种情况下`Activity`中的背景也会变成该图片
4. 为`Activity`设置背景图片。
###8. 同一个应用两个不相关的activity实时通信
现有A,B,C三个`Activity`，我们可以很容易的在A,B,C销毁的时候传数据给其他`Activity`,但如何在C不销毁的时候给A传数据呢？这里想到了三种方法：
####1. 广播
1. 在C中发送广播
   
		public static final String action = "jct.test.action";
		Intent intent = new Intent(action);
	    intent.putExtra("data","yes i am data");
	    sendBroadcast(intent);

2. 在a中接受广播

		IntentFilter intentFilter = new IntentFilter(ThirdActivity.action);
	    registerReceiver(broadcastReceiver,intentFilter);
	
		BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
	        @Override
	        public void onReceive(Context context, Intent intent) {
	            button.setText("我又被改变了。。。。");
	        }
	    };
####2. Handler
Handler需要在两个Activity之间共享Handler。在Androd中，多个Activity之间可以通过Application共享数据，代码如下：<br>

1. 创建MyAPP并继承Application

		public class MyAPP extends Application {
	    	private Handler handler = null;
			
			public void setHandler(Handler handler){
	       	 	this.handler = handler;
	    	}
	
	    	public Handler getHandler(){
	        	return handler;
			}
		}
	注意：要使MyAPP生效需要在manifest中设置application标签的name为MyAPP,`android:name=".MyAPP"`。<br>
2. 在A中初始化Handler并给MyAPP中的handler赋值。

		private Handler handler = new Handler(){
	        @Override
	        public void handleMessage(Message msg) {
	            super.handleMessage(msg);
	            button.setText("我又被改变了");
	        }
	    };
		private MyAPP myAPP;
	
		myAPP = (MyAPP) getApplication();
	    myAPP.setHandler(handler);

3. 在C可通过MyAPP的实例获取A的handler。
	
		 MyAPP myAPP = (MyAPP) getApplication();
	     Handler handler = myAPP.getHandler();
	     handler.sendEmptyMessage(0);
####3. 回调
1. 回掉接口代码如下：

		public interface NotifySendMessage {
	    	public void sendMessage(String msg);
		}

2. 回调管理器代码如下：

		public class NotifySendMessageManager {
	    	private NotifySendMessage listener;
	    	private static NotifyMessageManager notifyMessageManager = null;
	
	    	public static NotifyMessageManager getNotifyMessage(){
	        	if (notifyMessageManager == null){
	           		notifyMessageManager = new NotifyMessageManager();
	        	}
	        	return notifyMessageManager;
	    	}
	
	    	public void setNotifyMessagq(NotifySendMessage nm){
	    	    listener = nm;
	    	}
	    	public void sendNotifyMessage(String msg){
	    	    listener.sendMessage(msg);
	    	}
		}

	1. 首先，让A实现接口`NotifySendMessage`并注册到`NotifySendMessageManager`。此时，任何类调用`getNotifyMessage()`都会获取到与A一样的管理类实例。
	2. 在C中通过`getNotifyMessage()`获取管理类对象并调用`sendNotifyMessage(String msg)`会引起A中`sendMessage(String msg)`函数触发。

##三. Fragment详解

###1. Fragment为什么被称为第五大组件？
它有自己的生命周期，可动态加载到Activity。

###2. Fragment加载到Activity的两种方式：
- 添加Fragment到Activity的布局文件中
- 动态在activity中添加fragment

###3. FragmentPagerAdapter与FragmentStatePagerAdapter的区别：
viewpager -->  内存消耗

- FragmentStatePagerAdapter销毁Item的时候调用remove,真正的释放内存。适合较多的fragment页面。

- FragmentPagerAdapter销毁Item的时候调用的是detach，只是把fragment的UI与activity的UI脱离。适合较少的fragment页面。

###4. Fragment的生命周期

![](http://i.imgur.com/QIjdC3f.png)

###5. Fragment之间的通信
1. 在Fragment中调用Activity中的方法

	`getActivity()`
2. 在Activity中调用Fragment中的方法

	接口回调
	FragmentManager的findFragmentById()
3. 在Fragment中调用Fragment中的方法
	
	可以先获取Activity再获取Fragment
###6. Fragment管理器：FragmentManager
1. replace：替换fragment实例
2. add：添加fragment实例
3. remove：移除fragment实例
##四. Service详解
###1. Service是什么？
Service是一个一种可以在后台执行长时间运行操作（**不是耗时操作**）而没有用户界面的应用组件。Service可以由其他组件进行启动，如Activity、broadcast等，启动后与启动者无关，即使他们被销毁了。
###2. Service和Thread区别
Thread是程序执行的最小单元，是分配CPU的基本单位，可以执行异步操作。
Service运行在Android的主线程中，轻量级的IPC通信，并没有Thread那么独立。
他俩之间没有任何联系。完全不同的概念。
	
Activity创建线程后，无法放心的销毁。而Service不一样。
###3. 两种启动Service的方式
1. startService

	Activity被销毁后不受影响，除非调用stopService()或者Service调用stopSelf();
	>onStartCommand返回值int。若返回START_STICK。这个值意味着当服务因系统内存不足而被销毁后，当系统内存空闲的时候，系统会尝试重新创建service，重新调用onstartCommand，但是此时intent为空。

2. bindService

	允许service与activity传递数据；允许进程间通信（服务与activity不在同一个进程）
	1. 创建BindService服务端，继承自Service并在类中创建一个实现IBinder接口的实例对象并提供公共方法给客户端调用。
	2. 从onBind()回调方法返回此Binder实例。
	3. 在客户端中，从onServiceConnected()回调方法接收Binder，并使用提供的方法调用绑定服务。
	>onServiceDisconnected()方法正常情况下不会调用，它的调用时机是当Service服务被意外销毁时。
##五. Broadcast Receiver详解
###1. 广播的使用场景
1. 同一app具有多个进程的不同组件之间的消息通信
2. 不同app之间的组件之间的消息通信
>广播有三种：有序、无序、本地
###2. 实现广播-receiver
1. 静态注册：注册完成就一直运行，activity（应用）杀死了**仍然运行**。
2. 动态注册：跟随activity的生命周期，一定要**销毁**！
###3. 广播实现机制
1. 自定义广播接收者BroadcastReceiver,并复写onReceiver()方法
2. 广播通过Binder机制向AMS(Activity Manager Service)进行注册
3. 广播发送者通过Binder机制向AMS发送广播
4. AMS查找符合相应条件（IntentFilter/Permission等）的BroadcastReceiver，将广播发送到BroadcastReceiver(一般情况下是Activity)相应的消息循环队列中
5. 消息循环执行拿到此广播，回调BroadcastReceiver中的onReceiver()方法

>AMS：贯穿android系统组件的核心服务，负责四大组件的启动、切换、调度。应用程序的管理和调度。
>
>Binder：android进程间通信的核心，整体设计架构是客户端服务端架构。（C/S），客户端进程可以获取到服务端的代理，并通过代理接口中的方法读取数据，完成进程间的数据通信。
###4. LocalBroadcastManager详解
1. 使用它发送的广播将只在自身APP内传播，因此你不用担心泄露隐私数据。
2. 其他APP无法对你的APP发送该广播，因为你的APP根本就不可能接收到非自身应用发送的该广播，因此你不必担心有安全漏洞可以利用。
3. 比系统的全局广播更加高效。
>1. 高效的原因主要是因为它内部通过Handler实现的，它的sendBroadcast()方法其实是通过handler发送一个Message实现的。
>2. 既然它内部是通过Handler来实现广播的发送的，那么相比与系统广播通过Binder实现那肯定是更高效了，同时使用Handler来实现，别的应用无法向我们的应用发送该广播，而我们应用内发送的广播也不会离开我们的应用。
>3. LocalBroadcastManager内部协作主要是靠这两个Map集合：HasMap<BroadcastReceiver,ArrayList<IntentFilter>> mReceivers（广播接收者对IntentFilter）和HasMap<String,ArrayList<ReceiverRecord>> mActions（action与广播对象），当然还有一个List集合ArrayList<BroadcastRecord> mPendingBroadcasts，这个主要是存储待接收的广播对象。

##六. 数据存储详解

###1. 文件获取方式
####/data/data：`context.getFileDir().getPath()`
它是一个应用程序的私有目录，只有当前应用程序有权限访问读写，其他应用无权限访问。一些安全性要求比较高的数据存放在该目录，一般用来存放size比较小的数据。
####/sdcard：`Enviroment.getExternalStorageDirectory().getPath()`
它是一个外部存储目录，一般用来存放一些安全性不高的数据，文件size比较大的数据。**需要读写权限。**	
###2. 文件的权限概念 
1. 通过context对象获取一个私有目录的文件读取流。`/data/data/packagename/files/userinfoi.txt`

	`FileInputStream fileInputStream = context.openFileInput("userinfo.txt");`

2. 通过context对象得到私有目录下一个文件写入流。

	`FileOutputStream fileOutputStream = context.openFileOutput("userinfo.txt", Context.MODE_PRIVATE);`	

	> name：私有目录文件的名称。mode：文件的操作模式。包括，私有，追加，全局读，全局写（全局读，全局写已废弃）
####linux下一个文件的权限由10位标示：

- 1位：文件的类型，d：文件夹 l：快捷方式  -：文件
- 2-4： 该文件所属用户对本文件的权限，rwx：用二进制标示，如果不是-就用1标示，是-用0标示
- 5-7：该文件所属用户组对本文件的权限
- 8-10：其他用户对该文件的权限。
	
###3. xml格式数据的解析

- dom解析：基于全文加载的解析方式   
- sax解析：基于事件的逐行解析方式  
- pull解析：同sax

###4. 数据库的事务 
**事务：**执行多条sql语句，要么同时执行成功，要么同时执行失败，不能有的成功，有的失败。

银行转账

	//点击按钮执行该方法
	public void transtation(View v){
		//1.创建一个帮助类的对象
		BankOpenHelper bankOpenHelper = new BankOpenHelper(this);
		//2.调用数据库帮助类对象的getReadableDatabase创建数据库，初始化表数据，获取一个SqliteDatabase对象去做转账（sql语句）
		SQLiteDatabase db = bankOpenHelper.getReadableDatabase();
		//3.转账,将李四的钱减200，张三加200
		db.beginTransaction();//开启一个数据库事务
		try {
			db.execSQL("update account set money= money-200 where name=?",new String[]{"李四"});
			int i = 100/0;//模拟一个异常
			db.execSQL("update account set money= money+200 where name=?",new String[]{"张三"});
			db.setTransactionSuccessful();//标记事务中的sql语句全部成功执行
		} finally {
			db.endTransaction();//判断事务的标记是否成功，如果不成功，回滚错误之前执行的sql语句 
		}
	}
##七. Webview详解
###1. Webview常见的一些坑
1. Android API level 16 以及之前的版本存在远程代码执行安全漏洞，该漏洞源于程序没有正确限制使用`Webview.addJavascriptInterface`方法,远程攻击者可通过使用Java Reflection（反射） API利用该漏洞执行任意Java对象的方法。
2. webview在布局文件中的使用：webview写在其他容器中时。
	>当销毁webview时，需要从其他容器先把webview remove掉，在进行webview的removeAllViews();
3. jsbridge：通过javascript来构建桥，使本地native端可以调用web端的代码，也可以使web端调用native端代码。
4. webviewClient.onPageFinished->WebChromeClient.onProgressChanged
	>前者在网页加载完成时回调。会判断网页内容是否真的加载完毕，跳转的时候，会回调无数次。所以最好使用后者，靠谱些。
5. 后台耗电：webview会开启线程，所以如果没有很好的销毁的话，残余的线程会一直运行，导致应用程序耗电量居高不下。在Activity.onDestroy()中直接调用System.exit(0)，使应用程序完全移除虚拟机。
6. Webview硬件加速导致页面渲染问题
	>会导致页面加载白块，闪屏等效果。解决方法就是webview设置暂时关闭硬件加速。
###2. 关于webview的内存泄露问题
webview关联Activity，webview内部执行的操作在新的线程中，Activity无法控制线程，所以webview一直持有Activity引用，导致Activity无法正常回收。

**解决办法：**

1. 独立进程，简单暴力，不过可能涉及到进程间通信。
2. 动态添加webview，对传入WebView中使用的Context使用弱引用，动态添加webview意思是在布局外创建个ViewGroup来放置webview，Activity创建时add进来，在Activity停止时remove掉。
##八. Binder详解
###1. Linux内核的基础知识
1. 进程隔离/虚拟地址空间
	
	保护操作系统中的进程互不干扰，实现利用了虚拟地址空间。
2. 系统调用
	
	Linux中有保护机制告诉应用程序只可访问某些许可的资源，也就是把系统内核层与上层应用程序抽象分离开，用户可通过系统调用在用户空间访问内核的某些程序。
3. binder驱动
	
	各个进程通过binder内核进行通信的驱动机制。
###2. 为什么使用Binder
Android使用的Linux内核拥有非常多的跨进程通信机制，如管道、socket。为什么使用Binder？

1. 性能（更加高效）
2. 安全（传统的没有进行严格的身份认证，Binder可对通信双方进行身份校验）
###3. Binder通信模型

1. 通信录：binder驱动（编号等）
2. 电话基站：serviceManage 

![](http://i.imgur.com/rEmqRAI.png)

>Client只是持有服务端一个代理对象，是通过代理对象协助驱动完成跨进程通信。
###4. 到底什么是Binder
1. 通常意义下，Binder指的是一种通信机制（跨进程）
2. 对于Server进程来说，Binder指的是Binder本地对象；但是对于Client来说，Binder指的是Binder代理对象。
3. 对于传输过程而言，Binder是可以进行跨进程传递的对象。它自动完成代理对象和本地对象的转换。
###5. aidl
1. asInterface()：若A跟B是同一个进程，就不会跨进程；若不是同一个进程，则返回proxy类，也就是代理对象。
2. onTransact()：会根据aidl函数返回的编号进行相应的方法调用。

##九. 消息处理详解
###1. 什么是Handler
handler通过发送和处理Message和Runnable对象来关联相对应线程的MessageQueue。

1. 可以让对应的Message和Runnable在未来的某个时间点进行相应处理。
2. 让自己想要处理的耗时操作放在子线程，让更新ui的操作放在主线程。
###2. handler的使用方法
1. post(runnable)
	
	底层调用sendMessage。
2. sendMessage(message)

###3. handler原理

有几个主要元素：

1. Message:用来携带子线程中的数据。
2. MessageQueue:用来存放所有子线程发来的Message.
3. Handler:用来在子线程中发送Message，在主线程中接受Message，处理结果
4. Looper:是一个消息循环器，一直循环遍历MessageQueue，从MessageQueue中取一个Message，派发给Handler处理。
5. 另一种方式：Message msg = Messge.obtain;可控制msg对象的数量只有一个。

###4. handler引起的内存泄漏及解决方法
原因：非静态内部类持有外部类的匿名引用，导致外部activity无法释放

解决办法：

1. handler内部持有外部activity的弱引用
2. 把handler改为静态内部类
3. activity销毁的时候调用mHandler.removeCallback();

###5. 常见消息处理api

面试：子线程一定不能更新UI？ 

答：
1. SurfaceView：多媒体视频播放，可以在子线程中更新UI
2. Progress（进度）相关的控件：也是可以在子线程中更新Ui
>审计机制：activity完全显示的时候审计机制才会去检测子线程有没有更新Ui。
	
方法：
	
1. 使用activity的runOnUiThread方法更新ui,无论当前线程是否是主线程，都将在主线程执行。
			
		runOnUiThread(new Runnable() {		
			@Override
				public void run() {
					tv_simple.setText("我被更新了");
				}
			});
2. 使用handler直接post到主线程，handler需要在主线程创建

		//延迟多少毫米执行runnable。
		mHandler.postDelayed(new Runnable() {
			@Override
			public void run() {
				tv_simple.setText("我被更新了");
			}
		}, 1000*5);

##十. AsyncTask详解
本质上就是一个封装了线程池和handler的异步框架。

**使用方法：**(三个参数，五个方法)

1. AsyncTask<Integer（传入）,Integer（显示的进度）,String（返回值）>
2. 五个方法
	1. onPreExecute()：在耗时操作之前做一些初始化操作，UI线程中调用。
	2. doInBackground()：在前一个调用完之后调用，会产生结果返回到onPostExecute()方法中。
	3. onProgressUpdate()：动态显示进度条等。
	4. onPostExecute()：后台计算完成后调用。
	5. UpdateInfoAsyncTask()：不用。

**AsyncTask的机制原理：**

1. AsyncTask的本质是一个静态的线程池，AsyncTask派生出的子类可以实现不同的异步任务，这些异步任务都是提交到静态的线程池中执行。
2. 线程池中的工作线程执行doInBackground(mParams)方法执行异步任务。
3. 当任务状态改变后，工作线程会向UI线程发送消息，AsyncTask内部的InternalHandler响应这些消息，并调用相关的回调函数。

**注意事项：**

1. 内存泄漏
2. 生命周期
	>activity销毁后，若不处理asyncTask可能会导致崩溃。（无法更新ui）
3. 结果丢失
	>如果activity被销毁后重新创建，之前的asyncTask持有之前的引用，但是已无效，所以会丢失
4. 并行or串行

##十一. handlerThread详解
多次创建和销毁线程很消耗系统资源。

handlerThread = handler + thread + looper

###handlerThread的特点：
1. HandlerThread本质上是一个线程类，它继承了Thread。
2. HandlerThread有自己的内部Looper对象，可以进行looper循环。
3. 通过获取HandlerThread的looper对象传递给Handler对象，可以在handlerMessage方法中执行异步任务。
4. 优点是不会阻塞，减少了对性能的消耗，缺点是不能同时进行多任务的处理，需要等待进行处理。处理效率低。
5. 与线程池重并发不同，HandlerThread是一个串行队列，HandlerThread内部只有一个Thread。

##十二. IntentService详解

IntentService是继承并处理异步请求的一个Service，在IntentService内有一个工作线程来处理耗时操作。

- 启动IntentService的方式和启动传统的Service一样。
- 同时，当任务执行完后，IntentService会自动停止，而不需要我们去控制或者stopself()。
- 可以启动IntentService多次，而每一个耗时操作会以工作队列的方式在IntentService的onHandleIntent回调方法中执行。
- 每次只会执行一个工作线程，执行完第一个再执行第二个。它是一个特殊的service，且优先级比service高。

它实际上是一个封装了HandlerThread和Service的异步框架。

##十三. View详解

###1. View树的绘制过程
measure->layout->draw

>1. findViewById()方法，就是在控件树中以树的深度优先遍历来查找对应元素。
>2. ViewGroup通常情况下不需要绘制，因为它本身就没有需要绘制的东西，如果不是指定了ViewGroup的背景颜色，那么ViewGroup的`onDraw()`方法都不会被调用。但是，ViewGroup会使用`dispatchDraw()`方法来绘制其子View。
>3. ViewGroup存在的目的就是为了对其子view进行管理，为其子view添加显示、响应的规则。

###2. measure
1. ViewGroup.LayoutParams
2. MeasureSpec是一个32位的int值，其中高2位为测量的模式，低30位为测量的大小。测量的模式分为3种：EXACTLY（精确值模式：具体值、match_parent）、AT_MOST（最大值模式：wrap_content）、UNSPECIFIED（不指定模式）。
3. measure -> onMeasure -> setMeasuredDimension()
>view类默认的`onMeasure()`只支持EXACTLY模式。所以如果要让自定义View支持wrap_content属性，就必须重写`onMeasure()`方法来制定wrap_contet时的大小。

###3. draw
1. invalidate()：视图若不发生变化，不调用layout
2. requestLayout()：布局发生变化的时候去调用，触发measure、layout，但是不触发draw

###4. View方法区别

- scrollTo(int x,int y)：移动到View中**内容**的x,y坐标。
- scrollBy(int x,int y)：移动到View中**内容**的mScrollX+x,mScrollY+y坐标。
- getScrollX()：获取mScrollX，也就是X方向的偏移量。
- getScrollY()：获取mScrollY，也就是Y方向的偏移量。
- drawRect(左x距离，上距离，右距离，下距离)。

###6. 自定义view

自定义控件通常有三种方法：

1. 对现有控件进行拓展。
2. 通过组合来实现新的控件。（新建属性）
3. 重写View来实现全新的控件。

###7. ListView详解
####RecycleBin内部类
![](http://i.imgur.com/RTNxpnB.png)

半透明比全透明绘制更加耗时。
开启硬件加速。
当listview滑动停止后再加载图。

###8. 常用获取inflate的写法 

1. context：上下文；resource：要转换成view对象的layout的id；root：将layout用root(ViewGroup)包一层作为codify的返回值，一般传null
 
		view = View.inflate(context, R.layout.item_news_layout, null);//将一个布局文件转换成一个view对象

2. 通过LayoutInflater将布局转换成view对象

		view =  LayoutInflater.from(context).inflate(R.layout.item_news_layout, null);

3. 通过context获取系统服务得到一个LayoutInflater，通过LayoutInflater将一个布局转换为view对象
	
		LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
		view = layoutInflater.inflate(R.layout.item_news_layout, null);

##十四. Android 构建
###1. Android构建流程
![](http://i.imgur.com/HV4DVPs.png)

###2. Gradle
Gradle是一个非常先进的项目构建工具，它使用了一种基于Groovy的领域特定语言（DSL）来声明项目设置，摒弃了传统的基于XML的各种繁琐配置。

build.gradle文件中repositories闭包中都声明了jcenter()进行配置。
jcenter是一个代码托管工具，声明后，项目即可使用（引用）任何jcenter上的开源项目。

###3. Proguard
ProGuard工具用于压缩、优化、混淆我们的代码，主作用是可以移除代码中的无用类，字段，方法和属性，同时可以混淆代码。

ProGuard技术的功能：

1. 压缩：检查并移除代码中无用的类
2. 优化：移除无用的字节码命令
3. 混淆：把名称变成无意义的名称等
4. 预检测：对处理后的代码再次进行检测

原理：

EntryPoint：一种标志，表示，proguard过程中，不会被处理的类或方法。

##十五. git常用命令
- git init
- git status：查看仓库的状态
- git diff：后面加文件名，看到这次到上次到底修改了哪些内容
- git add：后面加文件名，提交到暂存区
- git commit：提交代码到仓库
- git clone：从远程仓库克隆一份到我们本地
- git branch：查看当前的分支
- git checkout：切换分支
##十六. Android异常与性能优化
###1. ANR：Application Not Responding
原因：应用程序的响应性是由Activity Manager和WindowManager系统服务监视的。

1. 主线程被IO操作阻塞
2. 主线程中存在耗时的计算
###2. Android中哪些操作是在主线程呢？
- Activity的所有生命周期回调都是执行在主线程的。
- Service默认是执行在主线程的
- BroadcastReceiver的onReceive回调是执行在主线程的
- 没有使用子线程的looper的Handler的handleMessage，post(Runnable)是执行在主线程的
- AsyncTask的回调中除了doInBackground，其他都是执行在主线程

###3. 如何解决anr
1. 使用AsyncTask处理耗时IO操作
2. 使用Thread或者HandlerThread提高优先级（否则优先级与主线程一样，也会造成ANR）
3. 使用handler来处理工作线程的耗时任务
4. Activity的onCreate()和onResume()回调中尽量避免耗时的代码

###4. 布局优化

**Android UI渲染机制**

在Android系统中，通过VSYNC信号触发对UI的渲染，重绘，其时间间隔为16ms。如果不能在16ms内完成绘制，那么就会造成丢帧现象，即当前该重绘的帧被未完成的逻辑阻塞，等待下次信号才开始绘制，及16*2ms，
就会造成画面卡顿。Android手机提供Profile GPU Rendering工具查看渲染时间。

**Overdraw**

Overdraw，过度绘制会浪费很多的CPU、GPU资源，例如系统默认会绘制Activity的背景，而如果再给布局绘制了重叠的背景，那么默认Activity的背景就属于无效的过度绘制。Android手机提供Enable GPU overdraw工具，(尽量增大蓝色的区域，减少红色区域)。

**优化布局层级**

在Android中，系统对View进行测量、布局和绘制时，都是通过对View树的遍历来进行操作的。如果一个View树的高度太高，就会严重影响测量、布局、绘制的速度，因此优化布局的第一个方法就是**降低View树的高度**。

**避免嵌套过多无用布局**

嵌套的布局会让View树的高度变得越来越高，因此在布局时，需要根据自身布局的特点来选择不同的Layout组件，从而避免通过某一种Layout组件来实现功能时的局限性，从而造成嵌套过多的情况发生。

1. 使用<include>标签重用Layout
2. 使用`<ViewStub>`实现View的延迟加载（跟<include>使用类似）
	1. `<ViewStub>`是一个非常轻量级的组件，它不仅不可视，而且大小为0。
	2. 使用`setVisibility(View.VISIBLE)`以及`inflate()`来显示这个View。
	3. `inflate()`方法可以返回引用的布局。
	4. 显示后，会被inflate的Layout取代，多次显示会出错。并将这个Layout的ID重新设置为<viewStub>中通过android:inflatedId属性所指定的ID。
	>`<ViewStub>`与GONE的区别：
	>`<ViewStub>`标签只会在显示时，才去渲染整个布局，而View.GONE，在初始化布局树的时候就已经添加在布局树上了，相比之下`<ViewStub>`标签的布局具有更高的效率。

**Hierarchy Viewer工具，可以帮助我们很快地在视图树中找到冗余的布局，从而有目的地优化布局。**

###5. 内存

通常我说所说的内存是指手机的RAM，它包括以下几个部分：

- **寄存器（Registers）：**速度最快的存储场所，因为寄存器位于处理器内部，在程序中无法控制。
- **栈（Stack）：**存放基本类型的数据和对象的引用，但对象本身不存放在栈中，而是存放在堆中。
- **堆（Heap）：**堆内存用来存放由new创建的对象和数组。在堆中分配的内存，由Java虚拟机的自动垃圾回收期（GC）来管理。
- **静态存储区域（Static Field）：**静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据，Java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量。
- **常量池（Constant Pool）：**JVM虚拟机必须为每个被装载的类型维护一个常量池，常量池就是该类型所用到的常量的一个有序集合，包括直接常量（基本类型，String）和对其他类型、字段和方法的符号引用。

###6. OOM：Out of memory
当前占用的内存加上我们申请的内存资源超过了Dalvik虚拟机的最大内存限制就会抛出的异常，大部分OOM异常都跟Bitmap加载有关。

###7. 内存溢出/内存抖动/内存泄漏
- 内存溢出：OOM。
- 内存抖动：短时间内大量的对象被创建又被马上释放，瞬间产生的对象严重占用内存区域。
- 内存泄漏：进程中的某些对象没有被其他地方引用到了，但它直接或间接引用其他还没被回收的对象，导致gc无法起作用。到一定程度会引起OOM。
###8. 有关bitmap
1. recycle
2. LRU
3. 计算inSampleSize的值
4. 缩略图
5. 三级缓存

###9. 代码优化
1. 对常量使用static修饰符。
2. 使用SurfaceView来替代View进行大量、频繁的绘图操作。
3. 。。。。。

###10.UI卡顿
1. 人为在UI线程中做轻微耗时操作，导致UI线程卡顿。
2. 布局Layout过于复杂，无法在16ms内完成渲染。
3. 同一时间动画执行的次数过多，导致CPU或GPU负载过重。
4. View过度绘制，导致某些像素在同一帧时间内被过度绘制多次，从而使CPU或GPU负载过重。
5. view频繁的触发measure、layout，导致measure、layout累计耗时过多及整个View频繁的重新渲染。
6. 内存频繁触发GC过多，导致暂时阻塞渲染操作。
7. 冗余资源及逻辑等导致加载和执行缓慢。
8. ANR

###13.Android内存管理
1. Android内存管理机制
	1. 分配机制：弹性分配
	2. 回收机制：

2. 内存优化方法
	1. 当Service完成任务后，尽量停止它。
	2. 在UI不可见时，释放掉一些只有UI使用的资源。
	3. 在系统内存紧张的时候，尽可能多的释放掉一些非重要资源。
	4. 避免滥用Bitmap导致的内存浪费。
	5. 使用针对内存优化过的数据容器。
	6. 避免使用依赖注入的框架。
	7. 使用ZIP对齐的APK
	8. 使用多进程。
###14.冷启动优化
冷启动就是在启动应用前，系统中没有该应用的任何进程信息。

1. 冷启动/热启动的区别
		
	热启动：用户使用返回键退出应用，然后马上又重新启动应用。
	冷启动会走APPLICATION类，热启动不走。

2. 冷启动时间的计算
	
	这个时间值从应用启动（创建进程）开始计算，到完成视图的第一次绘制（即Activity内容对用户可见）为止。

3. 冷启动流程
	1. Zygote进程中fork创建出一个新的进程。
	2. 创建和初始化Application、创建MainActivity。
	3. inflate布局、当onCreate/onStart/onResume方法都走完。
	4. contentView的measure/layout/draw显示在界面上。

4. 优化
	1. 减少onCreate()方法的工作量。
	2. 不要让Application参与业务的操作
	3. 不要再Application进行耗时操作
	4. 不要以静态变量的方式在Application中保存数据
	5. 布局/mainThread

###15.其他优化
1. android中不用静态变量存储数据
	- 静态变量等数据由于进程已经被杀死而初始化

2. 有关Sharepreference问题
	-  不能跨进程同步
	-  存储SharePreference的文件过大问题
3. 内存对象序列化
	- Serializable		磁盘、网络
	- Parcelable	内存（进程间通信）
	>1. Serailizable在序列化的时候会产生大量的临时变量，从而引起频繁的GC
	>2. Parcelable不能使用在要将数据存储在磁盘上的情况

###16.MAT工具使用

##十七. 源码详解
###1. OKHttp
1. 创建一个OKHttpClient对象。
2. 创建一个request对象，通过内部类Builder调用生成Request对象。
3. 创建一个Call对象，调用execute/enqueue。

>execute为同步请求，enqueue为异步请求。

intercept：分层的思想。把各种复杂的任务拆分成每个具体的任务。在内部多次实现RealInterceptorChain类，并把index加1。递归。

同步请求：

	public Response execute() throws IOException {
		//同步检查，每个Call只能被执行一次。
	    synchronized (this) {
	      if (executed) throw new IllegalStateException("Already Executed");
	      executed = true;
	    }
	    captureCallStackTrace();
	    try {
			//dispatcher()：主要是执行异步请求，但也可以做同步请求。
	      	client.dispatcher().executed(this);
			//getResponseWithInterceptorChain()拦截器。精髓。
	      	Response result = getResponseWithInterceptorChain();
	      	if (result == null) throw new IOException("Canceled");
	      	return result;
	    } finally {
	      	client.dispatcher().finished(this);
	    }
	}

异步请求：
3个集合：
	
	private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
	private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
	private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

enqueue()

	synchronized void enqueue(AsyncCall call) {
	    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
	      runningAsyncCalls.add(call);
	      executorService().execute(call);
	    } else {
	      readyAsyncCalls.add(call);
	    }
	}	
>封装了线程池。

拦截器：

	Response getResponseWithInterceptorChain() throws IOException {
	    // Build a full stack of interceptors.
	    List<Interceptor> interceptors = new ArrayList<>();
	    interceptors.addAll(client.interceptors());
		//失败重定向
	    interceptors.add(retryAndFollowUpInterceptor);
		//桥接
	    interceptors.add(new BridgeInterceptor(client.cookieJar()));
	    interceptors.add(new CacheInterceptor(client.internalCache()));
	    interceptors.add(new ConnectInterceptor(client));
	    if (!forWebSocket) {
	      	interceptors.addAll(client.networkInterceptors());
	    }
		//网络服务端链接拦截器
	    interceptors.add(new CallServerInterceptor(forWebSocket));
	
	    Interceptor.Chain chain = new RealInterceptorChain(
	        interceptors, null, null, null, 0, originalRequest);
	    return chain.proceed(originalRequest);
    }



![](http://i.imgur.com/5OSWpC6.png)
###2. volley
1. 首先需要获取到一个RequestQueue对象。（多次请求只创建一个）
2. 创建一个StringRequest对象。
3. 将StringRequest对象添加到RequestQueue里面。

![](http://i.imgur.com/D7Qo2Xe.png)

>开启两个线程
###3. OKHttp与volley的区别

OKHTTP：

使用的是HttpURLConnection,不要担心android版本的变换。（至少目前是都支持的）。     

volley：
     
非常适合进行数据量不大，但通信频繁的网络操作。对大文件下载，Volley的表现非常糟糕。

##十八. Java基础
###1. 基本数据类型与包装类型
####拆箱与装箱

- “=”：判断两个值的内存地址。基本数据类型可通过“=”号来进行判等。
- equals()：判断两个数中的内容是否相等。对象。
- Integer以及long会有缓存机制。-128~127

####基本类型之间的转换原则
1. 运算时，容量下的类型自动转换成容量大的类型。
2. 容量大的类型转换成容量小的类型时，要加强制转换符，且精度可能丢失。
3. short、char之间不会互相转换（需要强制转换），byte、short、char三者在计算时首先转换为int类型。
4. 实数常量默认为double类型，整数常量默认为int类型。

####包装类及String类都是定义为public final class的，因此这几个类都不能被继承。
####原始类型是可以通过==直接判断是否相等的，而包装类型时类，通过==判断值是否相等是不对的，必须通过equals()函数。
####加赋值与直接赋值是不等的。
short s1 = 1; s1 = s1 + 1;：出错，需要强制转换成short。

short s1 = 1; s1 += 1;：无错，直接在值上加1。 

###2. String、StringBuffer、StringBuilder
String是字符串常量，它不可更改，因为其内部定义的是一个final类型的数组来保存值的。所以我们每次去更改String值的时候，其实是新建了一个String对象来保存新的值。

String的两种创建方式：

1. String str = "abc";
2. String str2 = new String("def");

第一种方式创建的String对象，是存放在字符串常量池当中，创建过程是，首先在字符串常量池中查找有没有“abc”对象。如果有则将str直接指向它，没有的话就在字符串常量池中创建出来“abc”，然后在将str指向它，而当有另一个String变量被赋值为abc时，直接将字符串常量池中的地址给它。

第二种创建方式其实分两步：
	
	String s = "def";
    String str2 = new String(s);

第一步就是上面那种情况，第二种在堆内存中new出一个String对象，将str2指向该堆内存地址，新new出的String对象内容，是在字符串常量池中找到的或者创建出“def”对象，相当于此时存在两份“def”对象拷贝，一份存在字符串常量池中，一份被堆内存的String对象私有化管理着，所以使用第二种创建方式，实际上创建了两个对象。

StringBuffer是线程安全的，StringBuilder是线程不安全的。不需要重新创建String，直接在后面添加。如果对线程安全要求不高，使用StringBuilder效果更优。

###3. switch支持类型

1. 在1.7之间只支持byte、short、char、int或者其对应的封装类以及Enum类型。
2. 在1.7之后支持string类型。

###4. 构造函数相关

1. 如果一个类中没有写任何的构造方法，JVM会生成一个默认的无参构造方法。
2. 如果一个基类中写了有参构造函数，没有定义无参构造函数，基类是不会默认生成无参构造函数的，而且子类的构造函数中，如果没有显示通过super调用基类构造函数，那么默认是调用父类的无参构造函数。（省略）

###5. Java创建对象的几种方式：
1. new
2. 反射，调用java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。
3. 调用对象的clone()方法。
4. 运用反序列化手段，调用java.io.ObjectInputStream对象的readObject()方法。

###6. 静态代码块执行顺序

执行顺序： 静态代码块->子类静态代码块->父普通代码块->父构造方法->子类普通代码块->子类构造方法

###7. &与&&的区别
&是位运算符，&&是与运算符

###8. 小知识点
1. public<protected<default<private
2. final修饰的类不能被继承，没有子类。
3. abstract修饰的类不能被实例化，必须被子类继承。类只要有一个抽象方法就必定是抽象类，但抽象类不一定要有抽象方法。

##操作系统
###1. 死锁
所谓死锁：是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相的等待的现象，若无外力作用，它们都将无法推动下去，此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

####产生死锁的主要原因有：
1. 系统资源不足。
2. 进程运行推进的顺序不合适。
3. 资源分配不当等。

如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁，其次，进程运行推进顺序与速度不同，也可能出现死锁。

###2. JAVA与死锁

在分布式开发中，锁是线程控制的重要途径。Java为此也提供了2种锁机制，synchronized和lock。

####一、synchronized和lock的用法区别

synchronized：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

lock：需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。

用法区别比较简单，这里不赘述了，如果不懂的可以看看Java基本语法。

####三、synchronized用法

1. synchronized方法

		Public synchronized void methodAAA(){
		
		//….
		
		}

2. 同步代码块

		public void method3(SomeObject so){
		
		    synchronized(so)
		    { 
		       //….. 
		    }
		}

	>这时，锁就是so这个对象，谁拿到这个锁谁就可以运行它所控制的那段代码。当有一个明确的对象作为锁时，就可以这样写程序，但当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的instance变量（它得是一个对象）来充当锁：

		class Foo implements Runnable{
		
		        private byte[] lock = new byte[0]; // 特殊的instance变量
		
		        Public void methodA() {
		
		           synchronized(lock) {
		            //… 
		            }
		        }
		        //…..
		}

###4. 进程与线程
####进线与线程的区别

1. 进程的定义

	进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动，是系统进行资源分配和调度的一个独立单位。进程是程序的一次执行活动，是程序运行的实例，属于动态概念。
（注意：在Mac、Windows NT等采用微内核结构的操作系统中，进程只是资源分配的单位，而不再是调度运行的单位。在微内核系统中，真正调度运行的基本单位是线程。因此，实现并发功能的单位是线程。 ）
进程是系统进行资源分配的基本单位。

2. 线程的定义

	线程是进程的一个实体，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。
	线程是CPU调度和程序执行的最小单位。

