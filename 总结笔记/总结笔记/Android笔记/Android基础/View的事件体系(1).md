##view的位置参数
view是Android中所有控件的基类。View的位置主要由它的四个顶点来决定，分别对应于View的四个属性：top、left、right、bottom。

- top：左上角纵坐标
- left：左上角横坐标
- right：右下角横坐标
- bottom：右下角纵坐标
>**注意：**这些坐标都是相对坐标，是相对于View的父容器来说的。

所以，我们可以借此得出view的宽高和坐标：width = right - left&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;height = bottom - top。

从Android3.0开始，View增加了额外的几个参数：x、y、translationX和translationY。

- x和y：view左上角的坐标
- translationX和translationY：左上角相对于父容器的偏移量

其中，x = left + translationX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;y = top + translationY
>**注意：**view在平移时，top和left表示的是原始左上角的位置信息，其值并不会改变，此时发生变化的是x、y、translationX、translationY。
##MotionEvent、TouchSlop、VelocityTracker、GestureDetector和Scroller
###MotionEvent
通过MotionEvent对象我们可以得到点击事件发生的x和y坐标。系统提供的方法为：getX/getY和getRawX/getRawY。其中，getX/getY返回的是相对于当前View左上角的x和y坐标。getRawX/getRawY返回的是相对于手机屏幕左上角的x和y坐标。
###TouchSlop
TouchSlop是系统所能识别出的被认为是滑动的最小距离。这个值和设备有关，在不同设备上这个值可能是不同的。可通过`ViewConfiguration.get(getContext()).getScaledTouchSlop()`获取。
>只是一个常量，需要我们自己在代码中使用。
###VelocityTracker
**速度追踪：**用来追踪手指在滑动过程的速度，包括水平和竖直方向的速度。

	//初始化VelocityTracker，设置追踪对象
    VelocityTracker velocityTracker = VelocityTracker.obtain();
    velocityTracker.addMovement(event);
    //需要使用的时候进行计算
    //1000为设置的时间间隔，速度 = (终点位置 - 起点位置) / 时间间隔，单位是毫秒
    velocityTracker.computeCurrentVelocity(1000);
    int xVelocity = (int) velocityTracker.getXVelocity();
    int yVelocity = (int) velocityTracker.getYVelocity();
	//当不需要它的时候需要调用clear方法来重置并回收内存
    if (!hasUserful){
    	velocityTracker.clear();
        velocityTracker.recycle();
    }
###GestureDetector
**手势检测：**用于**辅助**检测用户的单机、滑动、长按、双击等行为。

需要创建一个`GestureDetector`对象并实现`OnGestureListener`接口，根据需要还可以实现`OnDoubleTapListener`从而能够监听双击行为。
>**建议：**如果只是监听滑动相关的，建议自己在onTouchEvent中实现，如果要监听双击这种行为的话，那么就使用GestureDetector。

###Scroller
**弹性滑动对象：**用于实现View的弹性滑动。

当时用View的scrollTo/scrollBy方法来进行滑动时，其过程是瞬间完成的，体验不好。而弹性滑动不是瞬间完成的，是在一定的时间间隔内完成的。Scroller本身无法让View弹性滑动，它需要配合使用View的`computeScroll`方法才可以完成。



##Android中的图像格式
- A：透明度
- R：红色
- G：绿色
- B：蓝色

###Bitmap.Config
**1. ALPHA_8：**原图的每一个像素以半透明显示。只有透明度，没有颜色，每个像素点占8位，1字节

**2. ARGB_4444：**A=4，R=4，G=4，B=4，每个像素点占4+4+4+4=16位，2字节

**3. ARGB_8888：**A=8，R=8，G=8，B=8，每个像素点占8+8+8+8=32位，4字节

**3. RGB_565：**R=5，G=6，B=5，每个像素点占5+6+5=16位，2字节


##Bitmap的加载和Cache
目前比较常用的缓存策略是LruCache和DiskLruCache,其中，LruCache常被用作内存缓存，而DiskLruCache常被用作存储缓存。
>Lru：Least Recently Used的缩写，最近最少使用算法。

###Bitmap的高效加载
BitmapFactory类提供了四种方法加载图片：
`decodeFile`、`decodeResource`、`decodeStream`、`decodeByteArray`。


# Android系统启动流程
* 当系统引导程序启动Linux内核，内核会加载各种数据结构，和驱动程序，加载完毕之后，Android系统开始启动并加载第一个用户级别的进程：init（system/core/init/Init.c）

* 查看Init.c代码，看main函数

		int main(int argc, char **argv)
		{
	   		 ...
			//执行Linux指令
		    mkdir("/dev", 0755);
		    mkdir("/proc", 0755);
		    mkdir("/sys", 0755);
	
	      	...
	    	//解析执行init.rc配置文件
	    	init_parse_config_file("/init.rc");
			...
		}

* 在system\core\rootdir\Init.rc中定义好的指令都会开始执行，其中执行了很多bin指令，启动系统服务

		//启动孵化器进程，此进程是Android系统启动关键服务的一个母进程
		service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    	socket zygote stream 666
   	 	onrestart write /sys/android_power/request_state wake
    	onrestart write /sys/power/state on
    	onrestart restart media
    	onrestart restart netd

* 在app_process文件夹下找到app_main.cpp，查看main函数，发现以下代码

		int main(int argc, const char* const argv[])
		{
	   		...
			//启动一个系统服务：ZygoteInit
	        runtime.start("com.android.internal.os.ZygoteInit",startSystemServer);
			...
		}

* 在ZygoteInit.java中，查看main方法

		 public static void main(String argv[]) {
			...
			//加载Android系统需要的类
			preloadClasses();
			...
			if (argv[1].equals("true")) {
				//调用方法启动一个系统服务
                startSystemServer();
            }
			...
		}

* startSystemServer()方法的方法体

		String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,3001,3002,3003",
            "--capabilities=130104352,130104352",
            "--runtime-init",
            "--nice-name=system_server",
            "com.android.server.SystemServer",
        };

		...
		//分叉启动上面字符串数组定义的服务
		 pid = Zygote.forkSystemServer(
         parsedArgs.uid, parsedArgs.gid,
         parsedArgs.gids, debugFlags, null,
         parsedArgs.permittedCapabilities,
         parsedArgs.effectiveCapabilities);
* SystemServer服务被启动

		public static void main(String[] args) {
			...
			//加载动态链接库
			 System.loadLibrary("android_servers");
        	//执行链接库里的init1方法
			init1(args);
			...
		}

* 动态链接库文件和java类包名相同，找到com\_android\_server\_SystemServer.cpp文件
* 在com\_android\_server\_SystemServer.cpp文件中，找到了

		static JNINativeMethod gMethods[] = {
		    /* name, signature, funcPtr */
			//给init1方法映射一个指针，调用system_init方法
		    { "init1", "([Ljava/lang/String;)V", (void*) android_server_SystemServer_init1 },
		};

* android\_server\_SystemServer\_init1方法体中调用了system\_init()，system\_init()没有方法体
* 在system\_init.cpp文件中找到system\_init()方法，方法体中
		//执行了SystemServer.java的init2方法
		runtime->callStatic("com/android/server/SystemServer", "init2");

* 回到SystemServer.java，在init2的方法体中

		//启动一个服务线程
		Thread thr = new ServerThread();
        thr.start();

* 在ServerThread的run方法中
		
		//准备消息轮询器
		Looper.prepare();
		...
		//启动大量的系统服务并把其逐一添加至ServiceManager
		ServiceManager.addService(Context.WINDOW_SERVICE, wm);
		...
		//调用systemReady，准备创建第一个activity
		 ((ActivityManagerService)ActivityManagerNative.getDefault())
                .systemReady(new Runnable(){
				...
		}）；

* 在ActivityManagerService.java中，有systemReady方法，方法体里找到

		//检测任务栈中有没有activity，如果没有，创建Launcher
		mMainStack.resumeTopActivityLocked(null);
* 在ActivityStack.java中，方法resumeTopActivityLocked

		// Find the first activity that is not finishing.
        ActivityRecord next = topRunningActivityLocked(null);
        ...
        if (next == null) {
            // There are no more activities!  Let's just start up the
            // Launcher...
            if (mMainStack) {
                return mService.startHomeActivityLocked();
            }
        }
		...

---
#Handler消息机制
* Message类的obtain方法
	* 消息队列顺序的维护是使用单链表的形式来维护的
	* 把消息池里的第一条数据取出来，然后把第二条变成第一条

		   if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                sPoolSize--;
                return m;
            }

* 创建Handler对象时，在构造方法中会获取Looper和MessageQueue的对象

		public Handler() {
	        ...
			//拿到looper
	        mLooper = Looper.myLooper();
	        ...
			//拿到消息队列
	        mQueue = mLooper.mQueue;
	        mCallback = null;
    	}
	
* 查看myLooper方法体，发现Looper对象是通过ThreadLocal得到的，再查找ThreadLocal的set方法时发现
	* Looper是直接new出来的，并且在Looper的构造方法中，new出了消息队列对象

			sThreadLocal.set(new Looper());
	
			private Looper() {
		        mQueue = new MessageQueue();
		        mRun = true;
		        mThread = Thread.currentThread();
	   		}
	* sThreadLocal.set(new Looper())是在Looper.prepare方法中调用的

* prepare方法是在prepareMainLooper()方法中调用的

		public static final void prepareMainLooper() {
       		prepare();
			...
		}

* 在应用启动时，主线程要被启动，ActivityThread会被创建，在此类的main方法中

		public static final void main(String[] args) {
        	...
			//创建Looper和MessageQueue
        	Looper.prepareMainLooper();
			...
			//轮询器开始轮询
			Looper.loop();
			...
		}
* Looper.loop()方法中有一个死循环

		while (true) {
			//取出消息队列的消息，可能会阻塞
            Message msg = queue.next(); // might block
			...
			//解析消息，分发消息
			msg.target.dispatchMessage(msg);
			...
		}


* Linux的一个进程间通信机制：管道（pipe）。原理：在内存中有一个特殊的文件，这个文件有两个句柄（引用），一个是读取句柄，一个是写入句柄
* 主线程Looper从消息队列读取消息，当读完所有消息时，进入睡眠，主线程阻塞。子线程往消息队列发送消息，并且往管道文件写数据，主线程即被唤醒，从管道文件读取数据，主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠

* Handler发送消息，sendMessage的所有重载，实际最终都调用了sendMessageAtTime

		public boolean sendMessageAtTime(Message msg, long uptimeMillis)
    	{
	       ...
			//把消息放到消息队列中
            sent = queue.enqueueMessage(msg, uptimeMillis);
		   ...
		}

* enqueueMessage把消息通过重新排序放入消息队列

		final boolean enqueueMessage(Message msg, long when) {
	        ...
	        final boolean needWake;
	        synchronized (this) {
	           ...
				//对消息的重新排序，通过判断消息队列里是否有消息以及消息的时间对比
	            msg.when = when;
	            
	            Message p = mMessages;				
	            if (p == null || when == 0 || when < p.when) {
					// 如果消息队列中没有消息，或者当前消息的时候比队列中的消息的时间小，则让当前消息成为队列中的第一条消息
	                msg.next = p;
	                mMessages = msg;
	                needWake = mBlocked; // new head, might need to wake up
	            } else {		
					// 代码能进入到这里说明消息队列中有消息，且队列中的消息时间比当前消息时间小，说明当前消息不能做为队列中的第一条消息			
	                Message prev = null;	// 当前消息要插入到这个prev消息的后面
					// 这个while循环用于找出当前消息(msg)应该插入到消息列表中的哪个消息的后面（应该插入到prev这条消息的后面）
	                while (p != null && p.when <= when) {	
	                    prev = p;
	                    p = p.next;
	                }
					
					// 下面两行代码
	                msg.next = prev.next;
	                prev.next = msg;
	                needWake = false; // still waiting on head, no need to wake up
	            }
	        }
			//唤醒主线程
	        if (needWake) {
	            nativeWake(mPtr);
	        }
	        return true;
	    }

* Looper.loop方法中，获取消息，然后分发消息

		//获取消息队列的消息
		 Message msg = queue.next(); // might block
         ...
		//分发消息，消息由哪个handler对象创建，则由它分发，并由它的handlerMessage处理  
        msg.target.dispatchMessage(msg);

* message对象的target属性，用于记录该消息由哪个Handler创建，在obtain方法中赋值
* Handler中的Callback接口，可能过构造方法或其它方法传入
* Handler的dispatchMessage方法中：

		public void dispatchMessage(Message msg) {
	        if (msg.callback != null) {
	            handleCallback(msg);					// 第一优先Runnable对象
	        } else {
	            if (mCallback != null) {
	                if (mCallback.handleMessage(msg)) { // 第二优先Callback对象
	                    return;
	                }
	            }
	            handleMessage(msg);
	        }
    	}
* Message中保存了callback(Runnabe)和target(Handler)，也可以调用Message的sendToTarget()方法来发消息，前提是必须已经给Message设置了target对象。

---
#AsyncTask机制
* AsyncTask基本使用：

		AsyncTask<String, Integer, Object> asyncTask = new AsyncTask<String, Integer, Object>() {
			
			protected void onPreExecute() {};
			
			@Override
			protected Object doInBackground(String... params) {
				return null;
			}
			
			protected void onPostExecute(Object result) {};
			
			protected void onProgressUpdate(Integer[] values) {};
			
		};
		asyncTask.execute("params")

* AsyncTask的execute方法，开始执行异步任务，在此方法体中

		public final AsyncTask<Params, Progress, Result> execute(Params... params) {
	        ...
	
	        mStatus = Status.RUNNING;
	
			//调用onPreExecute方法
	        onPreExecute();
	
			//把参数赋值给mWorker对象
	        mWorker.mParams = params;
			//线程池对象执行mFuture
	        sExecutor.execute(mFuture);
	
	        return this;
	    }

* mWorker是什么类型？，在AsyncTask的构造方法中

		mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                return doInBackground(mParams);
            }
        };

* 然后把mWorker对象封装至FutureTask对象

		mFuture = new FutureTask<Result>(mWorker)

* 在FutureTask的构造中，又把mWorker封装给Sync对象

		public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        	sync = new Sync(callable);
    	}
* 在Sync的构造方法中

		Sync(Callable<V> callable) {
			//这里的callable就是mWorker
            this.callable = callable;
        }

* 线程池执行mFuture对象，此对象是FutureTask的对象，而FutureTask实现了Runnable接口

		public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        ...

		//线程池对象开一个子线程去执行mFuture对象中的run方法
        sExecutor.execute(mFuture);	
		...
        
    }

* mFuture的run方法被调用了

		public void run() {
        	sync.innerRun();
    	}

* 在innerRun方法中，调用了callable的call方法，但是在sync被new出来的时候，在构造方法中就已经把mWorker赋值给了callable，所以实际上是调用mWorker的call方法

		void innerRun() {
            ...
				//调用mWorker的call()
                result = callable.call();
                
                set(result);
            ...
        }

* mWorker的call在mWorker被new出来时就已经重写了

		mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
               ...
				//在子线程中调用了doInBackground方法
                return doInBackground(mParams);
            }
        };

* call方法调用完毕后，得到doInBackground所返回的result

		void innerRun() {
            ...
                result = callable.call();
                //返回的result传入了set方法
                set(result);
            ...
        }

* set方法体

		protected void set(V v) {
       		 sync.innerSet(v);
    	}

* innerSet方法体

		 if (compareAndSetState(s, RAN)) {
                    result = v;
                    releaseShared(0);
					//关键的done方法
                    done();
                    return;
          }
* innerSet方法是属于FutureTask类的，那么done方法也是调用FutureTask类的，这个done方法定义的地方，在AsyncTask.java的构造方法里

		mFuture = new FutureTask<Result>(mWorker) {
			//此处重写done方法
            @Override
            protected void done() {
                

               //获取doInbackground方法返回的结果
                result = get();

				//创建一个消息
                message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                        new AsyncTaskResult<Result>(AsyncTask.this, result));
				//把这条消息发送给创建这个消息的Handler：target.sendMessage(this)
                message.sendToTarget();
            }
        };

* 然后sHandler的handlerMessage被触发

		public void handleMessage(Message msg) {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    //调用finish方法
                    result.mTask.finish(result.mData[0]);
                    break;
               
            }
        }
* 查看AsyncTaskResult类中的mTask成员，其实它就是AsyncTask对象

* 再看AsyncTask对象的finish的方法体

		private void finish(Result result) {
       	 if (isCancelled()) result = null;
			//调用onPostExecute方法，并传入结果
        	onPostExecute(result);
        	mStatus = Status.FINISHED;
    	}





##1. 各种命令

###1. SDK介绍
- aapt：android application package tool
- adb：建立电脑与手机之间的链接
- dx.bat：将多个.class 打包成一个.dex
	
sdk下的目录：

- add-ons：预留的一个附加目录
- build-tools：构建工具目录
- *****docs：文档目录
- extras：开发中额外提供的一些工具及jar
- *****platforms：不同版本android的核心jar包
- *****platforms-tools：平台一些相关的工具
- *****sources：源码
- system-images：系统镜像文件
- tools：开发中使用的一些工具，如9path进行图片拉伸适配。
###2. DDMS介绍
ddms： dalvik debug manitor services		

- devices: 列出当前电脑所连接的所有android设备，及android设备运行的进程，结束一个进程，设置程序为debug模式，截屏。
- logcat: 会打印系统运行过程中所有日志信息。
- file explorer: 列出当前设备所有目录。

目录结构：

- /data/app:安装的第三方apk都在此目录
- /system/app: 系统预装应用apk在此目录  
- /data/data:应用的私有目录，系统每安装一个新的应用程序，都会在此目录创建该应用包名的文件，用来存放该应用的私有数据，当应用卸载时，该包名的文件夹也会被删除。  	
- /sdcard :外部存储目录，一般会链接指向到另一个目录，用来存放大数据。

###3. ADB指令 
**ADB：**android debug bridge，建立手机与电脑直接的连接，adb运行的端口号是5037。

环境变量的配置：C:\kaifa\adt-bundle-windows-x86_64_20140101\sdk\platform-tools

- adb devices :列出当前电脑所连接的android设备
- adb push pc_path  phone_path :将电脑端文件放到手机端
- adb pull phone_paht pc_path :将手机端文件拉到电脑端
- adb install [-r] apkpath ; 安装一个电脑端的apk文件。-r：强制安装
- adb uninstall packagename; 卸载一个应用
	
三个指令联合使用来解决adb被占用，或断开连接的情况

- adb kill-server : 结束adb服务的链接
- adb start-server ：开启adb服务的链接
- netstat -oan 查看端口: 查看端口  
	
- adb shell：进入当前设备linux环境下
- adb shell + ls -l ：查看当前设备的目录结构
- adb shell+ logcat :查看系统运行中的日志信息
	
>**注意：** 如果当前电脑链接的是多台android设备，需要指定操作的是哪台设备，需要在adb后加 -s 设备序列号。

###4. 测试的相关概念
SUV  好的软件不是开发出来的是测试出来的
####测试是否知道源代码
分为：黑盒测试（不知道代码）以及白盒测试（知道代码）
####按照测试的粒度
分为：方法测试&nbsp;&nbsp;&nbsp;&nbsp;单元测试（Junit）&nbsp;&nbsp;&nbsp;&nbsp;集成测试&nbsp;&nbsp;&nbsp;&nbsp;系统测试
####按照测试的暴力程度
分为：冒烟测试（硬件）以及压力测试（12306）


#Android的IPC机制
IPC是Inter-Process Communication的缩写，含义为进程间通信。

##进程跟线程
1. 线程是CPU调度的最小单元，是一种有限的系统资源。
2. 进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。
3. 在Android中，对单个应用所使用的最大内存做了限制。多进程可以加大可使用的内存。

##Android中的多进程模式
使用多进程的方法：

通过给四大组件指定android:process属性，可以轻易的开启多进程。

1. android:process=":remote"
2. android:process="com.jct.test.remote"
>区别：
>
>1. “：”的含义是指要在当前的进程名前面附加上当前的包名，后者是一种完整的命名方式，不会附加报名信息。
>2. 以“：”开头的进程属于当前应用的私有进程，其他应用的组件不可以和他跑在同一个进程中。而进程名不以“：”开头的进程属于全局进程，其他应用通过ShareUID方式可以和他跑在同一个进程中

Android系统会为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据。这里要说明的是，两个应用通过ShareUID跑在同一个进程中是有要求的，需要这两个应用有相同的ShareUID并且签名相同才可以。在这种情况下，它们可以互相访问对方的私有数据，比如data目录，组件信息等，不管他们是否跑在同一个进程中。当然如果它们跑在同一个进程中，那么除了能共享data目录、组件信息，还可以共享内存数据，或者说他们看起来就像是一个应用的两部分。
>ShareUID是自己设置的?

###多进程模式的运行机制
所有运行在不同进程中的四大组件，只要它们之间需要通过**内存**来共享数据，都会**共享失败**。

多进程带来的问题：

- 静态成员和单例模式完全失效（内存不同）
- 线程同步机制完全失效（内存不同)
- SharedPreferences的可靠性下降（SharedPreferences不支持两个进程间同时去执行写操作，否则会导致一定几率的数据丢失，这是因为它的底层是通过读写XML文件来实现的，并发写显然是可能出问题的）
- Application多次创建（运行在同一个进程中的组件是属于同一个虚拟机和同一个Application的，运行在不同进程中的组件是属于两个不同的虚拟机和Application的）
>我们也可以这么理解同一应用间的多进程：它就相当于两个不同的应用采用了SharedUID的模式。

**跨进程通信就是解决这个问题，实现数据交互。**

##IPC基础概念介绍
###Serializable接口
Serializable接口是Java所提供的一个序列化接口，是一个空接口。
类继承该接口后使用ObjectInputStream/ObjectOutputStream即可完成序列化反序列化。
>恢复后的对象和被序列化对象的内容完全一致，但是两者并不是一个对象。

在类中，`private static final long serialVersionUID = 45487845154L;`可以指定或者不指定。serialVersionUID是用来辅助序列化和反序列化过程的，原则上序列化后的数据中的serialVersionUID只有和当前类的serialVerisionUID相同才能正常地被序列化。我们指定后，可以在很大程度上避免反序列化过程的失败。但是我们不指定的时候，只要类结构发生变化，就会反序列化失败。
>静态成员变量属于类不属于对象，所以不会参加序列化过程。用transient关键字标记的成员变量也不参与序列化过程。

###Parcelable接口
Android系统为我们提供了许多实现了Parcelable接口的类，他们都是可以直接序列化的。比如Intent、Bundle、Bitmap，同时List和Map也可以序列化，前提是它们里面的每一个元素都可以序列化。

###两种序列化方法的区别：
1. Serializable是Java中的序列化接口，其用起来简单但是开销很大，序列化和反序列化过程需要大量的I/O操作。
2. Parcelable是Android中的序列化方式，因此更适合用在Android平台上，他的缺点就是使用起来麻烦的，但是它的效率很高。
3. Parcelable主要用在内存序列化上，通过Parcelable将对象序列化到存储设备中或者将对象序列化后通过网络传输也是可以的，但是这个过程稍显复杂，因此这两种情况推荐使用Serializable。


