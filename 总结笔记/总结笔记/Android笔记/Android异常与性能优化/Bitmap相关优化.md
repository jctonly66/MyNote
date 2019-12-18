##Bitmap
####一. recycle

- 在安卓3.0以后Bitmap是存放在堆中的，我们只要回收堆内存即可。
- 在安卓3.0以前Bitmap是存放在内存中的，我们需要回收native层和Java层的内存。

官方建议我们**3.0以前**使用`recycle`方法进行回收，`recycle`方法会释放有关Bitmap的native内存以及相关的数据引用。但是并不是立即回收，只是会发送指令到垃圾回收器，并标记Bitmap为dead状态，此后，如果调用Bitmap相关方法会引起异常。

`recycle`方法也可以不主动调用，因为垃圾回收器会自动收集不可用的Bitmap对象进行回收。

####二. LRU（最近最少使用算法）

LruCache是个泛型类，内部采用LinkedHashMap来实现缓存机制，它提供get方法和put方法来获取缓存和添加缓存。

当缓存满时，LRU提供`trimToSize`方法来移除最久最少使用的缓存，并添加新的缓存对象。

####三. 图片压缩

设置BitmapFactory.Options的inSampleSize;

>设置`options.inJustDecodeBounds = true`会让Bitmap只加载图片的宽高等信息。

####三级缓存

- 网络缓存
- 本地缓存（SD卡）
- 内存缓存（内存）