#Java中集合类详解

##Collection与Map详解

在Java中，集合类是一种非常有用的工具类。可以用来存储对象，或者用来实现常用的数据结构，如栈，队列等。Java集合大致可分为一下4种：

- Set：无序、不可重复的集合。
- List：有序、重复的集合。
- Queue：队列集合。
- Map：具有映射关系的集合。

这四种集合主要通过Collection以及Map两个根接口派生出。我们来看一下Collection的继承树以及Map的继承树：

![](http://i.imgur.com/LhtlSTm.png)

上图显示了Collection以及Map体系里的集合。其中，Set、List、Queue为Collection派生出的子集合，分别代表了无序集合、有序集合以及队列实现。Map实现类用于保存具有映射关系的数据。现在有个简单印象即可，以后会具体分析。（这里只列出了部分常用的子接口与类，若需要详细的请查看JavaAPI文档）

>集合主要负责保存以及盛装其他数据，所以集合类也被称为容器类。

这里抛出两个问题：

1. 既然集合类是用来存储对象的，那我们经常使用的基本数据类型如何存入呢？

	相信你肯定可以回答出来。

2. 一个容器实例中可以存入类型不同的对象吗？

	答案是可以也不可以。（后续会有答案）

###一、Collection接口

Collection接口是List、Set、Queue接口的父接口，所以该接口里定义的方法既可用于操作Set集合，也可用于操作List和Queue集合。我们先学习它们共有的方法：

- `boolean add(Object o)`：向集合里添加一个元素。如果集合对象被添加操作改变了，则返回true。
- `boolean addAll(Collection c)`：把集合c里的所有元素添加到指定集合里，如果集合对象被添加操作改变了，则返回true。
- `boolean remove(Object o)`：删除集合中的指定元素o，当集合中包含了一个或多个元素o时,该方法只删除第一个符合条件的元素，并且返回true。否则返回false。
- `boolean removeAll(Collection c)`：从集合中删除集合c里包含的所有元素（相当于用调用该方法的集合减去集合c），如果删除了一个或一个以上的元素，则返回true。否则返回false。
- `boolean retainAll(Collection c)`：从集合中删除集合c里**不包含**的所有元素（相当于获取调用该方法的集合与集合c的交集），如果该操作改变了调用该方法的集合，则返回true。否则返回false。
- `boolean contains(Object o)`：如果集合中包含指定的元素，则返回 true。
- `boolean containsAll(Collection c)`：如果集合中包含集合c中的所有元素，则返回 true
- `int size()`：返回集合中的元素个数。
- `void clear()`：移除集合中的所有元素。
- `boolean isEmpty()`：如果集合中不包含元素，则返回 true，否则返回false。
- `Iterator iterator()`：返回集合的迭代器（用来遍历集合）。
- `Object[] toArray()`：返回一个包含集合中所有元素的数组。

>不要被这些方法吓着了。其实，既然是一个容器，无非是一些添加对象、删除对象、查找对象、清空容器、获取大小、判空、遍历等操作。

既然Collection是Set、List、Queue的父接口，所以这些方法在这些接口中都可使用。只是会派生出一些支持自身特性的新方法而已。

下面，用一个例子来学习上面的方法：

	public static void main(String[] args){
        Collection c = new ArrayList();
        c.add("元素1");
        c.add("元素2");
        c.add("元素3");
        System.out.println("集合c中的元素："+c);
        c.remove("元素1");
        System.out.println("集合c中的元素："+c);
        c.add("元素4");
        c.add("元素5");
        Collection deleteC = new ArrayList();
        deleteC.add("元素2");
        deleteC.add("元素3");
        c.removeAll(deleteC);
        System.out.println("集合c中的元素："+c);
        System.out.println("集合c中是否包含元素5："+c.contains("元素5"));
        deleteC.retainAll(deleteC);
        System.out.println("集合c中的元素："+c);
        System.out.println("集合c中的元素个数："+c.size());
    }
执行结果：

	集合c中的元素：[元素1, 元素2, 元素3]
	集合c中的元素：[元素2, 元素3]
	集合c中的元素：[元素4, 元素5]
	集合c中是否包含元素5：true
	集合c中的元素：[元素4, 元素5]
	集合c中的元素个数：2
>这里只是简单的使用了下，非常简单。打印时，使用集合对象c就会输出集合中所有元素，这是因为Collection的实现类都重写了toString()方法，该方法可以一次性输出集合中的所有元素。

###二、遍历集合
既然容器中存放了我们需要使用的对象，那我们肯定需要去遍历集合来进行相应的操作。下面我们学习遍历集合的两种常用方法（有其他方法，但是不常用）：

####1、 使用Iteratir遍历集合
Iterator接口隐藏了各种Collection实现类的底层细节，提供了遍历Collection集合元素的统一编程接口。主要有以下3个方法：

- `boolean hasNext()`：如果仍有元素可以迭代，则返回 true。
- `Object next()`：返回迭代的下一个元素。
- `void remove()`：移除上一次next方法返回的元素。

请看代码：

	public static void main(String[] args){
        Collection c = new ArrayList();
        c.add("元素1");
        c.add("元素2");
        c.add("元素3");

        //获取集合c的迭代器
        Iterator iterator = c.iterator();
        while (iterator.hasNext()){
            Object o = iterator.next();
            System.out.println(o);
        }
    }

####2、 使用foreach遍历集合

	public static void main(String[] args){
        Collection c = new ArrayList();
        c.add("元素1");
        c.add("元素2");
        c.add("元素3");

        for (Object o:c) {
            System.out.println(o);
        }
    }
>很简单有没有。

注意：

1. 这里只是把集合元素的值传给迭代变量o，对它进行更改的话对集合元素没有任何影响。
2. 遍历过程中，不要去修改集合，否则会引起异常。

----------------------

问题提示：

1. 基本数据类型的装箱与拆箱。
2. 如果使用集合类时指定了泛型，往集合类中添加非指定类型的对象时，编译是通不过的，所以是不可以的。**但是**，如果不指定泛型的话，我们就可以为所欲为。。。


###三、Map接口
Map用于保存具有映射关系的数据，因此Map集合里保存着两组值，一组值用于保存Map里的key值，另外一组保存Map里面的Value，key和value都可以是任何引用类型的数据，Map的key值不允许重复，但是value值可以重复。并且，key与value值之间存在一对一的关系。
>类比数学函数中的x,y。

Map中定义了如下常用的方法：

- `void	clear()`：从Map集合中移除所有映射关系（清空）。
- `boolean containsKey(Object key)`：如果Map集合中包含指定键key，则返回 true。
- `boolean containsValue(Object value) `：如果Map集合中有一个或多个key映射到指定value，则返回 true。
- `Set<Map.Entry<K,V>> entrySet()`：返回该Map集合对应的Set集合（由key-value组成），每个集合元素都是Map.Entry（Map内部类）对象。
- `Object get(Object key)`：返回指定键所映射的值；如果Map不包含该键的映射关系，则返回 null。
- `boolean isEmpty()`：如果此Map未包含任何键-值映射关系，则返回 true。
- `Set<K> keySet()`：返回此Map中所有key组成的Set集合。
- `Object put(K key, V value)`：将指定键-值映射关系存入Map集合。
- `void	putAll(Map<? extends K,? extends V> m)`：从指定Map中将所有映射关系复制到此Map中。
- `Object remove(Object key)`：如果Map中存在key的映射关系，则将它与它映射的value从Map中移除。
- `int size()`：返回此Map中的键-值映射关系个数。
- `Collection<V> values()`：返回此Map中所有值的Collection集合。

你可能对`Set<Map.Entry<K,V>> entrySet()`感觉无法理解。暂时可以把Set理解成Collection，它里面存的对象是Map.Entry<K,V>类型的。还是无法理解吗？Entry是Map的内部类，Entry中有K类型的key值，以及V类型的value值（这里推荐了解下泛型）。好比如下代码：

	//注意，源码中并没有这样的代码，这里只是为了理解
	static class Entry<K, V>{
        K key;
        V value;
    }

在Entry中，包含了如下三个方法：

- `K getKey()`：返回key值。
- `V getValue()`：返回value值。
- `V setValue(V value)`：用指定的值替换value值。

>记住，一个Entry只封装了一个key-value映射关系。

现在是不是豁然开朗了。如果没有理解的话静下来多读几遍。
>上面的API都比较简单，这里就不再示例了。

###四、 Map集合遍历。
我们学会了Collection集合的遍历，但是如何进行Map集合的遍历呢？

你一定会想，肯定跟Collection集合一样，提供了遍历器Iterator。对的，确实是这样，但是，Map集合存储的是映射关系，key-value值。Map是如何实现一次遍历获取两个值的呢？

还记得刚刚的entry内部类吗？它对两个值封装一下，就变成了一个值。

而且，Map还有两个方法返回了两个Collection集合，我们已经学过遍历Collection集合的方法不是吗？

下面我们介绍4种遍历Map的方式：

	public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "a");
        map.put(2, "b");
        map.put(3, "ab");
        map.put(4, "ab");
        map.put(4, "ab");// 和上面相同，key值重复，会覆盖。

        System.out.println(map.size());

		// 第一种：	
        System.out.println("第一种：通过Map.keySet遍历key和value：");
        for (Integer in : map.keySet()) {
         	//map.keySet()返回的是所有key的集合
        	String str = map.get(in);//得到每个key多对用value的值
            System.out.println(in + "     " + str);
        }

        // 第二种：
        System.out.println("第二种：通过Map.entrySet使用iterator遍历key和value：");
        Iterator<Map.Entry<Integer, String>> it = map.entrySet().iterator();

        while (it.hasNext()) {
            Map.Entry<Integer, String> entry = it.next();
            System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
        }

        // 第三种：推荐，尤其是容量大时
        System.out.println("第三种：通过Map.entrySet遍历key和value");
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
        }

        // 第四种：
        System.out.println("第四种：通过Map.values()遍历所有的value，但不能遍历key");
        for (String v : map.values()) {
            System.out.println("value= " + v);
        }
	}

-----------
好了。

Collection与Map就介绍到这里，下次我们介绍Set集合。

##Set集合详解

今天我们说一说Set集合。上一篇我们提到过：Set集合是**无序**、**不可重复**的集合。这正是Set集合最重要的特点。

请看Set的继承树：

![](http://i.imgur.com/JbFkMXv.png)

Set集合中，我们常用的主要有：

- HashSet
- LinkedHashSet
- EnumSet
- TreeSet

下面，我们对这4个类进行学习。

###一、HashSet
HashSet是Set集合中，我们最常用的一个实现类。它主要由两部分构成：Hash和Set。

Set我们都知道是集合类，那么Hash呢？在Java中，Object类中提供了一个`hashCode()`的方法，不知道还有没有印象。这个方法返回该对象的哈希码值。

>哈希码值主要是让同一个类的对象按照自己不同的特征尽量的有与其他对象不同的哈希码。当然，这里只是尽量，不表示不同的对象哈希码完全不同。也有相同的情况，看程序员如何写哈希码的算法。

按照Hash算法，我们可以获取到元素的索引，并且根据这个索引获取到该元素在Set集合中插入/查找的位置。就好比数组的索引0，1，2，3。所以，使用hash会有更好的存储和查找性能。

HashSet的特点主要有：

1. 无序，不可重复。
2. 线程不安全。
3. 集合元素值可以为null。 

####hashCode与equals()

当向hashSet集合中存入一个元素时，HashSet会调用对象的hashcode()方法来得到该对象的hashcode值，然后根据该hashcode值决定该对象在HashSet中的存储位置。

注意：**HashSet集合判断两个元素相当的标准是两个对象通过`equals()`方法比较想等，并且两个对象的`hashCode()`方法返回值也相等。**

	class B{
	    @Override
	    public int hashCode() {
	        return 1;
	    }
	
	    @Override
	    public String toString() {
	        return "B";
	    }
	}	
	class C{
	    @Override
	    public boolean equals(Object o) {
	        return true;
	    }
	
	    @Override
	    public int hashCode() {
	        return 2;
	    }
	
	    @Override
	    public String toString() {
	        return "C";
	    }
	}
	
	public class Test {
	    HashSet hashSet = new HashSet();
        hashSet.add(new C());
        System.out.println("hashSet中的对象："+hashSet);
        hashSet.add(new C());
        System.out.println("hashSet中的对象："+hashSet);
        hashSet.add(new B());
        System.out.println("hashSet中的对象："+hashSet);
        
        //输出结果：
        //hashSet中的对象：[C]
		//hashSet中的对象：[C]
		//hashSet中的对象：[B, C]
	}

从代码可以看出：

	- 如果两个对象的`hashcode()`相等，并且`equals()`返回true，则存入失败。也就是说集合中元素不可重复。
	- Set集合是无序的，我们存入顺序为C->B，但是最后打印顺序为B->C。如果你存入更多的`hashcode()`值不同的对象会看得更加明显。

但是，这里是根据两个方法作为判断两个元素是否像相等。并且那么就会出现很多问题：

1. 如果hashCode()相等而equals()不等，到底应该存入哪一个？
2. 如果hashCode()不等但equals()相等，还会存入吗？

首先来看第一个问题：
**如果hashCode()相等而equals()不等，到底应该存入哪一个？**

我们修改B类以及C类的，使他们返回相同的`HashCode`值，但`equals()`方法返回false。


	class B{
	    @Override
	    public int hashCode() {
	        return 1;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        return false;
	    }
	
	    @Override
	    public String toString() {
	        return "B";
	    }
	}
	class C{
	    @Override
	    public boolean equals(Object o) {
	        return false;
	    }
	
	    @Override
	    public int hashCode() {
	        return 1;
	    }
	
	    @Override
	    public String toString() {
	        return "C";
	    }
	}
	
	public class Test {
	    public static void main(String[] args){
	        HashSet hashSet = new HashSet();
	        hashSet.add(new C());
	        System.out.println("hashSet中的对象："+hashSet);
	        hashSet.add(new B());
	        System.out.println("hashSet中的对象："+hashSet);
	
	        //输出结果：
	        //hashSet中的对象：[C]
			//hashSet中的对象：[C, B]
	    }
	}

竟然可以正常的存入，这是为什么呢？

在这种情况下，因为两个对象的hashCode()值相同，HashSet将试图把它们保存在同一个位置，但又不能替换，所以实际上会在这个位置用链式结构来保存多个对象。会导致访问集合中元素是性能下降。


我们接着看第二个问题：**如果hashCode()不等但equals()相等，还会存入吗？**

更改B类与C类的hashCode()值与equals()返回值如下：

	class B{
	    @Override
	    public int hashCode() {
	        return 2;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        return true;
	    }
	
	    @Override
	    public String toString() {
	        return "B";
	    }
	}
	class C{
	    @Override
	    public boolean equals(Object o) {
	        return true;
	    }
	
	    @Override
	    public int hashCode() {
	        return 1;
	    }
	
	    @Override
	    public String toString() {
	        return "C";
	    }
	}
	
	public class Test {
	    public static void main(String[] args){
	        HashSet hashSet = new HashSet();
	        hashSet.add(new C());
	        System.out.println("hashSet中的对象："+hashSet);
	        hashSet.add(new B());
	        System.out.println("hashSet中的对象："+hashSet);
	
	        //输出结果：
	        //hashSet中的对象：[C]
	        //hashSet中的对象：[C, B]
	    }
	}

还是可以存入。这是因为根据HashSet判断相等机制，C与B不相等，且在HashSet集合中C与B的索引位置不相同。

所以，我们在使用HashSet的时候要注意：

**如果需要把某个类的对象保存到HashSet集合中，重写这个类的`equals()`方法和`hashCode()`方法时，应该尽量保证两个对象通过`equals()`方法比较返回true时，它们的`hashCode()`方法返回值也相等。**


>由于HashSet集合采用hash码作为索引，所以在向HashSet中添加可变对象时，不要去修改HashSet集合中的对象，因为这样有可能导致该对象与集合中的其他对象相等，从而导致HashSet无法准确访问该对象。

###LinkedHashSet
LinkedHashSet是HashSet的子类。

LinkedHashSet是在HashSet的基础上使用链表来维护元素的次序，使元素变的有序。

LinkedHashSet需要维护元素的插入顺序，因此性能略低于HashSet的性能，但在迭代访问Set里的全部元素时将有很好的性能，因为它以链表来维护内部顺序。

LinkedHashSet的其他方面跟HashSet一致。

###TreeSet
TreeSet是Sorted接口的实现类，它可以使集合中的元素处于排序状态。

>由于TreeSet是有序的，所以它还提供了一系列的有关次序的方法。比如最大值（第一个）、最小值（第二个）、比某个值大（小）的元素等。。。可以去查看API文档学习，这里不再列出。

与HashSet集合采用hash算法来决定元素的储位置不同，TreeSet采用红黑树的数据结构来存储集合元素。它支持两种排序方法:自然排序和定制排序。

####自然排序

自然排序：TreeSet会调用集合元素的compareTo(Object obj)方法来比较元素之间的大小关系，然后将集合元素按升序排列。

>java常用类如Date、Time等都实现了该方法。如果我们需要存入我们自定义的类，则需要实现Comparable接口里的compareTo方法。

####定制排序
Comparator接口的compare方法。






