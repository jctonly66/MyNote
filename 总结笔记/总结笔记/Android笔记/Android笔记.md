
##1.Android系统架构

分层的架构   

*	application :应用层 ； java	
*	application framework :应用框架层  ， java+JNI	
*	libraries 和 dalvik ： 函数库和虚拟机层，  c/c++
*	linux kernel : linux 内核驱动层， c

##2.两种虚拟机的不同
版权问题:  
jvm ： java虚拟机 sun  
dvm：  dalvik虚拟机  google    
  
区别：  
1. 基于的架构不同，jvm 基于栈架构，栈是位于内存上的一个空间，执行指令操作，需要向cpu寻址； dvm 基于寄存器架构,寄存器是cpu的一个组成部分，执行指令操作无需寻址直接执行。  
2. 执行文件的格式不同，jvm执行的是多个.class文件。 dvm执行的是一个.dex文件
	
##3.art 模式  android runtime
空间换时间的概念。  
art：程序在安装时需要预编译读取，将代码转换为机器码。  
好处：程序运行时，无需时时转换，运行速度快；  
缺点：安装时间稍长，由于转换机器码，所以占用略高的存储空间。

##4.SDK介绍
aapt：android application package tool  
adb：建立电脑与手机之间的链接  
dx.bat：将多个.class 打包成一个.dex
	
sdk下的目录：  

*	add-ons:预留的一个附加目录  
*	build-tools:构建工具目录
*	~docs: 文档目录
*	extras：开发中额外提供的一些工具及jar
*	~platforms: 不同版本android的核心jar包
*	~platforms-tools：平台一些相关的工具
*	~sources：源码
*	system-images：系统镜像文件
*	tools：开发中使用的一些工具，如9path，做图片拉伸适配的。


##5.DDMS介绍
DDMS： dalvik debug manitor services  

*	devices: 列出当前电脑所连接的所有android设备，及android设备运行的进程，结束一个进程，设置程序为debug模式，截屏。
*	 logcat: 会打印系统运行过程中所有日志信息。  
*	 file explorer: 列出当前设备所有目录。 
*	 /data/app:安装的第三方apk都在此目录
*	 /system/app: 系统预装应用apk在此目录  
*	 /data/data:应用的私有目录，系统每安装一个新的应用程序，都会在此目录创建该应用包名的文件，用来存放该应用的私有数据，当应用卸载时，该包名的文件夹也会被删除。  
*	 /sdcard :外部存储目录，一般会链接指向到另一个目录，用来存放大数据。


##6.Android的打包过程 
jdk---->dx.bat　　　　aapt---->签名jarsigner  
>.java -----> .class ------>.de(res,assets,androidmanifest.xml)------->.apk--------->final apk
	
##7.ADB指令 

ADB :android debug bridge 建立手机与电脑直接的连接  adb运行的端口号是5037

环境变量的配置：C:\kaifa\adt-bundle-windows-x86_64_20140101\sdk\platform-tools

*	adb devices :列出当前电脑所连接的android设备  
*	adb push pc_path  phone_path :将电脑端文件放到手机端
*	adb pull phone_paht pc_path :将手机端文件拉到电脑端
*	adb install [-r] apkpath ; 安装一个电脑端的apk文件。-r：强制安装
*	adb uninstall packagename; 卸载一个应用
	//三个指令联合使用来解决adb被占用，或断开连接的情况
*	adb kill-server : 结束adb服务的链接
*	adb start-server ：开启adb服务的链接
*	netstat -oan 查看端口: 查看端口  
*	adb shell：进入当前设备linux环境下
*	adb shell + ls -l ：查看当前设备的目录结构
*	adb shell+ logcat :查看系统运行中的日志信息
	
注意： 如果当前电脑链接的是多台android设备，需要指定操作的是哪台设备，需要在adb后加 -s 设备序列号。

##8.测试的相关概念
	
SUV  好的软件不是开发出来的是测试出来的
	

1. 测试是否知道源代码  
	黑盒测试　　不知道代码  
	白盒测试　　知道代码
2. 按照测试的粒度  
	方法测试  
	单元测试 Junit  
	集成测试  
	系统测试　
3. 按照测试的暴力程度  
	冒烟测试  硬件  
	压力测试 12306  

monkey测试： adb shell下的一个测试指令。  
adb shell +　monkey -p packagename count;
	
##9.数据存储
1. 通过context对象获取私有目录/data/data/packagename/filse  
	
		context.getFileDir().getPath()；  
2. 硬性编码问题：通过 Environment可以获取sdcard的路径  
	
		Environment.getExternalStorageDirectory().getPath();
3. 使用前需要判断sdcard状态
	
		if(!Environment.getExternalStorageState().equals( Environment.MEDIA_MOUNTED)){
			//sdcard状态是没有挂载的情况
			Toast.makeText(mContext, "sdcard不存在或未挂载", Toast.LENGTH_SHORT).show();
			return ;
		}
4. 需要判断sdcard剩余空间  

		//判断sdcard存储空间是否满足文件的存储
		File sdcard_filedir = Environment.getExternalStorageDirectory();
		//得到sdcard的目录作为一个文件对象
		long usableSpace = sdcard_filedir.getUsableSpace();
		//获取文件目录对象剩余间
		long totalSpace = sdcard_filedir.getTotalSpace();
		//将一个long类型的文件大小格式化成用户可以看懂的M，G字符串
		String usableSpace_str = Formatter.formatFileSize(mContext, usableSpace);
		String totalSpace_str = Formatter.formatFileSize(mContext, totalSpace);
		if(usableSpace < 1024 * 1024 * 200){//判断剩余空间是否小于200M
			Toast.makeText(mContext, "sdcard剩余空间不足,无法满足下载；剩余空间为："+usableSpace_str, Toast.LENGTH_SHORT).show();
		return ;	

两种方式区别：  
/data/data:`context.getFileDir().getPath()`:是一个应用程序的私有目录，只有当前应用程序有权限访问读写，其他应用无权限访问。一些安全性要求比较高的数据存放在该目录，一般用来存放size比较小的数据。

/sdcard:`Enviroment.getExternalStorageDirectory().getPath()`:是一个外部存储目录，只用应用声明了权限，就可以访问读写sdcard目录；所以一般用来存放一些安全性不高的数据，文件size比较大的数据。  
`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>`

##10.SharedPreferences介绍（用来做数据存储）

sharedPreferences是通过xml文件来做数据存储的。一般用来存放一些标记性的数据，一些设置信息。

	*********使用sharedPreferences存储数据
		
	1.通过Context对象创建一个SharedPreference对象
	//name:sharedpreference文件的名称    mode:文件的操作模式
		SharedPreferences sharedPreferences = context.getSharedPreferences("userinfo.txt", Context.MODE_PRIVATE);
	2.通过sharedPreferences对象获取一个Editor对象
		Editor editor = sharedPreferences.edit();
	3.往Editor中添加数据
		editor.putString("username", username);
		editor.putString("password", password);
	4.提交Editor对象
		editor.commit();

	*********使用sharedPreferences读取数据

	1.通过Context对象创建一个SharedPreference对象
		SharedPreferences sharedPreferences = context.getSharedPreferences("userinfo.txt", Context.MODE_PRIVATE);
				
	2.通过sharedPreference获取存放的数据
		//key:存放数据时的key   defValue: 默认值,根据业务需求来写
		String username = sharedPreferences.getString("username", "");
		String password = sharedPreferences.getString("password", "");

	3.通过PreferenceManager可以获取一个默认的sharepreferences对象		
		SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
		
##11.文件的权限概念 

通过context对象获取一个私有目录的文件读取流  /data/data/packagename/files/userinfoi.txt  

	FileInputStream fileInputStream = context.openFileInput("userinfo.txt");  
通过context对象得到私有目录下一个文件写入流； name : 私有目录文件的名称    mode： 文件的操作模式， 私有，追加，全局读，全局写

	FileOutputStream fileOutputStream = context.openFileOutput("userinfo.txt", Context.MODE_PRIVATE);
linux下一个文件的权限由10位标示：  

*	1位：文件的类型，d：文件夹 l:快捷方式  -:文件
*	2-4： 该文件所属用户对本文件的权限 ， rwx ：用二进制标示，如果不是-就用1标示，是-用0标示；chmod指令赋权限。
*	5-7：该文件所属用户组对本文件的权限
*	8-10：其他用户对该文件的权限。
##12.android 6.0以上运行时权限
在android 6.0以上，需要在运行时检查APP是都否已经拥有所需权限。没有则需要申请。

	if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
              != PackageManager.PERMISSION_GRANTED) {
          //申请WRITE_EXTERNAL_STORAGE权限
          ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                  WRITE_EXTERNAL_STORAGE_REQUEST_CODE);
      }
在用户选择允许后，系统回调onRequestPermissionsResult()方法，
##13.使用pull解析xml格式的数据
dom解析：基于全文加载的解析方式   sax解析：基于事件的逐行解析方式  pull解析：同sax

##14.Android下数据库创建 
	
什么情况下我们才用数据库做数据存储？ 大量数据结构相同的数据需要存储时。

创建数据库步骤：    

1. 创建一个类集成SqliteOpenHelper，需要添加一个构造方法，实现两个方法oncreate ,onupgrade  
构造方法中的参数介绍：  

		//context :上下文   ， name：数据库文件的名称    factory：用来创建cursor对象，默认为null 
		//version:数据库的版本号，从1开始，如果发生改变，onUpgrade方法将会调用,4.0之后只能升不能将
		super(context, "info.db", null,1);
 
2. 创建这个帮助类的一个对象，调用getReadableDatabase()方法，会帮助我们创建打开一个数据库
3. 复写oncreate和onupgrdate方法：  
	*	oncreate方法是数据库第一次创建的时候会被调用;  特别适合做表结构的初始化,需要执行sql语句；SQLiteDatabase db可以用来执行sql语句
	*	onUpgrade数据库版本号发生改变时才会执行； 特别适合做表结构的修改

帮助类对象中的getWritableDatabase 和 getReadableDatabase都可以帮助我们获取一个数据库操作对象SqliteDatabase.

区别：  
getReadableDatabase:
	先尝试以读写方式打开数据库，如果磁盘空间满了，他会重新尝试以只读方式打开数据库。  
getWritableDatabase:
	直接以读写方式打开数据库，如果磁盘空间满了，就直接报错。

数据库操作：

1. 创建一个帮助类的对象，调用getReadableDatabase方法，返回一个SqliteDatebase对象  
2. 使用SqliteDatebase对象调用execSql()做增删改,调用rawQuery方法做查询。
	******特点:增删改没有返回值，不能判断sql语句是否执行成功。sql语句手动写，容易写错  
3. 使用SqliteDatebase对象调用insert,update,delete ,query方法做增删改查。

	******特点:增删改有了返回值，可以判断sql语句是否执行成功，但是查询不够灵活，不能做多表查询。所以在公司一般人增删改喜欢用第二种方式，查询用第一种方式。
##15.数据库的事务 
		
事务： 执行多条sql语句，要么同时执行成功，要么同时执行失败，不能有的成功，有的失败
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

##16.常用获取inflate的写法 

1. 
	`view = View.inflate(context, R.layout.item_news_layout, null);//将一个布局文件转换成一个view对象`

2. `view =  LayoutInflater.from(context).inflate(R.layout.item_news_layout, null);`
			
3. `LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);`
	`view = layoutInflater.inflate(R.layout.item_news_layout, null);`
					
##17.消息机制原理

有几个主要元素：

*	Message:用来携带子线程中的数据。
*	MessageQueue:用来存放所有子线程发来的Message.
*	Handler:用来在子线程中发送Message，在主线程中接受Message，处理结果
*	Looper:是一个消息循环器，一直循环遍历MessageQueue，从MessageQueue中取一个Message，派发给Handler处理。
*	`Message msg = Messge.obtain;`
	可控制msg对象的数量只有一个。
				
##18.常见消息处理api

问：子线程一定不能更新UI？  
答：  
SurfaceView ：多媒体视频播放 ,可以在子线程中更新UI；   
Progress（进度）相关的控件：也是可以在子线程中更新Ui；  
审计机制：activity完全显示的时候审计机制才会去检测子线程有没有更新Ui.

1. 使用activity的runOnUiThread方法更新ui,无论当前线程是否是主线程，都将在主线程执行.  

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

	应用场景:广告展示后，做页面跳转。

##19.get方式和post方式：

1. 请求的URL地址不同：
2. 请求头不同：  
	post方式多了几个请求头：Content-Length   Cache-Control  Origin

		openConnection.setRequestProperty("Content-Length", body.length()+"");
		openConnection.setRequestProperty("Cache-Control", "max-age=0");
		openConnection.setRequestProperty("Origin", "http://192.168.13.83:8080");
		****post方式还多了请求的内容：username=root&pwd=123

		//设置UrlConnection可以写请求的内容
		openConnection.setDoOutput(true);
		//获取一个outputstream，并将内容写入该流
		openConnection.getOutputStream().write(body.getBytes());
3. 请求时携带的内容大小不同  
	get:1k 　post:理论无限制

4. 使用POST方式提交数据时的中文乱码解决方法

   	解决办法：使用客户端和服务器两边的字符集编码保持一致。
   	UTF-8，
5. 使用GET方式提交数据的中文乱码的解决方法  
   使用URLEncoder.encode(name,"UTF-8")进行url编码：

		String path = "http://192.168.22.136:8080/web/servlet/LoginServlet
		?username="+URLEncoder.encode(name,"UTF-8")+"&password="+URLEncoder.encode(pwd,"UTF-8");
##20.多线程断点下载（java）
步骤：  

1. 在客户端创建一个与服务器端大小一样的空白文件  
2. 设置子线程的个数  
3. 计算每个子线程下载的数据块大小和下载起始位置、结束位置  
4. 创建子线程开始下载数据  
5. 得到每个子线程都下载完成的标记  


		代码：
		MultiThreadDownLoader.java:

		import java.io.RandomAccessFile;
		import java.net.HttpURLConnection;
		import java.net.URL;

		public class MultiThreadDownLoader {
	
		//2、使用的子线程的个数
		private static int threadCount =3;

		public static void main(String[] args) {
			try {
				String path = "http://192.168.22.136:8080/sogou.exe";
				URL url = new URL(path);
				HttpURLConnection conn = (HttpURLConnection) url.openConnection();
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(3000);
				
				int code = conn.getResponseCode();
				if(code == 200){
					int length =	conn.getContentLength();
					//1、在客户端创建一个与服务端文件一样大小的文件
					RandomAccessFile file = new RandomAccessFile("temp.exe", "rw");
					file.setLength(length);
					//3、每个子线程下载数据块 ,下载的起始位置和结束位置
					int blockSize = length/threadCount;
				
					// threadId * blockSize  ---- （threadId+1）* blockSize -1
					for(int threadId =0; threadId < threadCount; threadId++){
						//下载的起始位置和结束位置
						int startIndex = threadId * blockSize;
						int endIndex = 0;
					
						if(threadId != (threadCount -1)){
							 endIndex = (threadId + 1) * blockSize - 1;
						}else{
							endIndex = length-1;
						}
					
						//开启子线程下载数据
					new ThreadDownLoader(path, startIndex, endIndex, threadId, threadCount).start();
					
				}
					
				}else{
					//抛出异常
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
			}
		}
	
		ThreadDownLoader.java:
		public class ThreadDownLoader extends Thread {
	
		private String path;
		private int startIndex;
		private int endIndex;
		private int threadId;
		private int threadCount;
		//正在运行的线程个数
		private static  int runningThreadCount = 3; 
		ThreadDownLoader(String path,int startIndex,int endIndex,int threadId,int threadCount){
			this.path = path;
			this.startIndex = startIndex;
			this.endIndex = endIndex;
			this.threadId = threadId;
			this.threadCount = threadCount;
			
		}
		
		@Override
		public void run() {
			downLoad(path,startIndex,endIndex,threadId,threadCount);
		}
		
		public void downLoad(String path,int startIndex,int endIndex,int threadId,int threadCount){
			try {
			    URL url = new URL(path);
				HttpURLConnection conn = (HttpURLConnection) url.openConnection();
				
				conn.setRequestMethod("GET");
				conn.setConnectTimeout(3000);
				
				//获取上一次下载的位置，接着下载
				File threadFile = new File(threadId+".txt");
				if(threadFile.exists() && threadFile.length() > 0){
					FileReader fr = new FileReader(threadFile);
					BufferedReader br = new BufferedReader(fr);
					String position = br.readLine();
					//设置子线程请求数据的范围
					conn.setRequestProperty("Range", "bytes="+position+"-"+endIndex);
					startIndex = Integer.valueOf(position);
				}else{
					//设置子线程请求数据的范围
					conn.setRequestProperty("Range", "bytes="+startIndex+"-"+endIndex);
				}
							
				int code = conn.getResponseCode();
				if(code == 206){//请求部分数据
					
					InputStream is = conn.getInputStream();
					RandomAccessFile file = new RandomAccessFile("temp.exe","rw");
				  //指定从哪个位置开始写数据
					file.seek(startIndex);
				  
				  	byte[] buffer = new byte[1024*1024];
				    int len = -1;
				    int total = 0;
				    while((len = is.read(buffer)) != -1){
						file.write(buffer, 0, len);
					    total = total + len;
					    int currentPosition = startIndex + total;
					    //能够即时的把数据写到数据
					    RandomAccessFile f = new RandomAccessFile(threadId+".txt", "rwd");
					    f.write((currentPosition+"").getBytes());
					    f.close(); 
				    }
				  file.close();
				  System.out.println("线程"+threadId+"下载完成...............");
				  synchronized (ThreadDownLoader.this) {
					  runningThreadCount--;
					  if(runningThreadCount == 0){
						  System.out.println("文件下载完成...............");
					  }
				}
				}
		} catch (Exception e) {
			 System.out.println("线程"+threadId+"下载失败...............");
			e.printStackTrace();
		}
		  
	  }
	}
##21.横竖屏切换Activity的生命周期

这个生命周期跟清单文件里的配置有关系。

1. 不设置Activity的android:configChanges时，切屏会重新调用各个生命周期
默认首先销毁当前activity,然后重新加载  

2. 为了防止横竖屏切换 生命周期会发生变化 所以把Activity配置如下    `android:screenOrientation="portrait"` 

3. 设置Activity的android:configChanges="orientation|keyboardHidden|screenSize"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法  

游戏开发中, 屏幕的朝向都是写死的. 

问：如何将一个Activity设置成窗口的样式。  
答：可以自定义一个activity的样式  
 	`android:theme="@android:style/Theme.Dialog"`

##22.Android中Task任务栈的分配。
首先我们来看下Task的定义，Google是这样定义Task的： 
 
a task is what the user experiences as an "application." It's a group of related activities, arranged in a stack. A task is a stack of activities, not a class or an element in the manifest file.  

这意思就是说Task实际上是一个Activity栈，通常用户感受的一个Application就是一个Task。从这个定义来看，Task跟Service或者其他Components是没有任何联系的，它只是针对Activity而言的。  
Activity有不同的启动模式, 可以影响到task的分配  

Task，简单的说，就是一组以栈的模式聚集在一起的Activity组件集合。它们有潜在的前后驱关联，新加入的Activity组件，位于栈顶，并仅有在栈顶的Activity，才会有机会与用户进行交互。而当栈顶的Activity完成使命退出的时候，Task会将其退栈，并让下一个将跑到栈顶的Activity来于用户面对面，直至栈中再无更多Activity，Task结束。

eg:

*	点开Email应用，进入收件箱（Activity A）	A
*	选中一封邮件，点击查看详情（Activity B）	AB
*	点击回复，开始写新邮件（Activity C）	ABC
*	写了几行字，点击选择联系人，进入选择联系人界面（Activity D）	ABCD
*	选择好了联系人，继续写邮件	ABC
*	写好邮件，发送完成，回到原始邮件	AB
*	点击返回，回到收件箱	A
*	退出Email程序	null

如上表所示，是一个实例。从用户从进入邮箱开始，到回复完成，退出应用整个过程的Task栈变化。这是一个标准的栈模式，对于大部分的状况，这样的Task模型，足以应付，但是，涉及到实际的性能、开销等问题，就会变得残酷许多。

比如，启动一个浏览器，在Android中是一个比较沉重的过程，它需要做很多初始化的工作，并且会有不小的内存开销。但与此同时，用浏览器打开一些内容，又是一般应用都会有的一个需求。设想一下，如果同时有十个运行着的应用（就会对应着是多个Task），都需要启动浏览器，这将是一个多么残酷的场面，十个Task栈都堆积着很雷同的浏览器Activity，是多么华丽的一种浪费啊。

于是你会有这样一种设想，浏览器Activity，可不可以作为一个单独的Task而存在，不管是来自那个Task的请求，浏览器的Task，都不会归并过去。这样，虽然浏览器Activity本身需要维系的状态更多了，但整体的开销将大大的减少，这种舍小家为大家的行为，还是很值得歌颂的。

standard", "singleTop", "singleTask", "singleInstance":

*	standard模式， 是默认的也是标准的Task模式，在没有其他因素的影响下，使用此模式的Activity，会构造一个Activity的实例，加入到调用者的Task栈中去，对于使用频度一般开销一般什么都一般的Activity而言，standard模式无疑是最合适的，因为它逻辑简单条理清晰，所以是默认的选择。

*	而singleTop模式，基本上于standard一致，仅在请求的Activity正好位于栈顶时，有所区别。此时，配置成singleTop的Activity，不再会构造新的实例加入到Task栈中，而是将新来的Intent发送到栈顶Activity中，栈顶的Activity可以通过重载onNewIntent来处理新的Intent（当然，也可以无视...）。这个模式，降低了位于栈顶时的一些重复开销，更避免了一些奇异的行为（想象一下，如果在栈顶连续几个都是同样的Activity，再一级级退出的时候，这是怎么样的用户体验...），很适合一些会有更新的列表Activity展示。一个活生生的实例是，在Android默认提供的应用中，浏览器（Browser）的书签Activity（BrowserBookmarkPage），就用的是singleTop。

*	singleTask，和singleInstance，则都采取的另辟Task的蹊径。标志为singleTask的Activity，最多仅有一个实例存在，并且，位于以它为根的Task中。所有对该Activity的请求，都会跳到该Activity的Task中展开进行。singleTask，很象概念中的单件模式，所有的修改都是基于一个实例，这通常用在构造成本很大，但切换成本较小的Activity中。最典型的例子，还是浏览器应用的主Activity（名为Browser...），它是展示当前tab，当前页面内容的窗口。它的构造成本大，但页面的切换还是较快的，于singleTask相配，还是挺天作之合的。
如果一个activity的创建需要占用大量的系统资源（cpu，内存）一般配置这个activity为singletask的启动模式。

*	singleInstance显得更为极端一些。在大部分时候singleInstance与singleTask完全一致，唯一的不同在于，singleInstance的Activity，是它所在栈中仅有的一个Activity，如果涉及到的其他Activity，都移交到其他Task中进行。这使得singleInstance的Activity，像一座孤岛，彻底的黑盒，它不关注请求来自何方，也不计较后续由谁执行。在Android默认的各个应用中，很少有这样的Activity，在我个人的工程实践中，曾尝试在有道词典的快速取词Activity中采用过，是因为我觉得快速取词入口足够方便（从notification中点选进入），并且会在各个场合使用，应该做得完全独立。

默认任务栈名字为包名, 可以自己配置Activity的android:affinity属性  
1. 配置后 当启动这个activity时就先去找有没有activity的亲和力属性相同。有就加入这个activity所在的任务中没有就新开任务栈  
2. affinity起作用需要的条件二者具备一个:  

*	intent包含FLAG_ACTIVITY_NEW_TASK标记
*	activity元素启用了allowTaskReparenting属性.

##23.特殊广播接收者

比如操作特别频繁的广播事件 屏幕的锁屏和解锁 电池电量的变化 这样的广播接收者在清单文件里面注册无效

	import android.os.Bundle;
	import android.app.Activity;
	import android.content.IntentFilter;
	import android.view.Menu;

	public class MainActivity extends Activity {
	private ScreenReceiver screenReceiver;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
     
		//[1]动态的去注册屏幕解锁和锁屏的广播
		screenReceiver = new ScreenReceiver();
		//[2]创建intent-filter对象
		IntentFilter filter = new IntentFilter();
		//[3]添加要注册的action
		filter.addAction("android.intent.action.SCREEN_OFF");
		filter.addAction("android.intent.action.SCREEN_ON");
		//[4]注册广播接收者
		this.registerReceiver(screenReceiver, filter);	
	}
	@Override
	protected void onDestroy() {
		//当activity销毁的时候  取消注册广播接收者
		unregisterReceiver(screenReceiver);
		
		super.onDestroy();
	}}

##24.样式和主题
style  Theme  

1. 共同点:  
定义的方式是一样的   
2. 不同点:  
style作用范围比较窄 (控件 button textview)    
theme 作用在activity或者Application节点下  
##25.进程概念介绍
四大组件都是运行在主线程  
Android中的服务 也是在后台运行  可以理解成是在后台运行并且是没有界面的Activity  

*	Foreground process:前台进程  用户正在交互,可以理解成相当于Activity执行onResume方法
*	Visible process: 可视进程 用户没有在交互,但用户还一直能看得见页面,相当于Activity执行了onPause方法
*	Service Process:  服务进程 通过startService()开启了一个服务
*	Background process:  后台进程  当前用户看不见页面,相当于Activity执行了onStop方法
*	Empty process: 空进程

##26.服务的特点
服务是在后台运行 可以理解成是没有界面的activity  
定义四大组件的方式都是一样的    
定义一个类继承Service
  
特点:

1. 服务通过startservice方式开启 第一次点击按钮开启服务 会执行服务的onCreate 和 onStart方法
2. 如果第二次开始在点击按钮开启服务 服务之后执行onStrat方法
3. 服务被开启后 会在设置页面里面的running里面找得到这个服务
4. startservice 方式开启服务 服务就会在后台长期运行 直到用户手工停止 或者调用StopService方法 服务才会被销毁
 
bindService 方式开启服务的特点 (为了调用Service里面的方法):
  
1. 当点击按钮第一次开启服务 会执行服务的onCreate方法 和 onBind()方法
2. 当我第二次点击按钮在调用bindservice  服务没有响应
3. 当activity销毁的时候服务也销毁  不求同时生但求同时死 
4. 通过bind方式开启服务  服务不能再设置页面里面找到  相当于是一个隐形的服务
5. bindservice不能多次解绑 多次解绑会报错

##27.使用服务注册特殊的广播接收者
1. 创建我们要注册的广播接收者 

		public class ScreenReceiver extends BroadcastReceiver {
			@Override
			public void onReceive(Context context, Intent intent) {		//获取广播事件的类型
				String action = intent.getAction();
				if ("android.intent.action.SCREEN_OFF".equals(action)) {	System.out.println("说明屏幕锁屏了");
				}else if("android.intent.action.SCREEN_ON".equals(action)){	System.out.println("说明屏幕解锁了");
				}
			}
		}
2. 创建一个服务,用来注册广播接收者

		public class ScreenService extends Service {
		private ScreenReceiver receiver;
			@Override
			public IBinder onBind(Intent intent) {
				return null;
			}
			//当服务第一次启动的时候调用
			@Override
			public void onCreate() {
			//在这个方法里面注册广播接收者
			//[1]获取ScreenReceiver实例
			receiver = new ScreenReceiver();
			//[2]创建IntentFilter对象
			IntentFilter filter = new IntentFilter();
			//[3]添加注册的事件
			filter.addAction("android.intent.action.SCREEN_OFF");		filter.addAction("android.intent.action.SCREEN_ON");
			//[4]通过代码的方式注册
			registerReceiver(receiver, filter);
			super.onCreate();
			}
			//当服务销毁的时候调用
			@Override
			public void onDestroy() {
			//当actvivity销毁的时候  取消注册广播接收者
			unregisterReceiver(receiver);
			super.onDestroy();
			}
		}
		(3)一定要记得配置service

##28.通过接口方式调用服务里面的方法
 	接口可以隐藏代码内部的细节 让程序员暴露自己只想暴露的方法
 	(6)定义一个接口 把想暴露的方法都定义在接口里面 
  	(7)我们定义的中间人对象 实现我们定义的接口
 	(8)在获取我们定义的中间人对象方式变了

##29.aidl介绍
*	远程服务 运行在其他应用里面的服务 
*	本地服务 运行在自己应用里面的服务
*	进行进程间通信  IPC
*	aidl Android interface Defination Language: 
	Android接口定义语言,专门是用来解决进程间通信的 
  
aidl 实现步骤和之前调用服务里面的方法的区别 

1. 先把Iservice.java文件变成aidl文件
2. adil 不认识public 把public 给我去掉
3. 会自动生成一个Stub类 实现ipc
4. 我们定义的中间人对象 直接继承stub
5. 想要保证2个应用程序的aidl文件是同一个 要求aidl文件所在包名相同
6. 获取中间人对象Stub.asinterface(Ibinder obj) 

##30.实现内容提供者步骤
1. 定义内容提供者 定义一个类继承contentProvider
2. 在清单文件里面配置一下
3. 定义一个urimatcher
4. 写一个静态代码块 添加匹配规则
5. 按照我们添加的匹配规则 暴露想暴露的方法
6. 如果你发现如下log日志 就说明内容提供者写的没有问题
`09-11 02:02:31.142: I/ActivityThread(16636): Pub com.itheima.provider: com.itheima.db.AccountProvider  `
7. 只要是通过内容提供者暴露出来的数据 其他应用访问的方式都是一样的 就是通过内容解析者 

##31.计算机表示图形的几种方式
多媒体:(包含文字 图片 音频 视频)  
图形的大小 = 图片的总像素 * 每个像素的大小  

*	单色：每个像素最多可以表示2种颜色，只需要使用长度为1的二进制位来表示，那么每个像素占1/8byte
*	16色：每个像素最多可以表示16种颜色，0000 - 1111，那么只需要使用长度为4的二进制表示，那么每个像素占1/2个byte
*	256色：每个像素最多可以表示256种颜色，0000 0000 - 1111 1111，那么只需要使用长度8的二进制位表示 那么每个像素占1byte 

		24位 rgb 
   		r 1byte   0-255
    	g 1byte  0-255
    	b 1byte  0-255     那么一个像素占3byte 
    	jpg 格式
    	png 格式 Android采用的是png格式 
##32.Activity 生命周期。
生命周期描述的是一个类：从创建(new出来)到死亡(垃圾回收)的过程中会执行的方法..  
在这个过程中，会针对不同的生命阶段会调用不同的方法。   
Activity从创建到销毁有多种状态，从一种状态到另一种状态时会激发相应的回调方法，这些回调方法包括：  
	`oncreate ondestroy`   
	`onstop onstart`  
	`onresume onpause `  
其实这些方法都是两两对应的，onCreate创建与onDestroy销毁；onStart可见与onStop不可见；onResume可编辑（即焦点）与onPause；  
这6个方法是相对应的，那么就只剩下一个onRestart方法了，这个方法在什么时候调用呢？
答案就是：在Activity被onStop后，但是没有被onDestroy，在再次启动此Activity时就调用onRestart（而不再调用onCreate）方法；如果被onDestroy了，则是调用onCreate方法。

##33.两个Activity之间跳转时必然会执行的方法。
一般情况比如说有两个activity,分别叫A,B ,当在A里面激活B组件的时候, A会调用onPause()方法,然后B.调用onCreate() ,onStart(), OnResume() ,

这个时候B覆盖了窗体, A会调用onStop()方法.  如果B呢 是个透明的,或者是对话框的样式, 就不会调用onStop()方法

##34.你后台的Activity被系统 回收怎么办？如果后台的Activity由于某原因被系统回收可了，如何在被系统回收之前保存当前状态？
	
除了在栈顶的activity,其他的activity都有可能在内存不足的时候被系统回收,一个activity越处于栈底,被回收的可能性越大.

	protected void onSaveInstanceState(Bundle outState) {
		super.onSaveInstanceState(outState);
		outState.putLong("id", 1234567890);
	}

	public void onCreate(Bundle savedInstanceState) {
	//判断 savedInstanceState是不是空.
	//如果不为空就取出来
        super.onCreate(savedInstanceState);
	}
也可以每隔一段时间保存一次, 保存到本地, 下次启动时恢复.

##35.如何退出Activity？如何安全退出已调用多个Activity的Application？

退出activity 直接调用 finish () 方法 . //用户点击back键 就是退出一个activity   
退出activity 会执行 onDestroy()方法 .  

1. 抛异常强制退出：
该方法通过抛异常，使程序Force Close。   
验证可以，但是，需要解决的问题是，如何使程序结束掉，而不弹出Force Close的窗口。
	100/0
	//安全结束进程  
	`android.os.Process.killProcess(android.os.Process.myPid());`
2. 记录打开的Activity：
每打开一个Activity，就记录下来。在需要退出时，关闭每一个Activity即可。
	
		List<Activity> lists ; 在application 全集的环境里面 
		lists = new ArrayList<Activity>();

		lists.add(this);

		for(Activity activity: lists)
		{
			activity.finish();
		}

		ondestory
		lists.remove(this);

3. 发送特定广播
   在需要结束应用时，发送一个特定的广播，每个Activity收到广播后，关闭即可。  
   给某个activity 注册接受接受广播的意图 `registerReceiver(receiver, filter)`
   如果过接受到的是,关闭activity的广播 ,就调用finish()方法,把当前的activity finish()掉 
4. 递归退出
   在打开新的Activity时使用startActivityForResult，然后自己加标志，在onActivityResult中处理，递归关闭。
5. 上面是网上的一些做法.  
   还可以通过 intent的flag 来实现.. intent.setFlag(FLAG_ACTIVITY_CLEAR_TOP)激活一个新的activity,然后在新的activity的oncreate方法里面 finish掉, 但是一个应用可以有多个任务栈, 这样可能会有问题.
##36.service是否在main thread中执行, service里面是否能执行耗时的操作?
默认情况,如果没有显示的指定service所运行的进程, Service和activity是运行在当前app所在进程的main thread(UI主线程)里面    
service里面不能执行耗时的操作(网络请求,拷贝数据库,大文件)  
在子线程中执行 `new Thread(){}.start();`  `Thread.currentThread().getName();`  
特殊情况 ,可以在清单文件配置 service 执行所在的进程 ,让service在另外的进程中执行`ActivityManagerService`
##37.序列化对象
让对象实现implements Serializable接口把对象存放到文件上.    
让类实现Serializable接口,然后可以通过ObjectOutputStream  

 	//对象输出流  
	File file = new File("c:\1.obj");
	FileOutputStream fos  = new FileOutputStream(file);
	ObjectOutputStream oos = new ObjectOutputStream(fos);
	Student stu = new Student();
	oos.writeObject(stu);
	//从文件中把对象读出来  
	ObjectInputStream ois = new ObjectInputStream(arg0);
	Student stu1 = (Student) ois.readObject();
	Paceable: 存入内存
	文件/网络 
	intent.setData(Uri) 
	Uri.fromFile();  //大图片的传递
##38.Activity怎么和service绑定，怎么在activity中启动自己对应的service？*
*	startService() 一旦被创建，调用者无关，没法使用service里面的方法 
*	 bindService () 把service与调用者绑定，如果调用者被销毁，service会销毁 ，可以使用service 里面的方法


		//构建一个intent对象,
		Intent service = new Intent(this,MyService.class);
 		//通过bindService的方法去启动一个服务,
	   	bindService(intent, new MyConn(), BIND_AUTO_CREATE);
		//ServiceConnection对象(重写onServiceConnected和OnServiceDisconnected方法)和BIND_AUTO_CREATE.
		private class myconn implements ServiceConnection
		{
			public void onServiceConnected(ComponentName name, IBinder service) {
				// TODO Auto-generated method stub
				//可以通过IBinder的对象 去使用service里面的方法
			}
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub	
			}
		}
##39.什么是Service以及描述下它的生命周期。Service有哪些启动方法，有什么区别，怎样停用Service？
在Service的生命周期中，被回调的方法比Activity少一些，只有onCreate, onStart, onDestroy, onBind和onUnbind。  
通常有两种方式启动一个Service,他们对Service生命周期的影响是不一样的。  

1. 通过startService  
	Service会经历 onCreate 到onStart，然后处于运行状态，stopService的时候调用onDestroy方法。
   	如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行。
2. 通过bindService   
    Service会运行onCreate，然后是调用onBind， 这个时候调用者和Service绑定在一起。调用者退出了，Srevice就会调用onUnbind->onDestroyed方法。  
   	所谓绑定在一起就共存亡了。调用者也可以通过调用unbindService方法来停止服务，这时候Srevice就会调用onUnbind->onDestroyed方法。  
	一个原则是Service的onCreate的方法只会被调用一次，就是你无论多少次的startService又bindService，Service只被创建一次。如果先是bind了，那么start的时候就直接运行Service的onStart方法，如果service运行期间调用了bindService，这时候再调用stopService的话，service是不会调用onDestroy方法的，service就stop不掉了，只能调用UnbindService，service就会被销毁。

	如果一个service通过startService 被start之后，多次调startService 的话，service会多次调用onStart方法。多次调用stopService的话，service只会调用一次onDestroyed方法。
	如果一个service通过bindService被start之后，多次调用bindService的话，service只会调用一次onBind方法。  
	多次调用unbindService的话会抛出异常。
##40.什么是IntentService？有何优点？
普通的service ,默认运行在uimain主线程  
Sdk给我们提供的方便的,带有异步处理的service类,必须有无参构造方法
`OnHandleIntent()` 处理耗时的操作
##41.什么时候使用Service？
拥有service的进程具有较高的优先级  
官方文档告诉我们，Android系统会尽量保持拥有service的进程运行，只要在该service已经被启动(start)或者客户端连接(bindService)到它。当内存不足时，需要保持，拥有service的进程具有较高的优先级。
	
前台, 可见, 服务, 后台, 空

1. 如果service正在调用onCreate, onStartCommand或者onDestory方法，那么用于当前service的进程相当于前台进程以避免被killed。
2. 如果当前service已经被启动(start)，拥有它的进程则比那些用户可见的进程优先级低一些，但是比那些不可见的进程更重要，这就意味着service一般不会被killed.
3. 如果客户端已经连接到service (bindService),那么拥有Service的进程则拥有最高的优先级，可以认为service是可见的。
4. Service长期在后台运行, 时间长了可能被系统杀死, 可以使用startForeground(int, Notification)方法来将service设置为前台状态，那么系统就认为是对用户可见的，并不会在内存不足时killed。
	
如果有其他的应用组件作为Service,Activity等运行在相同的进程中，那么将会增加该进程的重要性。

1. Service的特点可以让他在后台一直运行,可以在service里面创建线程去完成耗时的操作.
2. Broadcast receiver捕获到一个事件之后,可以起一个service来完成一个耗时的操作.
3. 远程的service如果被启动起来,可以被多次bind, 但不会重新create.  
##42.Intent Filter。
Intent filter: 可以理解为邮局或者是一个信笺的分拣系统…  

*	Action: 动作    view 
*	Data: 数据uri   uri
*	Category : 而外的附加信息

*	Action 匹配
	Action 是一个用户定义的字符串，用于描述一个Android应用程序组件，一个Intent Filter可以包含多个 Action。在 AndroidManifest.xml的Activity定义时可以在其<intent-filter>节点指定一个Action列表用于标示Activity所能接受的“动作”，例如：

 		<intent-filter > 
 		<action android:name="android.intent.action.MAIN" /> 
 		<action android:name="cn.itcast.action" /> 
		……
 		</intent-filter> 

	如果我们在启动一个 Activity 时使用这样的 Intent 对象:

 		Intent intent =new Intent(); 
 		intent.setAction("cn.itcast.action"); 

	那么所有的 Action 列表中包含了“cn.itcast”的 Activity 都将会匹配成功。  
	Android 预定义了一系列的 Action 分别表示特定的系统动作。这些 Action 通过常量的方式定义在 android.content.Intent中，以“ACTION_”开头。我们可以在 Android 提供的文档中找到它们的详细说明。
*	URI 数据匹配  
	一个Intent可以通过URI携带外部数据给目标组件。在 <intent-filter>节点中，通过 <data/>节点匹配外部数据。  
	mimeType属性指定携带外部数据的数据类型,scheme指定协议，host、port、path 指定数据的位置、端口、和路径。如下：

		<data android:mimeType="mimeType" android:scheme="scheme" 
		android:host="host" android:port="port" android:path="path"/> 
	电话的uri   　tel: 12345 　　http://www.baidu.com
	自己定义的uri  
	`itcast://cn.itcast/person/10`

	如果在 Intent Filter 中指定了这些属性，那么只有所有的属性都匹配成功时 URI 数据匹配才会成功。
*	Category 类别匹配  
	<intent-filter >节点中可以为组件定义一个Category类别列表，当Intent中包含这个列表的所有项目时Category类别匹配才会成功。
	默认是DEFAULT
##43.请描述一下Broadcast Receiver。
有很多广播接收者 ,系统已经实现了。 
广播分两种:	有序广播　无序广播
指定接收者的广播，是不可以被拦截掉的  

	<intent-filter android:priority="1000">
	<action android:name="android.provider.Telephony.SMS_RECEIVED"/>
	</intent-filter>
	abortBroadcast();

用于接收系统的广播通知, 系统会有很多sd卡挂载,手机重启,广播通知,低电量,来电,来短信等….有些应用通过这些广播启动后台服务, 避免被杀死

*	电量改变广播, 必须动态注册
*	来获取短信到来的广播, 根据黑名单来判断是否拦截该短信

画画板生成图片后,发送一个sd挂载的通知,通知系统的gallery去获取到新的图片.

	Intent intent = new Intent(Intent.ACTION_MEDIA_MOUNTED,Uri.parse("file://"+Environment.getExternalStorageDirectory()));
	sendBroadcast(intent);

可以作为组件间通信工具. EventBus
##44.请介绍下ContentProvider是如何实现数据共享的。
把自己的数据通过uri的形式共享出去  
android系统下不同程序数据默认是不能共享访问   
需要去实现一个类去继承ContentProvider  

	public class PersonContentProvider extends 	ContentProvider{
		Static{ }
			public boolean onCreate(){
			//..SqliteOpenHelper	
		}
		query(Uri, String[], String, String[], String)
		insert(Uri, ContentValues)
		update(Uri, ContentValues, String, String[])
		delete(Uri, String, String[])
	}
##45.为什么要用ContentProvider？它和sql的实现上有什么差别？
屏蔽数据存储的细节,对用户透明,用户只需要关心操作数据的uri就可以了  
不同app之间共享,操作数据  
Sql也有增删改查的方法,但是contentprovider还可以去增删改查本地文件.xml文件的读取,更改,网络数据读取更快。
##46.widget相对位置的完成在activity的哪个生命周期阶段实现。
1.	widget可以理解成桌面小控件,
	也可以理解成某个button, imageview这样的控件…
	onCreate, onStart, onResume,都得不到
	在onWindowFocusChanged可以得到
2.	如下：

		mTv.getViewTreeObserver	().addOnGlobalLayoutListener	(new 	OnGlobalLayoutListener() {
   	 		@Override
   			public void onGlobalLayout() {
        	System.out.println("onGlobalLayout: "+mTv.getHeight());
        	mTv.getViewTreeObserver().removeGlobalOnLayoutListener(this);
    		}
		}); 
##47.Activity是如何生成一个view的，机制是什么。
可以讲下activity的源码,比如说 每个activity里面都有window.callback和keyevent.callback,一些回调的接口或者函数吧. 框架把activity创建出来就会调用里面的这些回调方法,会调用activity生命周期相关的方法.  
Activity创建一个view是通过 ondraw 画出来的, 画这个view之前呢,还会调用onmeasure方法来计算显示的大小.
##48.Android程序与Java程序的区别？
Android程序用android sdk开发,java程序用javasdk开发.  
Android SDK引用了大部分的Java SDK，少数部分被Android SDK抛弃，比如说界面部分，java.awt  swing  package除了java.awt.font被引用外，其他都被抛弃，在Android平台开发中不能使用。  
android sdk 添加工具jar httpclient , pull  opengl 
##49.在Android中，怎么节省内存的使用，怎么主动回收内存？
回收已经使用的资源,    
合理的使用缓存  
合理设置变量的作用范围…  application 对象    
//未来的某一段时间执行  `System.gc();`
##50.Android系统中GC什么情况下会出现内存泄露呢？  视频编解码/内存泄露
检测内存泄露  工具  　mat
	
导致内存泄漏主要的原因是，先前申请了内存空间而忘记了释放。如果程序中存在对无用对象的引用，那么这些对象就会驻留内存，消耗内存，因为无法让垃圾回收器GC验证这些对象是否不再需要。如果存在对象的引用，这个对象就被定义为"有效的活动"，同时不会被释放。要确定对象所占内存将被回收，我们就要务必确认该对象不再会被使用。典型的做法就是把对象数据成员设为null或者从集合中移除该对象。但当局部变量不需要时，不需明显的设为null，因为一个方法执行完毕时，这些引用会自动被清理。

Java带垃圾回收的机制,为什么还会内存泄露呢?

	Vector v = new Vector(10);     
 		for (int i = 1; i < 100; i++)      {      
			Object o = new Object();      　
			v.add(o);      　
			o = null;      
		}//
此时，所有的Object对象都没有被释放，因为变量v引用这些对象。    
观察者模式解除注册  
Java 内存泄露的根本原因就是保存了不可能再被访问的变量类型的引用
##51.Android UI中的View如何刷新。

在主线程中  拿到view调用invalide()方法  
在子线程里面可以通过postInvalide()方法;  
requestLayout

##52.简单描述下Android 数字签名。
在Android系统中，所有安装到系统的应用程序都必有一个数字证书，此数字证书用于标识应用程序的作者和在应用程序之间建立信任关系.   
Android系统要求每一个安装进系统的应用程序都是经过数字证书签名的，数字证书的私钥则保存在程序开发者的手中。Android将数字证书用来标识应用程序的作者和在应用程序之间建立信任关系，不是用来决定最终用户可以安装哪些应用程序。  
这个数字证书并不需要权威的数字证书签名机构认证(CA)，它只是用来让应用程序包自我认证的。  
同一个开发者的多个程序尽可能使用同一个数字证书，这可以带来以下好处。  

1. 有利于程序升级，当新版程序和旧版程序的数字证书相同时，Android系统才会认为这两个程序是同一个程序的不同版本。如果新版程序和旧版程序的数字证书不相同，则Android系统认为他们是不同的程序，并产生冲突，会要求新程序更改包名。
2. 有利于程序的模块化设计和开发。Android系统允许拥有同一个数字签名的程序运行在一个进程中，Android程序会将他们视为同一个程序。所以开发者可以将自己的程序分模块开发，而用户只需要在需要的时候下载适当的模块。
	
在签名时，需要考虑数字证书的有效期：

1. 数字证书的有效期要包含程序的预计生命周期，一旦数字证书失效，持有改数字证书的程序将不能正常升级。
2. 如果多个程序使用同一个数字证书，则该数字证书的有效期要包含所有程序的预计生命周期。
3. Android Market强制要求所有应用程序数字证书的有效期要持续到2033年10月22日以后。 

Android数字证书包含以下几个要点：

1. 所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序
2. Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
3. 如果要正式发布一个Android ，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布。
4. 数字证书都是有有效期的，Android只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。
##53.什么是ANR 如何避免它？
在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：Application Not Responding）对话框。用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要，这样，系统不会显示ANR给用户。

	Activity 5秒  broadcast10秒, service20秒

	耗时的操作 worker thread里面完成, handler message…AsynTask , intentservice.等…
	Data/anr/traces.txt
##54.android中的动画有哪几类，它们的特点和区别是什么？
	
两种，一种是Tween动画、还有一种是Frame动画。  
Tween动画，这种实现方式可以使视图组件移动、放大、缩小以及产生透明度的变化；  
可以通过布局文件,可以通过代码

1. 	控制View的动画 

		a)	alpha(AlphaAnimation) 
		渐变透明	
		b)	scale(ScaleAnimation) 
		渐变尺寸伸缩	
		c)	translate(TranslateAnimation)
		画面转换、位置移动	
		d)	rotate(RotateAnimation)
		画面转移，旋转动画	
2. 	控制一个Layout里面子View的动画效果
	
		a)	layoutAnimation(LayoutAnimationController)
		b)	gridAnimation(GridLayoutAnimationController)
另一种Frame动画，传统的动画方法，通过顺序的播放排列好的图片来实现，类似电影。
	
		属性动画 ObjectAnimator , ValueAnimator, TypeEvaluator
		Nineoldandroids.jar, 和真正的属性动画还是不一样.
		scrollTo/scrollBy
##55.说说mvc模式的原理，它在android中的运用。
	
MVC英文即Model-View-Controller，即把一个应用的输入、处理、输出流程按照Model、View、Controller的方式进行分离，这样一个应用被分成三个层——模型层、视图层、控制层。

Android中界面部分也采用了当前比较流行的MVC框架，在Android中M就是应用程序中二进制的数据，V就是用户的界面。Android的界面直接采用XML文件保存的，界面开发变的很方便。在Android中C也是很简单的，一个Activity可以有多个界面，只需要将视图的ID传递到setContentView()，就指定了以哪个视图模型显示数据。

在Android SDK中的数据绑定，也都是采用了与MVC框架类似的方法来显示数据。在控制层上将数据按照视图模型的要求（也就是Android SDK中的Adapter）封装就可以直接在视图模型上显示了，从而实现了数据绑定。比如显示Cursor中所有数据的ListActivity，其视图层就是一个ListView，将数据封装为ListAdapter，并传递给ListView，数据就在ListView中显示。
	
MVP  
MVVM
##56.通过点击一个网页上的url 就可以完成程序的自动安装,描述下原理

AddJavascriptInterface, Java/JS互相调用
	
	new Object{
		callphone();
		installapk();
	}
http://1.vduntest.sinaapp.com/webview/WebView%E8%AF%A6%E8%A7%A3.html

##57.java中的soft reference是个什么东西
	
StrongReference 是 Java 的默认引用实现, 它会尽可能长时间的存活于 JVM 内， 当没有任何对象指向它时 GC 执行后将会被回收

SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收(虚拟机保证), 这一特性使得 SoftReference 非常适合缓存
	
##58.service里面可以弹土司么
	
可以, 对于对话框, 需要加权限
	
	AlertDialog dialog = new AlertDialog.Builder(this).setTitle("标题").setMessage("内容").setPositiveButton("确定", null).create();
	dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
	dialog.show();
	<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
	Stk
	显示悬浮窗
##59.ddms 和traceview的区别.
	
daivilk debug manager system 
	
1. 在应用的主activity的onCreate方法中加入`Debug.startMethodTracing("要生成的traceview文件的名字");`
2. 同样在主activity的onStop方法中加入
`Debug.stopMethodTracing();`
3. 同时要在AndroidManifest.xml文件中配置权限
   	`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"></uses-permission>`
4. 重新编译，安装，启动服务，测试完成取对应的traceview文件(adb pull /sdcard/xxxx.trace)。
5. 直接在命令行输入traceview xxxxtrace，弹出traceview窗口，分析对应的应用即可。

traceview 分析程序执行时间和效率

KPI : key performance information : 关键性能指标:  
splash界面不能超过5秒  
从splash 界面加载mainactivity 不能超过0.7秒 
 
##60.自定义属性

1. 在attrs.xml声明节点declare-styleable

		<declare-styleable name="ToggleView">
	        <attr name="switch_background" format="reference" />
	        <attr name="slide_button" format="reference" />
	        <attr name="switch_state" format="boolean" />
	    </declare-styleable>

2. R会自动创建变量

		attr 3个变量
		styleable 一个int数组, 3个变量(保存位置)

3. 在xml配置声明的属性/ 注意添加命名空间
	
	    xmlns:itheima="http://schemas.android.com/apk/res/com.itheima74.toggleview"
		
        itheima:switch_background="@drawable/switch_background"
        itheima:slide_button="@drawable/slide_button"
        itheima:switch_state="false"

4. 在构造函数中获取并使用

		// 获取配置的自定义属性
		String namespace = "http://schemas.android.com/apk/res/com.itheima74.toggleview";
		int switchBackgroundResource = attrs.getAttributeResourceValue(namespace , "switch_background", -1);
		

##61.Android的界面绘制

MeasureSpec类：一个32位的int值，其中高2位为测量的模式，低30位为测量的大小。

测量模式：  

*	EXACTLY：精确值模式
*	AT_MOST：最大值模式
*	UNSPECIFIED

View的默认onMeasure()方法只支持EXACTLY模式，如果要让自定义的View支持wrap_content属性，必须重写onMeasure()制定大小。

	private int measureHeight(int heightMeasureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(heightMeasureSpec);
        int specSize = MeasureSpec.getSize(heightMeasureSpec);

        if (specMode == MeasureSpec.EXACTLY){
            result = specSize;
        }else {
            result = 300;
            if (specMode == MeasureSpec.AT_MOST){
                result = Math.min(result,specSize);
            }
        }
        return result;
    }

	 
	  测量			 摆放		绘制
	  measure	->	layout	->	draw
	  	  | 		  |			 |
	  onMeasure -> onLayout -> onDraw 重写这些方法, 实现自定义控件
	  
	  都在onResume()之后执行
	  
	  View流程
	  onMeasure() (在这个方法里指定自己的宽高) -> onDraw() (绘制自己的内容)
	  
	  ViewGroup流程
	  onMeasure() (指定自己的宽高, 所有子View的宽高)-> onLayout() (摆放所有子View) -> onDraw() (绘制内容)

##62.UI界面架构图
![](http://i.imgur.com/lXkk2E9.png)
