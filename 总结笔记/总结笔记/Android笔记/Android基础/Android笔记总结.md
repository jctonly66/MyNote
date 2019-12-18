##Android动画总结
###View动画
View动画主要是对View进行动画操作，经过变化后，不具备交互性，其响应事件（点击事件）的位置仍然在原位置。该动画是在draw期间进行矩阵运算完成，并不会改变控件的实际参数。

- translate：平移动画
- scale：缩放动画
- rotate：旋转动画
- alpha：透明度动画

**监听事件：**

在View动画中，我们可以通过AnimationListener监听事件。

**自定义View动画**

我们也可以自定义View动画，用来实现一些Android未提供的动画操作。

需要继承Animation，重写`initialize()`和`applyTransformation()`方法。

- `initialize()`：进行初始化操作。
- `applyTransformation()`：进行矩阵变换操作

**LayoutAnimation**

LayoutAnimation主要为ViewGroup 指定一个动画后，它的子元素出场时都会具有这种动画。
>动画->layoutAnimator->viewGroup

###帧动画

顺序播放一组预定义的动画，容易引起 OOM，尽量避免大图片。

###属性动画
为什么设计出属性动画？

由于View动画仅针对view通过矩阵运算进行绘制，不改变具体的属性值。

####ValueAnimator
ValueAnimator是不断对值进行操作，是用来反映值的变化的动画。

- ofInt()
- ofFloat()
- ofObject()：可设置一个自定义的TypeEvaluator参数。
- ofArgb()

####ObjectAnimator
ObjectAnimator继承自ValueAnimator。相比ValueAnimator来说可设置一个指定的目标和目标的属性，对该属性的值不断进行set操作，完成动画。
>该属性的set、get方法需要存在。

####AnimatorSet
组合动画

- after();
- with();
- before();

####监听事件
- UpdateListener（数值变化监听）
- AnimatorListener（生命周期监听）
- AnimatorListenerAdapter（可只实现一个生命周期）
- AnimatorPauseListener（暂停与恢复监听）

####TypeEvaluator
告诉动画如何从初始值过渡到结束值。

- fraction：动画完成度，由插值器传入。
- 初始值
- 结束值

####Interpolator
控制动画的变化速率，随动画的运行时间变化，从0->1变化。input为(currentTime - mStartTime) / mDuration。

>系统计算（input）->interpolator（生成返回值）->fraction

###ViewPropertyAnimator
由于大部分操作还是针对view，所以Android提供了简化操作。

textview.animate().x(500).y(500).setDuration(5000);
>设置完成后，动画会自启动，不需要调用start()。

##事件分发机制
	
一个事件产生后，会先传递给当前的Activity，然后通过Window传递给DecorView。开始ViewGroup及View的事件分发。

1. 同一事件序列是指从手机接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以down事件开始，中间含有数量不定的move事件，最终以up结束。

2. 正常情况下，一个事件序列只能被一个View拦截且消耗。这一条的原因可以参考3，因为一旦一个元素拦截了某次事件，那么同一个事件序列内的所有事件都会**直接交给**它处理，因此同一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View处理。
	>mFirstTouchTarget != null

3. 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理（如果事件序列能够传递给他的话）,并且它的onInterceptTouchEnent不会再被调用。这条也很好理解，就是说当一个View决定拦截一个事件后，那么系统会把同一事件序列内的其他事件都交给它来处理，因此就不用再调用这个View的onInterceptTouchEvent去询问它是否要拦截了。

4. 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回false），那么同一事件序列中的其他事件都不会再交给他来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。

5. 如果View不消耗除ACTION_DOWN以外的其它事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。

6. ViewGroup默认不拦截任何事件。Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false。

7. View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。

8. View的onTouchEvent默认都会消耗掉事件，除非他是不可点击的。View的longClickable属性默认都为false，click属性要分情况，各个控件不一样。

9. View的enable属性不影响onTouchEvent的默认返回值。

10. onClick发生的前提是当前view是可点击的，并且它收到了up的事件。

11. 事件传递是由外向内的，即时间总是先传递给父元素，然后再有父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但ACTION_DOWN事件除外。 

####源码
Activity->Window->DecorView(dispatchTouchEvent)

####View的事件分发

`dispatchTouchEvent()`：

1. 是否注册`onTouchListener()`、并且是否可用enable
	1. 若已注册，调用`onTouch()`
	2. 若未注册或者`onTouch()`返回flase，调用`onTouchEvent()`
		1.  在`onTouchEvent()`中，处理长按、点击等事件。根据Move、Up、Down等
			>若有点击，则会默认消耗掉事件。
			>若有注册点击事件，则调用`onClickListener`。

####ViewGroup的事件分发

`dispatchTouchEvent()`分两种情况：

1. DOWN事件：
	1. 调用`onInterceptTouchEvent()`
		1. 若拦截，则调用自身的`onTouchEvent()`
		2. 若不拦截，则遍历子view，如果子View在点击事件范围内，则调用它的`dispatchTouchEvent()`。
			1. 如果消耗点事件，则给mFirstTouchTarget附值。
			1. 如果没有子View或者子View不消耗事件，则调用`super.dispatchTouchEvent()`调用viewGroup的`onTouchEvent()`。

2. UP、MOVE事件：
	1. 判断mFirstTouchTarget是否为空
		1. 若为空，不再调用`onInterceptTouchEvent()`，直接使用自己的`onTouchEvent()`。
		2. 若不为空，判断`disallowInterceptTouchEvent`是否为true
			1. 若为true，则直接返回false，调用mFirstTouchTarget的`dispatchTouchEvent()`。
			2. 若为false，调用`onInterceptTouchEvent()`判断是否拦截。拦截的话调用自身`onTouchEvent()`，不拦截的话调用子元素`onTouchEvent()`。


##高效的图片加载
加载图片的方法为BitmapFactory的4类方法：

- decodeFile
- decodeStream
- decodeResource
- decodeByteArray

>decodeFile以及decodeResource会间接调用decodeStram。

####如何高效地加载Bitmap呢？
采用BitmapFactory.Options来加载所需尺寸的图片。它可以按一定的采样率来加载缩小后的图片。

- inSampleSize参数。当值为1时，不缩放；当值为2时，那么采样后的图片其宽/高均为原图大小的1/2，像素数为原图的1/4，其占有内存的大小也为原图的1/4；依次类推....
>若小于1，其作用相当于1，即无缩放效果；若不为2的整数，则会向下取整（但是并非所有的Android版本上都成立）。

具体步骤：

1. 将BitmapFactory.Options的inJustDecodeBounds参数设置为true并加载图片。
2. 从BitmapFactory.Options中取出图片的原始宽高信息，它们对应于outWidth和outHeight参数。
3. 根据采样率的规则并结合目标View的所需大小计算出采样率inSampleSize。
4. 将BitmapFactory.Options的inJustDecodeBounds参数设置为false，然后重新加载图片。
>inJustDecodeBounds参数为true时，BitmapFactory只会解析图片的原始宽/高信息，并不会去真正的加载图片，所以这个操作是轻量级的。 

###Android中的缓存策略
内存>存储设备>网络

LRU：最近最少使用算法。当缓存满时，优先淘汰那些近期最少使用的缓存对象。

####LruCache
LruCache是一个泛型类，它内部采用了一个LinkedHashMap以强引用的方式存储外界的缓存对象。其提供了get和put方法来完成缓存的获取和添加操作，当缓存满时，LruCache会移除较早使用的缓存对象，然后再添加新的缓存对象。

>- sizeOf：完成对Bitmap对象的计算。
>- entryRemoved：在LruCache移除旧缓存时会调用，可完成一些资源的回收操作。

使用时get、put。
####DiskLruCache
DiskLruCache用于实现存储设备缓存。

1. DiskLruCache的创建`public static DiskLruCache open(File directory,int appVersion, int valueCount, long maxsize)`
	- directory：磁盘缓存在文件系统中的存储路径。
	- appVersion：版本号，一般存入1。当发生改变时会清空之前所有的缓存文件（可能会失效）。
	- valueCount：单个节点所对应的数据的个数，一般设为1即可。
	- maxsize：缓存的总大小，当超过后，会清除部分缓存。
2. DiskLruCache的缓存添加
	
	通过Editor来完成。Editor表示一个缓存对象的编辑对象。可根据key值通过`edit()`获取Editor对象，如果这个缓存正在被编辑，那么会返回null。
	>之所以把图片的url转换成key，是因为图片的url中很可能有特殊字符，这将影响url在Android中的使用，一般采用url的MD5值作为key。

3. DiskLruCache的缓存查找
	
	根据key值通过DiskLruCache的get方法得到一个Snapshot对象，接着再通过Snapshot对象即可得到缓存的文件输入流，借此自然就可得到Bitmap对象。


##Handler机制

系统启动->Loope.prepareMainLooper()->Loop.prepare()
>所以程序中会始终存在一个looper对象。

Looper中会包含一个MessageQueue。

创建Handler的时候会获取Looper对象以及MessageQueue。

发送消息的时候：sendMessageAtFrontOfQueue()之外都会调用sendMessageAtTime()，在入队的时候，所有的消息都按时间来排序，而sendMessageAtFrontOfQueue()会使when=0，直接插到队首。

取消息是Looper，一个死循环。获得消息后会调用dispatchMessage()，首先判断message有没有自己的Callback，有的话交给message的Callback处理，否则交给handler的handleMessage处理。

发送消息的三种方法：

- handler的post(Runnable r)：会把r赋给message的callback。

- view的post()：使用handler.post()

- activity的runOnUIThread()：如果在主线程，则直接调用runnable的run方法，否则调用使用handler.post()