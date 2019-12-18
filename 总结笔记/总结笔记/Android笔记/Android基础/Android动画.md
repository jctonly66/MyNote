
##View动画
View动画主要是对View进行动画操作，经过变化后，不具备交互性，其响应事件（点击事件）的位置仍然在原位置。该动画是在draw期间进行矩阵运算完成，并不会改变控件的实际参数。

View动画主要有4种：

- translate：平移动画
- scale：缩放动画
- rotate：旋转动画
- alpha：透明度动画


View动画可通过两种方式使用： 1. XML实现 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2.代码中实现。

###一、 XML文件实现View动画
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android"
    	android:duration="1000"
    	android:interpolator="@android:anim/accelerate_decelerate_interpolator">

    	<translate
        	android:fromXDelta="0"
        	android:fromYDelta="0"
        	android:toXDelta="200"
        	android:toYDelta="200"/>

		<scale
        	android:fromXScale="1.0"
        	android:fromYScale="1.0"
        	android:pivotX="100"
        	android:pivotY="50"
        	android:toXScale="0.1"
        	android:toYScale="0.1"/>

		<rotate
        	android:fromDegrees="0"
        	android:pivotX="100"
       	 	android:pivotY="50"
        	android:toDegrees="360"/>

		<alpha
        	android:fromAlpha="0.0"
        	android:toAlpha="1.0"/>
	</set>

>其中：`<set>`标签对应于`AnimationSet`类，表示动画集。

各动画支持的属性有：

**AnimationSet的属性**

- duration：时间长
- fillAfter：动画结束以后是否停留在结束位置，true表示View停留在结束位置，false则不停留
- interpolator：插值器，决定了动画的执行速度
- shareInterpolator：子标签动画是否共享同一个插值器

**TranslateAnimation的属性**

- fromXDelta：x 的起始值。
- fromYDelta：y 的起始值。
- toXDelta：x 的结束值。
- toYDelta：y 的结束值。

**ScaleAnimation的属性**

- fromXScale：水平方向缩放的起始值。
- fromYScale：垂直方向缩放的起始值。
- pivotX：缩放的轴心点 x。
- pivotY：缩放的轴心点 y。
- toXScale：水平方向缩放的结束值。
- toYScale：垂直方向缩放的结束值。

**RoatateAnimation的属性**

- fromDegrees：旋转开始的角度。
- pivotX：旋转的轴心点 x。
- pivotY：旋转的轴心点 y。
- toDegrees：旋转结束的角度。

**AlphaAnimation的属性**

- fromAlpha：透明度的起始值。
- toAlpha：透明度的结束值。

接下来，在代码中加载该动画的XML文件即可：

	// 平移，布局加载
	Animation animation = AnimationUtils.loadAnimation(this, R.anim.anim_translate);
	//开始动画
	view.startAnimation(animation)

###二、 代码实现View动画

Android中提供了`TranslateAnimation`、`ScaleAnimation`、`RotateAnimation`、`AlphaAnimation`、`AnimationSet`等动画类，分别对应平移动画、缩放动画、旋转动画、透明度动画、动画集合。


通过代码使用View动画如下：
	
	// 平移，代码设置
	TranslateAnimation translateAnimation = new TranslateAnimation(0, 0, 200, 200);
	translateAnimation.setDuration(1000);
	// 移动之后保留状态（默认为不保留）
	translateAnimation.setFillAfter(true);
	// view 启动动画
	view.startAnimation(translateAnimation)

###三、 动画监听
在View动画中，我们可以通过Animation的`setAnimationListener()`来添加AnimationListener监听事件。包括动画的开始、结束和重复事件。

	rotateAnimation.setAnimationListener(new Animation.AnimationListener() {
        @Override
        public void onAnimationStart(Animation animation) {
            
        }

        @Override
        public void onAnimationEnd(Animation animation) {

        }

        @Override
        public void onAnimationRepeat(Animation animation) {

        }
    });

###四、 自定义View动画
我们也可以自定义View动画，用来实现一些Android未提供的动画操作。

需要继承Animation，重写`initialize()`和`applyTransformation()`方法。

- `initialize()`：进行初始化操作。
- `applyTransformation()`：进行矩阵变换操作

>由于属性动画的出现以及View动画的局限性，自定义View并不常用。

###五、 LayoutAnimation
LayoutAnimation主要为ViewGroup 指定一个动画后，它的子元素出场时都会具有这种动画。这种效果一般可以作用于listview。


**1. 定义item入场的动画**

	<set xmlns:android="http://schemas.android.com/apk/res/android"
     	android:duration="300"
     	android:interpolator="@android:anim/accelerate_interpolator"
     	android:shareInterpolator="true">

    	<alpha
        	android:fromAlpha="0.0"
        	android:toAlpha="1.0"/>

    	<translate
        	android:fromXDelta="500"
        	android:toXDelta="0" />
	</set>

**2.定义LayoutAnimation**

	<?xml version="1.0" encoding="utf-8"?>
	<layoutAnimation
    	xmlns:android="http://schemas.android.com/apk/res/android"
    	android:animation="@anim/anim_item"
    	android:animationOrder="normal"
    	android:delay="0.5">

	</layoutAnimation>


- `animation`：入场动画

- `animationOrder`：动画的顺序（normal：依次显示，reverse：逆序显示，random：随机显示）

- `delay`：开始动画的延迟时间

**3. 为listview指定LayoutAnimation**

	<ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layoutAnimation="@anim/layout_anim"/>

或者在代码中为listview设定值
	
	Animation animation = AnimationUtils.loadAnimation(this, R.anim.anim_item);
    LayoutAnimationController controller = new LayoutAnimationController(animation);
    listView.setLayoutAnimation(controller);


##帧动画
顺序播放一组预定义的动画，容易引起 OOM，尽量避免大图片。<br>
在 XML 文件中使用 `<animation-list>` 定义一组图片。存放在 `res/drawable/`中

	<?xml version="1.0" encoding="utf-8"?>
	<animation-list xmlns:android="http://schemas.android.com/apk/res/android">

    	<item
        	android:drawable="@android:drawable/ic_menu_share"
        	android:duration="500"/>
    	<item
        	android:drawable="@android:drawable/ic_menu_today"
        	android:duration="500"/>
	    <item
	        android:drawable="@android:drawable/ic_menu_add"
	        android:duration="500"/>
	    <item
	        android:drawable="@android:drawable/ic_menu_agenda"
	        android:duration="500"/>
	    <item
	        android:drawable="@android:drawable/ic_menu_always_landscape_portrait"
	        android:duration="500"/>
	</animation-list>

在代码中调用使用，得到一个 AnimationDrawable 对象，然后 start

	view.setBackgroundResource(R.drawable.frame_animation);
    AnimationDrawable drawable = (AnimationDrawable) view.getBackground();
    drawable.start();

##View动画源码分析
每次绘制视图时View所在的ViewGroup中的drawChild函数获取该View的Animation的Transformation值，然后调用canvas.concat(transformToApply.getMatrix())，通过矩阵运算完成动画帧。不断调用，直到完成。

##属性动画
属性动画 Property Animation 是 API 11（3.0）新加入的特性，属性动画可以对**任何对象**做动画（一般还是对 View）。包括ValueAnimator、ObjectAnimator、AnimatorSet。

>与 View 动画不同，属性动画改变其属性，view 的属性真正改变，而不是 View 动画的假象那样。在3.0 以下，可以采用 nineoldandroids 库进行兼容低版本。在低版本本质任然是 View 动画，看起来为属性动画。

	// view 向下平移 500
	ObjectAnimator.ofFloat(view, "translationY", 500).start();

**特别注意：**ofFloat、ofInt... 依据属性的类型，属性是 int 型就用 ofInt，属性是 float 型就用 ofFloat 必须这么用，否则动画没效果。

###属性动画使用条件
该对象必须具有某属性的 get 、set 方法。因为内部实现就是通过反射获取属性的 get 、set 方法调用进行设置的。

###View一般属性：
- 坐标：x，y
- 透明度：alpha
- 背景颜色：backgroundColor
- 旋转：rotationX, rotationY
- 平移：translationX、translationY
- 缩放：scaleX、scaleY

###View的特殊属性

- 宽：width
- 高：height

View不存在setWidth()、setHeight()方法，所以不能直接对width进行动画，所以需要添加 setXXX 方法。

>如果动画没有传递初始值，那么还要提供get()方法，因为系统要去取属性的初始值。

方法：

1. 给对象加上get和set方法。
2. 用一个类包装原始对象，间接提供get和set方法。（装饰者模式）
3. 采用ValueAnimator的`addUpdateListener()`监听器方法动态更新。

**方法1（推荐）**

对 view 进行装饰，提供 setWidth、setHeight 方法。提供 ViewWrapper 类（装饰者模式）

	import android.view.View;
	public class ViewWrapper {
	    private View view;
	    private ViewWrapper(View view) {
	        this.view = view;
	    }

	    public static ViewWrapper decorator(View view) {
	        return new ViewWrapper(view);
	    }
	
	    public int getHeight() {
	        return view.getLayoutParams().height;
	    }
	
	    public void setHeight(int height) {
	        view.getLayoutParams().height = height;
	        view.requestLayout();
	    }
	
	    public int getWidth() {
	        return view.getLayoutParams().width;
	    }
	
	    public void setWidth(int width) {
	        view.getLayoutParams().width = width;
	        view.requestLayout();
	    }
	}	

具体使用方法：

	ObjectAnimator.ofInt(ViewWrapper.decorator(view), "width", 800).start();
**方法2**

采用 ValueAnimator 的 addUpdateListener 监听器方法动态更新。

	ValueAnimator valueAnimator = ValueAnimator.ofInt(view.getWidth(), 800);
    valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {

        @Override
        public void onAnimationUpdate(ValueAnimator animator) {
            int value = (Integer) animator.getAnimatedValue();
            // 拿到值以后自己处理
            view.getLayoutParams().width = value;
            view.requestLayout();
        }
    });

###插值器和估值器
####插值器
根据时间流逝的百分比来计算出当前属性值改变的百分比。系统预置的有LinearInterpolator、AccelerateDecelerateInterpolator、DecelerateInterpolator等。
####估值器
根据当前属性改变的百分比来计算改变后的属性值。系统预置的有IntEvaluator、FloatEvaluator、ArgbEvaluator等。
###属性动画的监听器
- AnimatorUpdateListener —— 数值变化的监听
- AnimatorListener —— 生命周期的监听
- AnimatorPauseListener —— 暂停与恢复的监听
- AnimatorListenerAdapter —— 生命周期的监听




##Android动画
###1. 逐帧动画
###2. 补间动画
补间动画可以对View进行一系列的动画操作，包括淡入淡出、缩放、平移、旋转4种。
###3. 属性动画
####为什么要有属性动画
1. 补间动画只是对view进行操作，如果对一个非View对象的画，就无能为力。
2. 补间动画扩展性不好，不支持背景色等功能。
3. 补间动画只是改变view的显示效果，而不是真正的改变view的属性。
4. 属性动画可以是任意的对象，是一种不断对值进行操作的机制，并将值赋值到指定对象的指定属性上。

####ValueAnimator
ValueAnimator是整个属性动画机制当中**最核心**的一个类，初始值和结束值之间的动画过渡就是由ValueAnimator这个类来负责计算的。它内部使用一种时间循环的机制来计算值与值之间的动画过渡，它还负责管理动画的播放次数、播放模式、以及对动画设置监听器等。

- ValueAnimator.ofInt()
- ValueAnimator.ofFloat()
- ValueAnimator.ofObject()
- ValueAnimator.ofArgb()
- setDuration()
- setStartDelay()
- setRepeatCount()

####ObjectAnimator
ValueAnimator只是对值进行了一个平滑的动画过渡，ObjectAnimator则是可以直接对任意对象的任意属性进行动画操作的，它继承自ValueAnimator。

####Animator监听器
Animator类中提供了一个addListener()这个方法。这个方法接收了一个AnimatorListener,我们只需要去实现这个AnimatorListener就可以监听各种动画了。
>ValueAnimator继承自Animator

	new Animator.AnimatorListener() {
    	@Override
        public void onAnimationStart(Animator animator) {
        	//动画开始的时候调用
        }

        @Override
        public void onAnimationEnd(Animator animator) {
        	//动画结束的时候调用
        }

        @Override
        public void onAnimationCancel(Animator animator) {
            //动画取消的时候调用
        }

        @Override
        public void onAnimationRepeat(Animator animator) {
            //动画重复的时候调用
        }
    }

但是，很多时候我们只需要监听其中的某一个事件。使用`AnimatorListenerAdapter()`就可以解决实现接口繁琐的问题。
####XML编写动画
- <animator>  对应代码中的ValueAnimator
- <objectAnimator>  对应代码中的ObjectAnimator
- <set>  对应代码中的AnimatorSet

示例：

	<set xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:ordering="sequentially" >  
	  
	    <objectAnimator  
	        android:duration="2000"  
	        android:propertyName="translationX"  
	        android:valueFrom="-500"  
	        android:valueTo="0"  
	        android:valueType="floatType" >  
	    </objectAnimator>  
	  
	    <set android:ordering="together" >  
	        <objectAnimator  
	            android:duration="3000"  
	            android:propertyName="rotation"  
	            android:valueFrom="0"  
	            android:valueTo="360"  
	            android:valueType="floatType" >  
	        </objectAnimator>  
	  
	        <set android:ordering="sequentially" >  
	            <objectAnimator  
	                android:duration="1500"  
	                android:propertyName="alpha"  
	                android:valueFrom="1"  
	                android:valueTo="0"  
	                android:valueType="floatType" >  
	            </objectAnimator>  
	            <objectAnimator  
	                android:duration="1500"  
	                android:propertyName="alpha"  
	                android:valueFrom="0"  
	                android:valueTo="1"  
	                android:valueType="floatType" >  
	            </objectAnimator>  
	        </set>  
	    </set>  
	</set>  

	Animator animator = AnimatorInflater.loadAnimator(context, R.animator.anim_file);  
	animator.setTarget(view);  
	animator.start();

####TypeEvaluator
TypeEvaluator就是告诉动画如何从初始值过渡到结束值。FloatEvaluator的代码实现：

	public class FloatEvaluator implements TypeEvaluator {  
	    public Object evaluate(float fraction, Object startValue, Object endValue) {  
	        float startFloat = ((Number) startValue).floatValue();  
	        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);  
	    }  
	}  

其中，evaluate()有三个参数：

- fraction：表示动画的完成度。
- startValue：动画的初始值。
- endValue：动画的结束值。

系统为我们内置了FloatEvaluator和IntEvaluator的类，对应于ofFloat()以及ofInt()，分别用于对浮点型和整形的数据进行动画操作。但是我们若要使用ofObject()方法对任意对象进行动画操作，由于没有内置的，所以我们需要实现一个自己的TypeEvaluator来告知系统如何进行过渡。

####Interpolator
在属性动画中新增了一个TimeInterpolator接口，这个接口是用于**兼容**之前的Interpolator的，这使得所有过去的Interpolator实现类都可以直接拿过来放到属性动画当中使用。
>- AccelerateInterpolator：加速运动。
>- DecelerateInterpolator：减速运动。
>- 等等。。。

**自定义Interpolator**

实现TimeInterpolator类，并重写getInterpolation()方法。
>getInterpolation()方法中接收一个input参数，这个参数的值会随着动画的运行而不断变化，不过它的变化是非常有规律的，就是根据设定的动画时长匀速增加，变化范围是0到1。也就是说当动画一开始的时候input的值是0，到动画结束的时候input的值是1，而中间的值则是随着动画运行的时长在0到1之间变化的。

input与fraction的区别：

input的值决定了fraction的值。input的值是由系统经过计算后传入到getInterpolation()方法中的，然后我们可以自己实现getInterpolation()方法中的算法，根据input的值来计算出一个返回值，而这个返回值就是fraction了。

####ViewPropertyAnimator
主要是对view的一种更加便捷的用法。请看示例：

	textview.animate().alpha(0f);	//将当前textview变成透明
	textview.animate().x(500).y(500);  //运动到（500，500）

- 整个`ViewPropertyAnimator`的功能都是建立在View类新增的`animate()`方法之上的，这个方法会创建并返回一个`ViewPropertyAnimator`的实例，之后的调用的所有方法，设置的所有属性都是通过这个实例完成的。
- 大家注意到，在使用`ViewPropertyAnimator`时，我们自始至终没有调用过`start()`方法，这是因为新的接口中使用了隐式启动动画的功能，只要我们将动画定义完成之后，动画就会自动启动。并且这个机制对于组合动画也同样有效，只要我们不断地连缀新的方法，那么动画就不会立刻执行，等到所有在`ViewPropertyAnimator`上设置的方法都执行完毕后，动画就会自动启动。当然如果不想使用这一默认机制的话，我们也可以显式地调用`start()`方法来启动动画。
- `ViewPropertyAnimator`的所有接口都是使用连缀的语法来设计的，每个方法的返回值都是它自身的实例，因此调用完一个方法之后可以直接连缀调用它的另一个方法，这样把所有的功能都串接起来，我们甚至可以仅通过一行代码就完成任意复杂度的动画功能。
