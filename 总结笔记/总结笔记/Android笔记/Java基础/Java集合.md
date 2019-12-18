#Java集合详解
容器，就是可以容纳其他Java**对象**的对象。
>对于基本类型，需要将其包装成对象类型后才能放到容器里。很多时候拆包装和解包装能够自动。

##Java集合接口
![](http://i.imgur.com/nRseCyB.png)

其中，Map接口没有继承自Collection接口，因为Map表示的是关联式容器而不是集合。但Java为我们提供了从Map转换到Collection的方法，可以方便的将Map切换到集合视图。上图中提供了Queue接口，却没有Stack，这是因为Stack的功能已被JDK1.6引入的Deque取代。

####实现类：
![](http://i.imgur.com/eYSWRvm.png)

##迭代器（Iterator）

JCF的迭代器（Iterator）为我们提供了遍历容器中元素的方法。只有容器本身清楚容器里元素的组织方式，因此迭代器只能通过容器本身得到。每个容器都会通过内部类的形式实现自己的迭代器。

	ArrayList<String> list = new ArrayList<>();
    list.add("Monday");
    list.add("Tuesday");
    list.add("Wensday");

    //实现方法一：获取迭代器
    Iterator iterator = list.iterator();
    while (iterator.hasNext()){
        String weekday = (String) iterator.next();
        System.out.println(weekday.toUpperCase());
    }
    //实现方法二：使用foreach
    for (String weekday:list) {
        System.out.println(weekday.toUpperCase());
    }

##ArrayList详解
ArrayList实现了List接口，它有几个特点：

- 是顺序容器，即元素存放的数据与放进去的顺序相同，
- 允许放入null元素，底层通过数组实现。
- 未实现同步，其余跟Vector大致相同。
- 每个ArrayList都有一个容量（capacity），表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。

>size(), isEmpty(), get(), set()方法均能在常数时间内完成。
>add()方法的时间开销跟插入位置有关，addAll()方法的时间开销跟添加元素的个数成正比。其余方法大都是线性时间。
>
>为追求效率，ArrayList没有实现同步（synchronized），如果需要多个线程并发访问，用户可以手动同步，也可使用Vector替代。

###方法解析
####set()
	public E set(int var1, E var2) {
        this.rangeCheck(var1);	//检测数组是否越界
        Object var3 = this.elementData(var1);	//获取被替换元素
        this.elementData[var1] = var2;	//把对象的引用插入到指定位置
        return var3;	//返回被替换的元素
    }
####get()
	public E get(int var1) {
        this.rangeCheck(var1);	//检测数组是否越界
        return this.elementData(var1);	//直接返回指定位置数组
    }
####add()
ArrayList添加元素用的是`add(E e)`，插入元素对应的方法是`add(int index, E e)`。这两个方法都是向容器中添加新元素，这可能会导致capacity不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过grow()方法完成的。

	private void grow(int minCapacity) {
	    int oldCapacity = elementData.length;
	    int newCapacity = oldCapacity + (oldCapacity >> 1);//原来的1.5倍
	    if (newCapacity - minCapacity < 0)
	        newCapacity = minCapacity;
	    if (newCapacity - MAX_ARRAY_SIZE > 0)
	        newCapacity = hugeCapacity(minCapacity);
	    elementData = Arrays.copyOf(elementData, newCapacity);//扩展空间并复制
	}

>add(int index, E e)需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。
####addAll()
`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本：

1. 在末尾添加的`addAll(Collection<? extends E> c)`方法
2. 从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。

>跟add()方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。
addAll()的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。
####remove()
`remove()`方法也有两个版本：

1. remove(int index)删除指定位置的元素
2. remove(Object o)删除第一个满足o.equals(elementData[index])的元素。

>删除操作是add()操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋null值。

	public E remove(int index) {
	    rangeCheck(index);
	    modCount++;
	    E oldValue = elementData(index);
	    int numMoved = size - index - 1;
	    if (numMoved > 0)
	        System.arraycopy(elementData, index+1, elementData, index, numMoved);
	    elementData[--size] = null; //清除该位置的引用，让GC起作用
	    return oldValue;
	}

##LinkList详解
LinkedList同时实现了List接口和Deque接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（Queue），同时又可以看作一个栈（Stack）。这样看来，LinkedList简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用LinkedList，一方面是因为Java官方已经声明不建议使用Stack类，更遗憾的是，Java里根本没有一个叫做Queue的类（它是个接口名字）。关于栈或队列，现在的首选是ArrayDeque，它有着比LinkedList（当作栈或队列使用时）有着更好的性能。
![](http://i.imgur.com/hVBs6Sh.png)
LinkedList底层通过双向链表实现，本节将着重讲解插入和删除元素时双向链表的维护过程，也即是之间解跟List接口相关的函数，而将Queue和Stack以及Deque相关的知识放在下一节讲。双向链表的每个节点用内部类Node表示。LinkedList通过first和last引用分别指向链表的第一个和最后一个元素。当链表为空的时候first和last都指向null。

	//Node内部类
	private static class Node<E> {
	    E item;
	    Node<E> next;
	    Node<E> prev;
	    Node(Node<E> prev, E element, Node<E> next) {
	        this.item = element;
	        this.next = next;
	        this.prev = prev;
	    }
	}

>LinkedList的实现方式决定了所有跟下标相关的操作都是线性时间，而在首段或者末尾删除元素只需要常数时间。为追求效率LinkedList没有实现同步（synchronized），如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装。

###方法解析
####add()
add()方法有两个版本：

1. add(E e)，该方法在LinkedList的末尾插入元素，因为有last指向链表末尾，在末尾插入元素的花费是常数时间。只需要简单修改几个相关引用即可；
2. add(int index, E element)，该方法是在指定下表处插入元素，需要先通过线性查找找到具体位置，然后修改相关引用完成插入操作。

源码如下：

	//add(E e)
	public boolean add(E e) {
	    final Node<E> l = last;
	    final Node<E> newNode = new Node<>(l, e, null);
	    last = newNode;
	    if (l == null)
	        first = newNode;//原来链表为空，这是插入的第一个元素
	    else
	        l.next = newNode;
	    size++;
	    return true;
	}

	//add(int index, E element)
	public void add(int index, E element) {
	    checkPositionIndex(index);//index >= 0 && index <= size;
	    if (index == size)//插入位置是末尾，包括列表为空的情况
	        add(element);
	    else{
	        Node<E> succ = node(index);//1.先根据index找到要插入的位置
	        //2.修改引用，完成插入操作。
	        final Node<E> pred = succ.prev;
	        final Node<E> newNode = new Node<>(pred, e, succ);
	        succ.prev = newNode;
	        if (pred == null)//插入位置为0
	            first = newNode;
	        else
	            pred.next = newNode;
	        size++;
	    }
	}
>上面代码中的`node(int index)`函数有一点小小的trick(设计)，因为链表双向的，可以从开始往后找，也可以从结尾往前找，具体朝那个方向找取决于条件index < (size >> 1)，也即是index是靠近前端还是后端。

####remove()

remove()方法也有两个版本：

1. 删除跟指定元素相等的第一个元素remove(Object o)
2. 删除指定下标处的元素remove(int index)。

两个删除操作都要：

1. 先找到要删除元素的引用
2. 修改相关引用，完成删除操作。在寻找被删元素引用的时候remove(Object o)调用的是元素的equals方法，而remove(int index)使用的是下标计数，两种方式都是线性时间复杂度。
>在步骤2中，两个revome()方法都是通过unlink(Node<E> x)方法完成的。这里需要考虑删除元素是第一个或者最后一个时的边界情况。

	//unlink(Node<E> x)，删除一个Node
	E unlink(Node<E> x) {
	    final E element = x.item;
	    final Node<E> next = x.next;
	    final Node<E> prev = x.prev;
	    if (prev == null) {//删除的是第一个元素
	        first = next;
	    } else {
	        prev.next = next;
	        x.prev = null;
	    }
	    if (next == null) {//删除的是最后一个元素
	        last = prev;
	    } else {
	        next.prev = prev;
	        x.next = null;
	    }
	    x.item = null;//let GC work
	    size--;
	    return element;
	}

####get()
`get(int index)`得到指定下标处元素的引用，通过调用上文中提到的node(int index)方法实现。

	public E get(int index) {
	    checkElementIndex(index);//index >= 0 && index < size;
	    return node(index).item;
	}

####set()
`set(int index, E element)`方法将指定下标处的元素修改成指定值，也是先通过node(int index)找到对应下表元素的引用，然后修改Node中item的值。

	public E set(int index, E element) {
	    checkElementIndex(index);
	    Node<E> x = node(index);
	    E oldVal = x.item;
	    x.item = element;//替换新值
	    return oldVal;
	}

##ArrayQueue
Java里有一个叫做Stack的类，却没有叫做Queue的类（它是个接口名字）。当需要使用栈时，Java已不推荐使用Stack，而是推荐使用更高效的ArrayDeque；既然Queue只是一个接口，当需要使用队列时也就首选ArrayDeque了（次选是LinkedList）。

要讲栈和队列，首先要讲Deque接口。Deque的含义是“double ended queue”，即双端队列，它既可以当作栈使用，也可以当作队列使用。

下表列出了Deque与Queue相对应的接口：

| Queue Method | Equivalent Deque Method | 说明 |
| :----:| :----: | :----: |
|add(e)|	addLast(e)|	向队尾插入元素，失败则抛出异常|
|offer(e)|	offerLast(e)|	向队尾插入元素，失败则返回false|
|remove()|	removeFirst()|	获取并删除队首元素，失败则抛出异常|
|poll()|	pollFirst()|	获取并删除队首元素，失败则返回null|
|element()|	getFirst()|	获取但不删除队首元素，失败则抛出异常|
|peek()|	peekFirst()|	获取但不删除队首元素，失败则返回null|


下表列出了Deque与Stack对应的接口：

| Stack Method | Equivalent Deque Method | 说明 |
| :----:| :----: | :----: |
|push(e)|	addFirst(e)|	向栈顶插入元素，失败则抛出异常|
|无|	offerFirst(e)|	向栈顶插入元素，失败则返回false|
|pop()|	removeFirst()|	获取并删除栈顶元素，失败则抛出异常|
|无|	pollFirst()|	获取并删除栈顶元素，失败则返回null|
|peek()|	peekFirst()|	获取但不删除栈顶元素，失败则抛出异常|
|无|	peekFirst()|	获取但不删除栈顶元素，失败则返回null|


上面两个表共定义了Deque的12个接口。添加，删除，取值都有两套接口，它们功能相同，区别是对失败情况的处理不同。一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值（false或null）。除非某种实现对容量有限制，大多数情况下，添加操作是不会失败的。虽然Deque的接口有12个之多，但无非就是对容器的两端进行操作，或添加，或删除，或查看。明白了这一点讲解起来就会非常简单。

从名字可以看出ArrayDeque底层通过数组实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即循环数组（circular array），也就是说数组的任何一点都可能被看作起点或者终点。ArrayDeque是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步；另外，该容器不允许放入null元素。
![](http://i.imgur.com/4015TZ0.png)

>上图中我们看到，head指向首端第一个有效元素，tail指向尾端第一个可以插入元素的空位。因为是循环数组，所以head不一定总等于0，tail也不一定总是比head大。

###方法解析
####addFirst()

`addFirst(E e)`的作用是在Deque的首端插入元素，也就是在head的前面插入元素，在空间足够且下标没有越界的情况下，只需要将elements[--head] = e即可。

实际需要考虑：1.空间是否够用，以及2.下标是否越界的问题。如果head为0之后接着调用addFirst()，虽然空余空间还够用，但head为-1，下标越界了。下列代码很好的解决了这两个问题。

//addFirst(E e)
public void addFirst(E e) {
    if (e == null)//不允许放入null
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;//2.下标是否越界
    if (head == tail)//1.空间是否够用
        doubleCapacity();//扩容
}
上述代码我们看到，空间问题是在插入之后解决的，因为tail总是指向下一个可插入的空位，也就意味着elements数组至少有一个空位，所以插入元素的时候不用考虑空间问题。

下标越界的处理解决起来非常简单，head = (head - 1) & (elements.length - 1)就可以了，这段代码相当于取余，同时解决了head为负值的情况。因为elements.length必需是2的指数倍，elements - 1就是二进制低位全1，跟head - 1相与之后就起到了取模的作用，如果head - 1为负数（其实只可能是-1），则相当于对其取相对于elements.length的补码。