###Java基本数据类型

8种原始类型 | 容量 | 对应包装类型
------------ | ------------- | ------------
byte(字节)    |         8 位   |           Byte
shot(短整型)   |        16位     |         Short
int(整型)      |           32 位 |            Integer
long(长整型)   |       64 位      |         Long
float(浮点型)  |        32 位     |          Float
double(双精度) |    64 位        |       Double
char(字符型)   |       16 位     |          Character
boolean(布尔型) |   1 位         |       Boolean

需要注意点：

1. 基本数据类型可以直接使用“==”进行判等。
2. 封装类型需要使用`equals()`进行判等，如果是同“==”只是判断实例的引用值
3. 装箱与拆箱：
	
	装箱是使用Integer.valueOf()。如果Integer或Long装箱的时候会使用到缓存-128~127。不在缓存范围内才会new出一个对象。
	>注意，只是装箱的时候才会使用到缓存，直接new的时候不会使用缓存。

		Integer a = 3;
	    Integer b = 3;
	    Integer c = Integer.valueOf(3);
	    System.out.print(a == b);   //true
	    System.out.print(a == c);   //true
	    
	    Integer m = new Integer(3);
	    Integer n = new Integer(3);
	    System.out.print(n == m);   //false
4. 封装类型与基本数据类型使用“==”会自动拆箱
5. 各数据类型按容量大小由小到大排列为：

	byte ——> short, char ——> int ——> long ——> float ——> double

6. 基本类型之间的转换原则：

	- 运算时，容量小的类型自动转换为容量大的类型；
	- 容量大的类型转换为容量小的类型时，要加**强制转换符**，且精度可能丢失；
	- 实数常量默认为double类型， 整数常量默认为int类型；
	- short，char之间不会互相转换（需要强制转换），byte、short、char在计算时首先转换为int类型；
	
7. 包装类及String类都是定定义为public final class的，因此这几个都不能被继承；
8. += 与“先+再=”的区别：
	
	+=赋值是直接在原来值基础上直接加，不涉及精度问题，而先+后=可能涉及到精度问题。
	
		short s1 = 1; s1 = s1 + 1;   //出错，需要强制转换
		short s1 = 1; s1 += 1;	//无错

9. 编译器相关错误：
	
	float f = 1.5; 会出错，需要强制转换    
	但是byte、short、char时  byte m = 1;则不会出错,只有超过自己的范围才会强制转换。

10. &和&&的区别

	&是位运算符，表示按位与运算，&&是逻辑运算符，表示逻辑与（and）。
###变量存放位置
- 成员变量：存放在堆内存中
- 局部变量：存放在栈中
- 静态变量：由static关键字修饰，存放在静态存储区

###String、StringBuffer、StringBuilder的区别

**String**

1. String是一个字符串常量，创建后不可更改，所以我们每次去更改String的时候，实际上创建了新的String对象。
	>其内部定义的是一个final类型的数组来保存值的。

2. String常量池：

	每次创建String对象时都会在字符串常量池中查找有没有该对象：如果有则直接使用它，如果没有就在字符串常量池中创建出来然后使用它。

	此时，两种String的创建方式会有所不同：
		
	- 当使用String s = "wss"的时候，会把s直接指向常量池。（存储常量池的地址）
	- 当使用new创建的时候，会在堆中创建一个对象，并从常量池中拷贝一份到值到堆中。（存储堆的地址）

	使用代码验证：

		String s1 = "abc";
	    String s2 = "abc";
	    String s3 = new String("abc");
	
	    System.out.print(s1 == s2);     //true
	    System.out.print(s1 == s3);     //false

**StringBuffer和StringBuilder的区别：**

- StringBuffer和StringBuilder在功能上是相同的，都可以直接对它们的值进行修改（区别于String）。

- StringBuffer和StringBuilder的最主要区别就是StringBuffer内大部分方法都添加了synchronized同步，所以StringBuffer是线程安全的，而StringBuilder不是线程安全的。
- 因此，如果我们需要线程安全的，就用StringBuffer，否则，使用StringBuilder会更加高效。

###关键字
1. switch只能支持byte、short、char、int或者其对应的封装类以及Enum类型。在jdk1.7及1.7以后，switch也支持了String类型。
2. break与continue区别：

	break的作用是跳出当前循环；continue用于结束循环体中的本次循环。

###值传递与引用传递

当传递的是数组名或对象实例的话，其实传递的都是地址。要分清楚传的是值还是地址。

指出下列程序运行的结果 （B）

	```
	public class Example {
	
	    String str = new String("good");
	
	    char[] ch = { 'a', 'b', 'c' };
	
	    public static void main(String args[]) {
	
	        Example ex = new Example();
	
	        ex.change(ex.str, ex.ch);
	
	        System.out.print(ex.str + " and ");
	
	        System.out.print(ex.ch);
	
	    }
	
	    public void change(String str, char ch[]) {
	
	        str = "test ok";
	
	        ch[0] = 'g';
	
	    }
	}
	```
	A、 good and abc
	
	B、 good and gbc
	
	C、 test ok and abc
	
	D、 test ok and gbc 

###构造函数

1. 如果一个类中没有写任何的构造方法，JVM会生成一个默认的无参构造方法。
2. 如果一个基类中写了有参构造函数，没有定义无参构造函数，基类是不会默认生成无参构造函数的。
3. 如果子类的构造函数中，如果没有显示通过super.调用基类构造函数，那么默认是调用父类的无参构造方法，即默认为super()。如果基类中没有无参的构造方法，则编译出错。

###创建对象的方式

1. 用new语句创建对象，这是最常见的创建对象的方法。
2. 运用反射手段。
3. 调用对象的clone()方法。
4. 运用反序列化手段。

>1和2都会明确的显式的调用构造函数；3是在内存上对已有对象的影印，所以不会调用构造函数；4是从文件中还原类的对象，也不会调用构造函数。

###override（重写）与Overloading（重载）

1. Override（重写）:  
  
	在子类中定义与父类完全相同（参数、返回值、名称）方法，通过子类创建的实例对象调用这个方法时，将调用子类中的定义方法。特点如下：
	
	- 子类方法的访问权限只能比父类的更大，不能更小（可以相同）；
	- 如果父类的方法是private类型，那么，子类则不存在覆盖的限制，相当于子类中增加了一个全新的方法；

2. Overload（重载）:
	同一个类中可以有多个名称相同的方法，但方法的**参数个数**和**参数类型**或者**参数顺序**不同；
	>3个条件满足一个即可。

###静态代码块执行顺序
1. 代码块执行顺序

	静态代码块 --> 普通代码块 --> 构造方法

2. 在父子类关系中，执行顺序为

	父类静态代码块-->子类静态代码块-->父类普通代码块-->父类构造方法-->子类代码块-->子类构造方法；

###修饰符

**1、访问控制符**
	
		|修饰符	|当前类	|同 包	|子 类	|其他包|
		| ----- |------ |-------|-------|------|
		|public	|  √    | √	    | √	    | √    |
		|protected | √  |	 √	|     √	|     ×|
		|default|	√	| √	    | ×	    | ×    |
		|private|	√	| ×	    | ×	    | ×    |

- public： 包内及包外的任何类（包括子类和普通类）均可以访问。
- protected： 包内的任何类及包外那些继承了该类的子类才能访问。
- default： 包内的任何类（包括继承了此类的子类）都可以访问它，而对于包外的任何类都不能访问它（包括包外继承了此类的子类）。
- private： 只有本类可以访问，而包内包外的任何类均不能访问它。　

**2、其他修饰符：**

1. final修饰的类不能被继承，没有子类。
2. abstract修饰的类不能被实例化，必须被子类继承。类只要有一个抽象方法就必定是抽象类，但抽象类不一定要有抽象方法。



