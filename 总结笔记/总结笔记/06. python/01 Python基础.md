6.13 Bytes

Python 对 bytes 类型的数据用带 b 前缀的单引号或双引号表示:  `x = b'ABC'`

要注意区分`'ABC'`和` b'ABC'`，前者是 str，后者虽然内容显示得和前者一 样，但 bytes 的每个字符都只占用一个字节。 

```python
'ABC'.encode('ascii')  #以Unicode表示的str通过encode()方法可以编码为指定的bytes
b'ABC'.decode('ascii') #如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为str，就需要decode()方法。
len('ABC') #计算str包含多少个字符，如果换成bytes，就是计算字节数。
```

为了让Python解析器读取源代码时按UTF-8编码读取，通常在文件开头写上这两行：

```python
#!/usr/bin/env python3	告诉Linux/OS X系统，这是一个Python可执行程序
#-*- coding: utf-8 -*-  告诉解释器按照UTF-8编码读取源代码
```

> 在bytes中，无法显示为ASCII字符的字节，用\x##表示。为了避免乱码问题，应当始终坚持使用UTF-8。

如果bytes中只有一小部分无效的字节，可以传入`errors='ignore'`忽略错误的字节；`xxx.decode('utf-8', errors='ignore')`

#### 6.14 dict与list的优缺点：

和 list 比较，dict 有以下几个特点：
1. 查找和插入的速度极快，不会随着key的增加而增加；
2. 需要占用大量的内存，内存浪费多。 

而 list 相反：
1. 查找和插入的时间随着元素的增加而增加;；
2. 占用空间小，浪费内存很少。

dict是用空间来换取时间的一种方法。需要牢记的一条是**dict的key必须是不可变对象**。在dict内部通过key计算位置的算法称为哈希算法（Hash）。

#### 6.15 Set

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

要创建一个set，需要提供一个list作为输入集合：

```python
s = set([1, 2, 3])
```

重复元素在set中自动被过滤。

```python
s.remove(4)	# 移除set中的元素
```

set 可以看成数学意义上的无序和无重复元素的集合，因此，两个 set 可 以做数学意义上的交集、并集等操作: 

```python
>>> s1 = set([1, 2, 3])
>>> s2 = set([2, 3, 4])
>>> s1 & s2
{2, 3}
>>> s1 | s2
{1, 2, 3, 4}
```

**Set与Dict一样，不可放入可变元素。**

### 7. 高级函数

#### 7.1 函数基础

函数名其实就是指向一个函数对象的引用，完全可以把函数名赋给一个变量，相当于给这个函数起了一个别名。

##### 7.1.1 定义

```python
def func_name(param1, param2):
    return 3	# 如果没有return语句，函数执行完毕也会返回结果，只是结果为None。
```

##### 7.1.2 导入函数

```python
from file_name import func_name  # 导入filename文件（.py）中的func_name函数 
```

##### 7.1.3 函数调用

调用函数很简单的，通过 `函数名()` 即可完成对函数的调用

能否将函数调用放在函数定义的上方？

答：不能！因为在使用函数名调用函数之前，必须要保证 `Python` 已经知道函数的存在。否则控制台会提示 `NameError: name 'say_hello' is not defined` (名称错误：say_hello 这个名字没有被定义)

##### 7.1.4 函数的文档注释

在开发中，如果希望给函数添加注释，应该在定义函数的下方，使用连续的三对引号。在连续的三对引号之间编写对函数的说明文字，在函数调用位置，使用快捷键 `CTRL + Q` 可以查看函数的说明信息

注意：因为函数体相对比较独立，函数定义的上方，应该和其他代码（包括注释）保持2行间隔。

##### 7.1.5 空函数

如果想定义一个什么事也不做的空函数，可以用 pass 语句；

pass 语句什么都不做，那有什么用?实际上 pass 可以用来作为**占位符**， 比如现在还没想好怎么写函数的代码，就可以先放一个 pass，让代码能运行起来。 如果代码中什么都没做，且缺少pass语句，代码运行就会有语法错误。

#### 7.2 函数的参数

Python的函数定义非常简单，但灵活性非常大。除了正常定义的必须参数外，还可以使用**默认参数**、**可变参数**以及**关键字参数**，使得函数定义出来的接口，不但能处理复杂的参数，还可以简化调用者的代码。

##### 7.2.1 默认参数

```python
def power(x, n = 2)	# 该情况下，如果不传入n的值，会自动赋值2
```

**缺省参数使用注意：**

1. 必须保证带有默认值的缺省参数在参数列表末尾；
2. 不按顺序提供部分默认参数时，需要把参数名加上；
3. 默认参数如果没有指向不可变参数，可能会造成函数出错（逻辑错误）。同时，由于对象不变，多任务环境下同时读取对象不需要加锁。所以，编写程序时，尽量使用不变对象。

##### 7.2.2 多值参数

有时可能需要一个函数能够处理的参数个数是不确定的，这个时候，就可以使用多值参数。`python` 中有两种多值参数：

1. 参数名前增加一个 `*` 可以接收元组；
2. 参数名前增加两个 `*` 可以接收字典。

一般在给多值参数命名时，习惯使用以下两个名字

1. `*args` —— 存放元组参数，前面有一个 `*`，`args` 是 `arguments` 的缩写，有变量的含义；
2. `**kwargs` —— 存放字典参数，前面有两个 `*`，`kw` 是 `keyword` 的缩写，`kwargs` 可以记忆键值对参数；

###### 1）可变参数

```python
def calc(*numbers):
	pass
```

当然，这里也可以定义一个list或tuple实现同样的效果，但是与此相比，可变参数仅仅在参数前面加了一 个*号。

在函数内部，参数 numbers 接收到的是一个 tuple，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括 0 个参数。

Python允许在list或tuple前面加一个*号，把list或tuple的元素变成可变参数传进去。即拆包。

###### 2）关键字参数

关键字参数允许传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

```python
def func(param1, param2, **kw):	#用**kw表示关键字参数
	pass
func('Adam', 45, gender='M', job='Engineer')
```

和可变参数类似，也可以先组装一个dict，然后通过**param把dict传入。注意，这边传入的是一份拷贝，不会影响到原来的的值。

###### 3）命名关键字参数

也可以使用命名关键字参数，规定好可以接受的关键字，关键字参数调用必须传入参数名。

```python
def person(name, age, *, city, job):	#这里*是特殊分隔符，不是参数。
    pass
```

###### 4）拆包

在调用带有多值参数的函数时，如果希望：

1. 将一个元组变量，直接传递给 `args`
2. 将一个字典变量，直接传递给 `kwargs`

就可以使用拆包，简化参数的传递，拆包的方式是：

1. 在元组变量前，增加一个 `*`
2. 在字典变量前，增加两个 `*`

##### 7.2.3 参数检查

我们可以在自定义的函数中对参数类型做检查，只允许某些类型的参数。数据类型检查可以用内置函数`isinstance()`实现。

```python
if not isinstance(x, (int, float)):
	raise TypeError('bad operand type')	#自定义错误语言
```

##### 7.2.4 参数组合

除了可变参数无法与命令关键字参数混合，其他参数类型都可组合使用。

参数定义的顺序必须是：必选参数、默认参数、可变参数/命名关键字参数、关键字参数。

```python
def f1(a, b, c=0, *args, **kw):	# 必选参数、默认参数、可变参数、关键字参数
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)
def f2(a, b, c=0, *, d, **kw):  # 必选参数、默认参数、命名关键字参数、关键字参数
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
```

```python
>>> f1(1, 2)
a = 1 b = 2 c = 0 args = () kw = {}
>>> f1(1, 2, c=3)
a = 1 b = 2 c = 3 args = () kw = {}
>>> f1(1, 2, 3, 'a', 'b')
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {}
>>> f1(1, 2, 3, 'a', 'b', x=99)
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {'x': 99}
>>> f2(1, 2, d=99, ext=None)
a = 1 b = 2 c = 0 d = 99 kw = {'ext': None}
```

最神奇的是通过一个 tuple 和 dict，你也可以调用上述函数：

```python
>>> args = (1, 2, 3, 4)
>>> kw = {'d': 99, 'x': '#'}
>>> f1(*args, **kw)
a = 1 b = 2 c = 3 args = (4,) kw = {'d': 99, 'x': '#'}
>>> args = (1, 2, 3)
>>> kw = {'d': 88, 'x': '#'}
>>> f2(*args, **kw)
a = 1 b = 2 c = 3 d = 88 kw = {'x': '#'}
```

所以，对于任意函数，都可以通过类似 `func(*args, **kw)` 的形式调用它， 无论它的参数是如何定义的。 

所以命名关键字参数到底有什么用？？？命名的关键字参数是为了限制调用者可以传入的参数名，同时可以提供默认值。 

使用*args 和**kw 是 Python 的习惯写法，当然也可以用其他参数名，但最好使用习惯用法。 

#### 7.3 函数的返回值

Python函数可以返回一个元组（实现多个返回值）。如果一个函数返回的是元组，括号可以省略。

在 `Python` 中，可以将一个元组使用赋值语句同时赋值给多个变量，变量的数量需要和元组中的元素数量保持一致。

Python专有：`a, b = b, a  # 将两个变量互换值`

如果没有return语句，函数执行完毕后也会返回结果，只是结果为None。return None可以简写为return。

#### 7.4 递归函数

精髓：一个函数在内部调用自己。

递归函数特点：

1. 函数内部的代码是相同的，只是针对参数不同，处理的结果不同
2. 当参数满足一个条件时，函数不再执行（**被称为递归的出口，否则会出现死循环！**）

提示：递归是一个编程技巧，初次接触递归会感觉有些吃力！在处理不确定的循环条件时，格外的有用，例如：遍历整个文件目录的结构。

##### 7.4.1 尾递归

使用递归函数需要注意防止栈溢出。在计算机中，函数调用是通过栈 (stack)这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的， 所以，递归调用的次数过多，会导致栈溢出。 

解决递归栈溢出的方法是通过**尾递归**优化，事实上尾递归优化和循环的效果是一样的，所以，把循环看成是一种特殊的尾递归函数也是可以的。 

尾递归是指，在函数返回的时候，调用自身本身，并且，return 语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。 

尾递归调用时，如果做了优化，栈不会增长，因此，无论多少次调用也不会导致栈溢出。 

```python
def fact(n):
	  return fact_iter(n, 1)
	 
def fact_iter(num, product):
	  if num == 1:
	  	  return product
	  return fact_iter(num - 1, num * product)
```

遗憾的是，大多数编程语言没有针对尾递归做优化，Python 解释器也没有做优化，所以，即使把上面的 fact(n)函数改成尾递归方式，也会导致栈溢出。 

### 8. 高级操作

#### 8.1 切片

| 描述 | Python 表达式      | 结果    | 支持的数据类型     |
| :--: | ------------------ | ------- | ------------------ |
| 切片 | "0123456789"[::-2] | "97531" | 字符串、列表、元组 |

切片方法适用于字符串、列表、元组：

1. 切片使用索引值来限定范围，从一个大的字符串中切出小的字符串；
2. 列表和元组都是有序的集合，都能够通过索引值获取到对应的数据
3. 字典是一个无序的集合，是使用键值对保存数据

```python
字符串[开始索引:结束索引:步长]  
# 指定的区间属于左闭右开型 [开始索引, 结束索引)；
# 从头开始，开始索引数字可以省略，冒号不能省略；
# 到末尾结束，结束索引数字可以省略，冒号不能省略；
# 步长默认为1，如果连续切片，数字和冒号都可以省略。
# 在 Python 中不仅支持顺序索引，同时还支持倒序索引，所谓倒序索引就是从右向左计算索引，最右边的索引值是 -1，依次递减
```

#### 8.2 迭代

如果给定一个 list 或 tuple，我们可以通过 for 循环来遍历这个 list 或 tuple，这种遍历我们称为迭代(Iteration)。 

只要是可迭代对象，无论有无下标，都可以迭代 。可以通过collections模块的Iterable类型判断一个对象是否是可迭代对象。

```python
from collections import Iterable  
isinstance('abc', Iterable) # str 是否可迭代 True  
```

如果需要使用下标来进行操作，可以使用Python内置的enumerate函数把list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身。

```python
for i, value in enumerate(['A', 'B', 'C']):
    print(i, value)
```

#### 8.3 列表生成式

列表生成式即 List Comprehensions，是 Python 内置的非常简单却强大的可以用来创建 list 的生成式。 

```python
[x * x for x in range(1, 11) [for y in xxx][if x ??]]  #可嵌套双层循环，if语句
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

#### 8.4 生成器

通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制， 列表容量肯定是有限的。而且，创建一个包含 100 万个元素的列表，不 仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面 绝大多数元素占用的空间都白白浪费了。 

所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢?这样就不必创建完整的 list，从而节省大量的空间。

在 Python 中，这种一边循环一边计算的机制，称为生成器:generator。 生成器是一类特殊的迭代器。

##### 8.3.1 生成方法

1. 列表生成式的`[]`替换为`()`

2. 如果逻辑太过复杂，可以通过函数来实现，只需要在函数输出时把打印（print）替换成yield。如果函数中包含yield关键字，那么这个函数就是一个generator。

   用for循环调用generator时，会拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中。

如果需要获取其中内容：	

- 可以通过`next()`获取下一个值，没有更多值时，抛出StopIteration的错误。
- 可以通过`for...in`循环获取，并且不需要担心StopIteration错误。

##### 8.3.2 yield

yield关键字有两点作用：

1. 保存当前运行状态（断点），然后暂停执行，即将生成器（函数）挂起
2. 将yield关键字后面表达式的值作为返回值返回，此时可以理解为起到了return的作用

##### 8.3.3 三种唤醒yield的方式

我们除了可以使用next()函数来唤醒生成器继续执行外，还可以使用`send()`函数来唤醒执行。使用`send()`函数的一个好处是可以在唤醒的同时向断点处传入一个附加数据。

```python
In [10]: def gen():
   ....:     i = 0
   ....:     while i<5:
   ....:         temp = yield i
   ....:         print(temp)
   ....:         i+=1
   ....:
```

###### 1）使用`seed()`唤醒

```
In [43]: f = gen()

In [44]: next(f)
Out[44]: 0

In [45]: f.send('haha')
haha
Out[45]: 1

In [46]: next(f)
None
Out[46]: 2

In [47]: f.send('haha')
haha
Out[47]: 3

In [48]:
```

###### 2）使用`next()`唤醒

```
In [11]: f = gen()

In [12]: next(f)
Out[12]: 0

In [13]: next(f)
None
Out[13]: 1

In [17]: next(f)
None
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-17-468f0afdf1b9> in <module>()
----> 1 next(f)

StopIteration:
```

###### 3）使用`__next__()`方法唤醒

该方法不常用

```
In [18]: f = gen()

In [19]: f.__next__()
Out[19]: 0

In [20]: f.__next__()
None
Out[20]: 1

In [24]: f.__next__()
None
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-24-39ec527346a9> in <module>()
----> 1 f.__next__()

StopIteration:
```

#### 8.4 迭代器

可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：Iterator。

生成器都是Iterator对象，但是list、dict、str等虽然是Iterable，却不是Iterator。把list、dict、str等Iterable变成Iterator可以使用`iter()`函数。

```python
isinstance(iter([]), Iterator)  # iter()函数实际上是调用了可迭代对象的__iter__方法。
True
```

 为什么list、dict、str等数据类型不是Iterator？

答：Python 的 Iterator 对象表示的是一个数据流，Iterator 对象可以被`next()`函数调用并不断返回下一个数据，直到没有数据时抛出 `StopIteration` 错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过 `next()`函数实现按需计算下一 个数据，所以 Iterator 的计算是**惰性**的，只有在需要返回下一个数据时 它才会计算。 

Iterator 甚至可以表示一个无限大的数据流，例如全体自然数。而使用 list 是永远不可能存储全体自然数的。

Python的for循环本质上就是通过不断调用next()函数实现的。

除了for循环能接收可迭代对象，list、tuple等也可以接收。

##### 8.4.1 迭代器的实现

实际上，在使用`next()`函数的时候，调用的就是迭代器对象的`__next__`方法（Python3中是对象的`__next__`方法，Python2中是对象的next()方法）。所以，我们要想构造一个迭代器，就要实现它的`__next__`方法。但这还不够，python要求迭代器本身也是可迭代的，所以我们还要为迭代器实现`__iter__`方法，而`__iter__`方法要返回一个迭代器，迭代器自身正是一个迭代器，所以迭代器的`__iter__`方法返回自身即可。

**一个实现了`__iter__`方法和`__next__`方法的对象，就是迭代器。**

```python
class MyList(object):
    """自定义的一个可迭代对象"""
    def __init__(self):
        self.items = []

    def add(self, val):
        self.items.append(val)

    def __iter__(self):
        myiterator = MyIterator(self)
        return myiterator


class MyIterator(object):
    """自定义的供上面可迭代对象使用的一个迭代器"""
    def __init__(self, mylist):
        self.mylist = mylist
        # current用来记录当前访问到的位置
        self.current = 0

    def __next__(self):
        if self.current < len(self.mylist.items):
            item = self.mylist.items[self.current]
            self.current += 1
            return item
        else:
            raise StopIteration

    def __iter__(self):
        return self


if __name__ == '__main__':
    mylist = MyList()
    mylist.add(1)
    mylist.add(2)
    mylist.add(3)
    mylist.add(4)
    mylist.add(5)
    for num in mylist:
        print(num)
```



### 9. 函数式编程

函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。 

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数。

Python 对函数式编程提供部分支持。由于 Python 允许使用变量，因此，Python 不是纯函数式编程语言。 

#### 9.1 高阶函数

既然**变量可以指向函数**，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

##### 9.1.1 map/reduce

Python内建了map()和reduce()函数。

`map()`函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。

```python
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

reduce() 把一个函数作用在一个序列[x1, x2, x3, ...] 上，这个函数必须接收两个参数，reduce 把结果继续和序列的下一个元素做累积计算，其效果就是: 

```python
from functools import reduce
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4) 
```

##### 9.1.2 filter

Python内建的`filter()`函数用于过滤序列。

和`map()`类似，`filter()`也接收一个函数和一个序列。和`map()`不同的时，` filter()`把传入的函数依次作用于每个元素，然后根据返回值是 True 还 是 False 决定保留还是丢弃该元素。 

`filter()`函数返回的是一个Iterator，也就是一个惰性序列。

##### 9.1.3 sorted

Python 内置的 `sorted()` 函数就可以对 list 进行排序。

此外，sorted()函数也是一个高阶函数，它还可以接收一个 key 函数来实现自定义的排序，例如按绝对值大小排序: 

```python
>>> sorted([36, 5, -12, 9, -21], key=abs)	# key指定的函数将作用于list的每一个元素上
[5, 9, -12, -21, 36]
```

#### 9.2 返回函数

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。

假定求和函数，如果不需要立刻求和，而是在后面的代码中，根据需要再计算怎么办?我们可以不返回求和的结果，而是返回求和的函数：

```python
def lazy_sum(*args):	#执行函数后，返回一个函数，继续执行这个函数才返回值。这种结构叫做“闭包（Closure）”
    def sum():
         ax = 0
		 for n in args:
         	ax = ax + n
   		return ax
   	return sum
```

##### 9.2.1 闭包

```python
# 定义一个函数
def test(number):

    # 在函数内部再定义一个函数，并且这个函数用到了外边函数的变量，那么将这个函数以及用到的一些变量称之为闭包
    def test_in(number_in):
        print("in test_in 函数, number_in is %d" % number_in)
        return number+number_in
    # 其实这里返回的就是闭包的结果
    return test_in

# 给test函数赋值，这个20就是给参数number
ret = test(20)

# 注意这里的100其实给参数number_in
print(ret(100))

#注 意这里的200其实给参数number_in
print(ret(200))
```

由于闭包引用了外部函数的局部变量，则外部函数的局部变量没有及时释放，消耗内存

返回闭包时牢记的一点就是：**返回函数不要引用任何循环变量，或者后续会发生变化的变量**。 如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变。

闭包可以提高代码可复用性。比如可以实现函数复用

**修改闭包全局变量**

```python
def counter(start=0):
    def incr():
        nonlocal start  # 这里如果有修改start的值就会变成局部变量，所有如果声明，会报错。Python特性
        start += 1
        return start
    return incr

c1 = counter(5)
print(c1())
print(c1())

c2 = counter(50)
print(c2())
print(c2())

print(c1())
print(c1())

print(c2())
print(c2())
```

#### 9.3 匿名函数

关键字lambda表示匿名函数，冒号前面的x表示函数参数。

匿名函数有个限制，就是只能有一个表达式，不用写return，返回值就是该表达式的结果。

#### 9.4  装饰器

在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。本质上，装饰器就是一个返回函数的高阶函数。

装饰器在@符号出现的时候就进行装饰（调用之前）。

##### 9.4.1 装饰器实现

```python
def w1(func):
    def inner():
        # 验证代码
        @functools.wraps(func)  # 需要把原始函数的`__name__`等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。
        return func()
    return inner

@w1
def f1():
    print('f1')
```

由于函数也是一个对象，而且函数对象可以被赋值给变量，所以，通过变量也能调用该函数。

装饰时执行了两步：

1. 执行w1函数 ，并将 @w1 下面的函数作为w1函数的参数，即：**@w1 等价于 w1(f1)** 。
2. 将执行完的w1函数返回值 赋值 给@w1下面的函数的函数名f1 即将w1的返回值再重新赋值给 f1。

##### 9.4.2 适用场景

1. 引入日志
2. 函数执行时间统计
3. 执行函数前预备处理
4. 执行函数后清理功能
5. 权限校验等场景
6. 缓存

##### 9.4.3 装饰器示例

###### 1）无参数的函数

```python
from time import ctime, sleep

def timefun(func):
    def wrapped_func():
        print("%s called at %s" % (func.__name__, ctime()))
        func()
    return wrapped_func

@timefun
def foo():
    print("I am foo")

foo()
sleep(2)
foo()
```

上面代码理解装饰器执行行为可理解成

```
foo = timefun(foo)
# foo先作为参数赋值给func后,foo接收指向timefun返回的wrapped_func
foo()
# 调用foo(),即等价调用wrapped_func()
# 内部函数wrapped_func被引用，所以外部函数的func变量(自由变量)并没有释放
# func里保存的是原foo函数对象
```

###### 2）被装饰的函数有参数

```python
from time import ctime, sleep

def timefun(func):
    def wrapped_func(a, b):
        print("%s called at %s" % (func.__name__, ctime()))
        print(a, b)
        func(a, b)
    return wrapped_func

@timefun
def foo(a, b):
    print(a+b)

foo(3,5)
sleep(2)
foo(2,4)
```

###### 3）被装饰的函数有不定长参数

```python
from time import ctime, sleep

def timefun(func):
    def wrapped_func(*args, **kwargs):
        print("%s called at %s"%(func.__name__, ctime()))
        func(*args, **kwargs)
    return wrapped_func

@timefun
def foo(a, b, c):
    print(a+b+c)

foo(3,5,7)
sleep(2)
foo(2,4,9)
```

###### 4）装饰器中的return

```python
from time import ctime, sleep

def timefun(func):
    def wrapped_func():
        print("%s called at %s" % (func.__name__, ctime()))
        func()
    return wrapped_func

@timefun
def foo():
    print("I am foo")

@timefun
def getInfo():
    return '----hahah---'

foo()
sleep(2)
foo()


print(getInfo())
```

执行结果:

```python
foo called at Fri Nov  4 21:55:35 2016
I am foo
foo called at Fri Nov  4 21:55:37 2016
I am foo
getInfo called at Fri Nov  4 21:55:37 2016
None
```

如果修改装饰器为`return func()`，则运行结果：

```python
foo called at Fri Nov  4 21:55:57 2016
I am foo
foo called at Fri Nov  4 21:55:59 2016
I am foo
getInfo called at Fri Nov  4 21:55:59 2016
----hahah---
```

一般情况下为了让装饰器更通用，可以有return

###### 5）装饰器带参数

```python
#decorator2.py

from time import ctime, sleep

def timefun_arg(pre="hello"):
    def timefun(func):
        def wrapped_func():
            print("%s called at %s %s" % (func.__name__, ctime(), pre))
            return func()
        return wrapped_func
    return timefun

# 下面的装饰过程
# 1. 调用timefun_arg("itcast")
# 2. 将步骤1得到的返回值，即time_fun返回， 然后time_fun(foo)
# 3. 将time_fun(foo)的结果返回，即wrapped_func
# 4. 让foo = wrapped_fun，即foo现在指向wrapped_func
@timefun_arg("itcast")
def foo():
    print("I am foo")

@timefun_arg("python")
def too():
    print("I am too")

foo()
sleep(2)
foo()

too()
sleep(2)
too()
```

可以理解为

```
foo()==timefun_arg("itcast")(foo)()
```

##### 9.4.4 类装饰器

装饰器函数其实是这样一个接口约束，它必须接受一个callable对象作为参数，然后返回一个callable对象。在Python中一般callable对象都是函数，但也有例外。只要某个对象重写了 `__call__()` 方法，那么这个对象就是callable的。

```python
class Test():
    def __call__(self):
        print('call me!')

t = Test()
t()  # call me
```

类装饰器demo

```python
class Test(object):
    def __init__(self, func):
        print("---初始化---")
        print("func name is %s"%func.__name__)
        self.__func = func
    def __call__(self):
        print("---装饰器中的功能---")
        self.__func()
#说明：
#1. 当用Test来装作装饰器对test函数进行装饰的时候，首先会创建Test的实例对象
#   并且会把test这个函数名当做参数传递到__init__方法中
#   即在__init__方法中的属性__func指向了test指向的函数
#
#2. test指向了用Test创建出来的实例对象
#
#3. 当在使用test()进行调用时，就相当于让这个对象()，因此会调用这个对象的__call__方法
#
#4. 为了能够在__call__方法中调用原来test指向的函数体，所以在__init__方法中就需要一个实例属性来保存这个函数体的引用
#   所以才有了self.__func = func这句代码，从而在调用__call__方法中能够调用到test之前的函数体
@Test
def test():
    print("----test---")
test()
showpy()#如果把这句话注释，重新运行程序，依然会看到"--初始化--"
```

运行结果如下：

```python
---初始化---
func name is test
---装饰器中的功能---
----test---
```

#### 9.5 偏函数

Python 的 functools 模块提供了很多有用的功能，其中一个就是偏函数 (Partial function)。要注意，这里的偏函数和数学意义上的偏函数不一 样。 

`functools.partial` 就是帮助我们创建偏函数的，就是把一个函数的某些参数给固定住（默认值），返回一个新的函数，调用这个新函数会更简单。

```python
import funtools
int2 = funtools.partial(int, base = 2)
```

创建偏函数时，实际上可以接收函数对象、*args 和**kw 这 3 个 参数。例如：

```python
max2 = functools.partial(max, 10) # 实际上会把 10 作为*args 的一部分自动加到左边
max2(5, 6, 7) <==> max(10, 5, 6, 7)	#结果为10
```

### 10 . 模块和包

#### 10.1 模块

##### 10.1.1 模块定义

在Python中，一个.py文件就称之为一个模块（Module）。使用模块可以大大提高了代码的可维护性，还可以避免函数名和变量名的冲突。相同名字的函数和变量完全可以分别存在不同的模块中。

模块就好比是工具包，在模块中定义的全局变量、函数、类都是提供给外界直接使用的工具，要想使用这个工具包中的工具，就需要先导入这个模块。

##### 10.1.2 模块导入

模块有两种导入方式：

**1）import导入**

一次性把模块中所有工具全部导入，导入之后，可以通过`模块名.`使用模块提供的工具——全局变量、函数、类。

```python
import 模块名1，模块名2 [as 模块别名]
```

如果模块的名字太长，可以使用`as`指定m模块的名称，以方便在代码中的使用，模块别名应该符合大驼峰命名法。

**2）from...import导入**

如果希望从某一个模块中，导入部分工具，就可以使用`from...import`的方式。

```python
from 模块名1 import 工具名
from 模块名2 import *  # 从模块导入所有工具，但是极不推荐这种方式，因为函数重名并没有任何的提示，出现问题不好排查。
```

这种方法导入后，不需要使用`模块名.`，可以直接使用模块提供的工具——全局变量、函数、类。

**3）问题结局**

如果两个模块，存在同名的函数，那么后导入模块的函数，会覆盖掉先导入的函数。

开发时`import`代码应该统一写在代码的顶部，更容易及时发现冲突。一旦发现冲突，可以使用`as`关键字给其中一个工具起一个别名。

##### 10.1.3 模块搜索顺序

`Python`中每一个模块都有一个内置属性`__file__`可以查看模块的完整路径。

当我们试图加载一个模块时，Python 会在指定的路径下搜索对应的.py 文件，如果找不到，就会报错。

默认情况下，Python 解释器会搜索当前目录、所有已安装的内置模块和第三方模块。搜索路径存放在 sys 模块的 path 变量中：

```python
>>> import sys
>>> sys.path
['',
'/Library/Frameworks/Python.framework/Versions/3.4/lib/python34.zip',
'/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4',
'/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/plat-darwin',
'/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/lib-dynload',
'/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/site-packages']
```

 如果我们要添加自己的搜索目录，有两种方法：

1. 直接修改 sys.path，添加要搜索的目录: 

```python
>>> import sys
>>> sys.path.append('/Users/michael/my_py_scripts') #这种方法是在运行时修改，运行结束后失效。
```

2. 设置环境变量 PYTHONPATH，该环境变量的内容会被自动添加到模块搜索路径中。设置方式与设置 Path 环境变量类似。注意只需要添加你自己的搜索路径，Python 自己本身的搜索路径不受影响。 

##### 10.1.4 模块的原则

1. 每一个文件都应该是可以被导入的；
2. 任何模块代码的第一个字符串都被视为模块的文档注释；
3. 自己创建模块时要注意命名，不能和 Python 自带的模块名称冲突；

`__name__`属性可以做到，测试模块的代码只在测试情况下被运行，而在被导入时不会被执行。

`__name__`是Python的一个内置属性，记录着一个字符串。如果是被其他文件导入的，`__name__`就是模块名，如果是当前执行的程序，`__name__`是`__main__`。这个特性可以让一个模块通过命令运行时执行一些额外的代码，最常见的就是运行测试。

```python
# 根据 __name__ 判断是否执行下方代码
if __name__ == "__main__":
    main()
```

##### 10.1.5 重新导入模块

模块被导入后，`import module`不能重新导入模块，重新导入需用`reload`

```python
from imp import reload
reload(module)  # 重新导入模块
```

如果导入一个变量，在程序中使用时记得使用global修饰，否则就是定义一个局部变量。

#### 10.2 包

但是模块也可能会冲突，使用包可以避免模块名冲突。

##### 10.2.1 包定义

包是一个包含多个模块的特殊目录，目录下有一个特殊的文件`__init__.py`，包名的命名方式和变量名一致，小写字母 + `_`。这个文件是必须存在的，否则，Python就把这个目录当成普通目录，而不是包。`__init.py__`可以是空文件，也可以有Python代码，因为`__init__`本身就是一个模块，而它的模块名就是包名。

要在外界使用包中的模块时，需要在`__init__.py`中指定对外界提供的模块列表。

可以有多级目录，组成多级层次的包结构。

##### 10.2.2 包导入

使用包的好处：使用`import 包名`可以一次性导入包中的所有模块

```python
# 从当前目录导入模块列表
from . import 模块1
from . import 模块2
```

引入包以后，模块的名字就变为 包名.模块名。

#### 12.3 安装第三方模块

在 Python 中，安装第三方模块，是通过包管理工具 pip 完成的。 Mac 或 Linux会默认安装 pip 。

一般来说，第三方库都会在 Python 官方的 pypi.python.org 网站注册，要安装一个第三方库，必须先知道该库的名称，可以在官网或者 pypi 上 搜索，比如 Pillow 的名称叫 Pillow，因此，安装 Pillow 的命令就是: 

```python
pip install Pillow  # 安装第三方模块
pip uninstall Pillow  # 卸载第三方模块
```

#### 12.4 发布模块

如果希望自己开发的模块，分享给其他人，可以按照以下步骤操作：

1. 制作发布压缩包

   1. 创建`setup.py`

      ```python
      from distutils.core import setup
      
      setup(name="hm_message",  # 包名
            version="1.0",  # 版本
            description="itheima's 发送和接收消息模块",  # 描述信息
            long_description="完整的发送和接收消息模块",  # 完整描述信息
            author="itheima",  # 作者
            author_email="itheima@itheima.com",  # 作者邮箱
            url="www.itheima.com",  # 主页
            py_modules=["hm_message.send_message",
                        "hm_message.receive_message"])
      ```

      有关字典参数的详细信息，可以参阅官方网站：<https://docs.python.org/2/distutils/apiref.html>

   2. 构建模块

      ```
      $ python3 setup.py build
      ```

   3. 生成发布压缩包

      ```
      $ python3 setup.py sdist
      ```

2. 安装模块

   ```
   $ tar -zxvf hm_message-1.0.tar.gz 
   $ sudo python3 setup.py install
   ```

   如果需要卸载模块，直接从安装目录下，把安装模块的目录删除就可以。

   ```
   $ cd /usr/local/lib/python3.5/dist-packages/
   $ sudo rm -r hm_message*
   ```

##### 15.3.3 pip安装第三方模块

### 11. 面向对象编程

OOP：Object Oriented Programming，是一种程序设计思想。

在 Python 中，所有数据类型都可以视为对象，当然也可以自定义对象。 自定义的对象数据类型就是面向对象中的类(Class)的概念。 

数据封装、继承和多态是面向对象的三大特点。 

#### 11.1 类和实例

类是抽象的模版，实例是根据类创建出来的一个个具体的“对象”。

Python中创建类的格式：

```python
class Student(object):		#括号中为继承的父类名
    
    name = '1234123'	#类属性，可被所有实例获取到
    
    def __init__(self, name, score):	#__init__方法的第一个参数永远是self，表示创建的实例本身。
        self.__name = name	    #__开头的变量名表示私有，外部无法直接访问
        self.__score = score 	#__开头的变量名表示私有，外部无法直接访问
        
    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
        
bart = Student('Bart Simpson', 59)
bart.print_score()

>>> def get_score(self, score):
...     return self.__score
...
>>> Student.set_score = MethodType(get_score, Student)	#给类动态添加get_score方法
```

和普通的函数相比，在类中定义的函数只有一点不同，就是第一个参数永远是实例变量 self，并且，调用时不用传递该参数。 由哪一个对象调用的方法，方法内的self就是哪一个对象的引用。

和静态语言不同，Python 允许对实例变量绑定任何数据，不必是类中定义的变量。也就是说，对于两个实例变量，虽然它们都是同一个类的不同实例，但拥有的变量名称都可能不同: 

```python
>>> bart = Student('Bart Simpson', 59)
>>> lisa = Student('Lisa Simpson', 87)
>>> bart.age = 8
>>> bart.age
>>> lisa.age
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'age'
```

> 以一个下划线开头的实例变量名，比如_name，这样的实例变量外部是可以访问的。当你看到这样的变量时，它的意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。 

> 双下划线开头的实例变量是不是一定不能从外部访问呢?其实也不是。 不能直接访问`__name `是因为 Python 解释器对外把`__name` 变量改成了` _Student__name`，所以，仍然可以通过`_Student__name` 来访问`__name` 变量。不同版本的Python解释器可能会把`__name`改成不同的变量名，所以请不要这样访问。

**Python本身没有任何机制阻止你干坏事，一切全靠自觉。**

`__方法名__`格式的方法是Python提供的内置方法/属性。

##### 11.1.1 内置方法

| 序号 |   方法名   | 类型 | 作用                                     |
| :--: | :--------: | :--: | :--------------------------------------- |
|  01  | `__new__`  | 方法 | 创建对象时，会被自动调用                 |
|  02  | `__init__` | 方法 | 对象被初始化时，会被自动调用             |
|  03  | `__del__`  | 方法 | 对象被从内存中销毁前，会被自动调用       |
|  04  | `__str__`  | 方法 | 返回对象的描述信息，`print` 函数输出使用 |

##### 11.1.2 私有属性和私有方法

在定义属性或方法时，在属性名或者方法名前增加两个下划线，定义的就是私有属性/方法。

在Python中，并没有真正意义的私有，在给属性、方法命名时，实际是对名称做了一些特殊处理，使得外界无法访问。

处理方式，在名称前面加上 `_类名`=> `_类名__名称`

##### 11.1.3 类属性和类方法

每个对象都有自己独立的内存空间，保存各自不同的属性。多个对象的方法，在内存中只有一份，在调用方法时，需要把对象的引用传递到方法内部。

类是一个特殊的对象，在Python中，一切皆对象。`class AAA`中定义的类属于类对象，`obj1 = AAA()`属于实例对象。

在程序运行时，类同样会被加载到内存，在程序运行时，类对象在内存中只有一份，使用一个类可以创建出很多个实例对象。除了封装实例的属性和方法外，类对象还可以拥有自己的属性和方法。

通过类名.的方式可以访问类的属性或者调用类的方法。

类属性就是针对类对象定义的属性，使用赋值语句在class关键字下方可以定义类属性。

类方法就是针对类对象定义的方法，在类方法内部可以直接访问类属性或者调用其他的类方法。

类方法需要由修饰器`@classmethod`来标识，告诉解释器这是一个类方法。

类方法的第一个参数应该是`cls`，由哪一个类调用的方法，方法内的`cls`就是哪一个类的引用。这个参数和实例方法的第一个参数是`self`类似。

使用其他名称也可以，不过习惯上使用`cls`

通过类名.调用类方法，调用方法时，不需要传递`cls`参数，在方法内部，可以通过`cls.`访问类的属性，也可以通过`cls.`调用其他的类方法。

```python
@classmethod
def 类方法名(cls):
	pass
```

##### 11.1.4 静态方法

在开发时，如果需要在类中封装一个方法，这个方法既不需要访问实例属性或者调用实例方法，也不需要访问类属性或者调用类方法。这个时候，可以把这个方法封装成一个静态方法。

```python
@staticmethod
def 静态方法名():
	pass
```

静态方法需要使用修饰器`@staticmethod`来标识，告诉解释器这是一个静态方法。

可以通过`类名.`调用静态方法。

#### 11.2 多态

对于静态语言(例如 Java)来说，如果需要传入父类类型，则传入的对象必须是父类类型或者它的子类，否则，将无法调用 run()方法。 

对于 Python 这样的动态语言来说，则不一定需要传入 Animal 类型。我们只需要保证传入的对象有一个 run()方法就可以了。这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。 

可能是因为方法传入参数无具体类型。

#### 11.3 获取对象信息

当我们拿到一个对象的引用时，如何知道这个对象是什么类型、有哪些方法呢？

##### 11.3.1 `type()`

判断对象类型，使用 type()函数，其返回对象对应的Class类型。

```python
>>> type(123)	#判断一个基本数据类型的类型
<class 'int'>
>>> type(abs)
<class 'builtin_function_or_method'>
>>> type(a)		#判断一个类的类型
<class '__main__.Animal'>
>>> type(123)==type(456)	#比较两个变量的type是否相同
True
>>> import types	# 导入types模块
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType	#判断是否为函数
True
```

##### 11.3.2 `isinstance()`

对于 class 的继承关系来说，使用 type()就很不方便。我们要判断 class 的类型，可以使用` isinstance()`函数。`isinstance()`判断的是一个对象是否是该类型本身，或者位于该类型的父继承链上。 

能用`type()`判断的基本类型也可以用`isinstance()`判断。

还可以判断一个变量是否是某些类型中的一种，比如下面的代码就可以判断是否是 list 或者 tuple: 

```python
>>> isinstance([1, 2, 3], (list, tuple))
True
>>> isinstance((1, 2, 3), (list, tuple))
True
```

总是优先使用`isinstance()`判断类型，可以将执行类型及其子类“一网打尽”。

##### 11.3.3 使用`dir()`

如果要获得一个对象的所有属性和方法，可以使用 dir()函数，它返回一个包含字符串的 list。

仅仅把属性和方法列出来是不够的，配合 `getattr()`、`setattr()`以及` hasattr()`，我们可以直接操作一个对象的状态。

```python
>>>hasattr(obj, 'x') # 有属性'x'吗? True
>>> obj.x
9 
>>> hasattr(obj, 'y') # 有属性'y'吗?
False
>>> setattr(obj, 'y', 19) # 设置一个属性'y' 
>>> hasattr(obj, 'y') #
True
>>> getattr(obj, 'y', [404]) #如果不存在属性y，可以设置一个默认值。这里是404
19
>>> obj.y # 获取属性'y' 19 
#可以使用同样的方法获取对象的方法。
```

下面是一个使用这些方法的例子：

```python
def readImage(fp):
    if hasattr(fp, 'read'):
        return readData(fp)
    return None
```

假设我们希望从文件流 fp 中读取图像，我们首先要判断该 fp 对象是否存在 read 方法，如果存在，则该对象是一个流，如果不存在，则无法读取。hasattr()就派上了用场。

要注意的是：只有在不知道对象信息的时候，我们才会去获取对象信息。 

#### 11.4 使用`__slots__`

我们可以在程序中动态添加实例或类的属性/方法。如果我们想要限制实例的属性怎么办？比如，只允许对 Student实例添加 name 和 age 属性。

为了达到限制的目的，Python 允许在定义 class 的时候，定义一个特殊的`__slots__`变量，来限制该 class 实例能添加的属性：

```python
class Student(object): 
	__slots__ = ('name', 'age') # 用 tuple 定义允许绑定的属性名称 
    
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name' >>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score' Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```

使用`__slots__`要注意，`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的。除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。 

#### 11.5 使用`@property`

在之前，如果为了保证Student中score数据的安全，我们设置了set/get方法来检查参数。但是这样比较复杂，我们可以通过内置的装饰器@property把一个方法变成属性调用。

```python
class Student(object):
    @property		#把score的调用方法变成属性
    def score(self):
        return self._score
        
    @score.setter	#该装饰器由property生成，实现为score属性设置值
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
        

>>> s = Student()
>>> s.score = 60 # OK，实际转化为 s.set_score(60) 
>>> s.score # OK，实际转化为 s.get_score()
60
```

@property 广泛应用在类的定义中，可以让调用者写出简短的代码，同时保证对参数进行必要的检查，这样，程序运行时就减少了出错的可能性。 

#### 11.6 继承

Python允许使用多重继承，

```Python
class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):	#多重继承
    pass
```

子类对象不能在自己的方法内部，直接访问父类的私有属性或私有方法，但可以通过父类的公有方法间接访问到私有属性或私有方法。

1. 私有属性、方法是对象的隐私，不对外公开，外界以及子类都不能直接访问
2. 私有属性、方法通常用于做一些内部的事情

##### 11.6.1 MinIn

在设计类的继承关系时，通常，主线都是单一继承下来的。但是，如果需要“混入”额外的功能，通过多重继承可以实现。这种设计通常称之为MinIn。

##### 11.6.2 MRO

Python中的MRO——方法搜索顺序

Python中针对类提供了一个内置属性`__mro__`可以查看方法搜索顺序。
MRO(method resolution order)，主要用于在多继承时判断方法、属性的调用路径。

```python
print(C.__mro__)
输出结果：
(<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
```

在搜索方法时，是按照`__mro__`的输出结果从左至右的顺序查找的，如果在当前类中找到方法，就直接执行，不在搜索。如果没有找到，就查找下一个类中是否有对应的方法，如果找到，就直接执行，不再搜索。如果找到最后一个类，还没有找到方法，程序报错。

实例：

```python
print("******多继承使用类名.__init__ 发生的状态******")
class Parent(object):
    def __init__(self, name):
        print('parent的init开始被调用')
        self.name = name
        print('parent的init结束被调用')

class Son1(Parent):
    def __init__(self, name, age):
        print('Son1的init开始被调用')
        self.age = age
        Parent.__init__(self, name)
        print('Son1的init结束被调用')

class Son2(Parent):
    def __init__(self, name, gender):
        print('Son2的init开始被调用')
        self.gender = gender
        Parent.__init__(self, name)
        print('Son2的init结束被调用')

class Grandson(Son1, Son2):
    def __init__(self, name, age, gender):
        print('Grandson的init开始被调用')
        Son1.__init__(self, name, age)  # 单独调用父类的初始化方法
        Son2.__init__(self, name, gender)
        print('Grandson的init结束被调用')

gs = Grandson('grandson', 12, '男')
print('姓名：', gs.name)
print('年龄：', gs.age)
print('性别：', gs.gender)

print("******多继承使用类名.__init__ 发生的状态******\n\n")
```

执行结果：

```python
******多继承使用类名.__init__ 发生的状态******
Grandson的init开始被调用
Son1的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son1的init结束被调用
Son2的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son2的init结束被调用
Grandson的init结束被调用
姓名： grandson
年龄： 12
性别： 男
******多继承使用类名.__init__ 发生的状态******
```

这种方法直接调用父类方法，根据调用语句执行被调用的方法。

```python
print("******多继承使用super().__init__ 发生的状态******")
class Parent(object):
    def __init__(self, name, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('parent的init开始被调用')
        self.name = name
        print('parent的init结束被调用')

class Son1(Parent):
    def __init__(self, name, age, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son1的init开始被调用')
        self.age = age
        super().__init__(name, *args, **kwargs)  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son1的init结束被调用')

class Son2(Parent):
    def __init__(self, name, gender, *args, **kwargs):  # 为避免多继承报错，使用不定长参数，接受参数
        print('Son2的init开始被调用')
        self.gender = gender
        super().__init__(name, *args, **kwargs)  # 为避免多继承报错，使用不定长参数，接受参数	
        # 此处的*args是解包，将传入参数原封不动的传出。
        print('Son2的init结束被调用')

class Grandson(Son1, Son2):
    def __init__(self, name, age, gender):
        print('Grandson的init开始被调用')
        # 多继承时，相对于使用类名.__init__方法，要把每个父类全部写一遍
        # 而super只用一句话，执行了全部父类的方法，这也是为何多继承需要全部传参的一个原因
        # super(Grandson, self).__init__(name, age, gender)  # 可以指定调用的父类，然后依照MRO顺序进行
        super().__init__(name, age, gender)
        print('Grandson的init结束被调用')

print(Grandson.__mro__)

gs = Grandson('grandson', 12, '男')
print('姓名：', gs.name)
print('年龄：', gs.age)
print('性别：', gs.gender)
print("******多继承使用super().__init__ 发生的状态******\n\n")
```

执行结果：

```
******多继承使用super().__init__ 发生的状态******
(<class '__main__.Grandson'>, <class '__main__.Son1'>, <class '__main__.Son2'>, <class '__main__.Parent'>, <class 'object'>)
Grandson的init开始被调用
Son1的init开始被调用
Son2的init开始被调用
parent的init开始被调用
parent的init结束被调用
Son2的init结束被调用
Son1的init结束被调用
Grandson的init结束被调用
姓名： grandson
年龄： 12
性别： 男
******多继承使用super().__init__ 发生的状态******
```

> 以上2个代码执行的结果不同
>
> 1. 如果2个子类中都继承了父类，当在子类中通过父类名调用时，parent被执行了2次
> 2. 如果2个子类中都继承了父类，当在子类中通过super调用时，parent被执行了1次

单继承中也是按照MRO的顺序来执行，但是单继承中该方式与直接调用类名无区别。

**总结：**

1. super().__init__`相对于`类名.__init`__，在单继承上用法基本无差
2. 但在多继承上有区别，super方法能保证每个父类的方法只会执行一次，而使用类名的方法会导致方法被执行多次，具体看前面的输出结果
3. 多继承时，使用super方法，对父类的传参数，应该是由于python中super的算法导致的原因，必须把参数全部传递，否则会报错
4. 单继承时，使用super方法，则不能全部传递，只能传父类方法所需的参数，否则会报错
5. 多继承时，相对于使用类名.__init__方法，要把每个父类全部写一遍, 而使用super方法，只需写一句话便执行了全部父类的方法，这也是为何多继承需要全部传参的一个原因

##### 11.6.3 父类变量的获取方式

```python
class Parent(object):
    x = 1

class Child1(Parent):
    pass

class Child2(Parent):
    pass

print(Parent.x, Child1.x, Child2.x)
Child1.x = 2
print(Parent.x, Child1.x, Child2.x)
Parent.x = 3
print(Parent.x, Child1.x, Child2.x)
```

答案：

```
class Parent(object):
    x = 1

class Child1(Parent):
    pass

class Child2(Parent):
    pass

print(Parent.x, Child1.x, Child2.x)
Child1.x = 2
print(Parent.x, Child1.x, Child2.x)
Parent.x = 3
print(Parent.x, Child1.x, Child2.x)
```

解析：

使你困惑或是惊奇的是关于最后一行的输出是 3 2 3 而不是 3 2 1。为什么改变了 Parent.x 的值还会改变 Child2.x 的值，但是同时 Child1.x 值却没有改变？

这个答案的关键是，在 Python 中，类变量在内部是作为字典处理的。如果一个变量的名字没有在当前类的字典中发现，将搜索祖先类（比如父类）直到被引用的变量名被找到（如果这个被引用的变量名既没有在自己所在的类又没有在祖先类中找到，会引发一个 AttributeError 异常 ）。

因此，在父类中设置 x = 1 会使得类变量 x 在引用该类和其任何子类中的值为 1。这就是因为第一个 print 语句的输出是 1 1 1。

随后，如果任何它的子类重写了该值（例如，我们执行语句 Child1.x = 2），然后，该值仅仅在子类中被改变。这就是为什么第二个 print 语句的输出是 1 2 1。

最后，如果该值在父类中被改变（例如，我们执行语句 Parent.x = 3），这个改变会影响到任何未重写该值的子类当中的值（在这个示例中被影响的子类是 Child2）。这就是为什么第三个 print 输出是 3 2 3。11.6.3 父变量的获取方式

#### 11.7 定制类

对于Python中特殊变量或函数名`__xxx___`都有一些特殊用途。

- `__len__`：让class可以作用于`len()`函数

- `__str__`：设置打印类时的显示文字，返回用户看到的字符串。

  ```python
  >>> print(Student('Michael'))
  <__main__.Student object at 0x109afb190>
  ```

  但是，此时，直接使用变量不会生效，这是因为直接使用变量用的是`__repr__()`，它用来返回程序开发者看到的字符串，是为调试服务的。

  ```python
  >>> s = Student('Michael')		
  >>> s
  <__main__.Student object at 0x109afb310>	#此时还需要设置__repr__()返回的值
  ####解决方法一般是让这两个函数相同
  __repr__ = __str__
  ```

- `__iter__`：如果一个类想被用于 for ... in 循环，类似 list 或 tuple 那样，就必须实现一个`__iter__()`方法，该方法返回一个迭代对象，然后，Python 的 for 循环就会不断调用该迭代对象的`__next__()`方法拿到循环的下一个值， 直到遇到 StopIteration 错误时退出循环。 

- `__getitem__`：要表现得像 list 那样按照下标取出元素，需要实现`__getitem__()`方法: 

  ```python
  class Fib(object):	#以斐波数为例
      def __getitem__(self, n):
  		if isinstance(n, int): # n 是索引 a, b = 1, 1
              for x in range(n):
                  a, b = b, a + b
  			return a
  		if isinstance(n, slice): # n 是切片
              start = n.start
              stop = n.stop
              if start is None:
                  start = 0
              a, b = 1, 1
              L = []
              for x in range(stop):
                  if x >= start:
                      L.append(a)
                  a, b = b, a + b
  			return L
  ```

- `__getattr__`：在Python在类中没有找到属性/方法的情况下，会调用`__getattr__`查找，已有的属性不会在`__getattr__`中查找。 

  ```python
  class Student(object):
      def __getattr__(self, attr):
          if attr=='age':
              return lambda: 25
  ```
  
- `__call__`：任何类，只需要定义一个`__call__()`方法，就可以直接对实例进行调用。 

  ```python
  class Student(object):
      def __init__(self, name):
          self.name = name
      def __call__(self):
          print('My name is %s.' % self.name)
          
  >>> s = Student('Michael') 
  >>> s() # self 参数不用传入 
  My name is Michael.
  
  ##如何判断一个对象是否能被调用，能调用的对象就是一个Callable对象。
  >>> callable(Student())
  True
  ```

#### 11.8 使用枚举类

当我们需要定义常量时，一个办法是用大写变量通过整数来定义。好处是简单，缺点是类型是 int，并且仍然是变量。 更好的方法是为这样的枚举类型定义一个 class 类型，然后，每个常量都是 class 的一个唯一实例。Python 提供了 Enum 类来实现这个功能: 

```python
from enum import Enum
Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
```

这样我们就获得了 Month 类型的枚举类，可以直接使用 Month.Jan 来引用 一个常量，或者枚举它的所有成员: 

```python
for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
```

value 属性则是自动赋给成员的 int 常量，默认从 1 开始计数。 

如果需要更精确地控制枚举类型，可以从 Enum 派生出自定义类：

```Python
from enum import Enum, unique 
@unique	#该装饰器可以帮助我们检查保证没有重复值
class Weekday(Enum):
    Sun = 0	# Sun的value被设定为0 
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```

Enum 可以把一组相关常量定义在一个 class 中，且 class 不可变，而且成员可以直接比较。 

#### 11.9 使用元类

##### 11.9.1 `type()`

动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。

比方说我们要定义一个 Hello 的 class，就写一个 hello.py 模块: 

```python
class Hello(object):
    def hello(self, name='world'):
        print('Hello, %s.' % name)
```

 当 Python 解释器载入 hello 模块时，就会依次执行该模块的所有语句， 执行结果就是动态创建出一个 Hello 的 class 对象，测试如下：

```python
>>> from hello import Hello 
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class 'hello.Hello'>
```

type()函数可以查看一个类型或变量的类型，Hello 是一个 class，它的类型就是 type，而 h 是一个实例，它的类型就是 class Hello。 

我们说 class 的定义是运行时动态创建的，而创建 class 的方法就是使用 type()函数。 

type()函数既可以返回一个对象的类型，又可以创建出新的类型，比如， 我们可以通过 type()函数创建出 Hello 类，而无需通过 class Hello(object)...的定义: 

```python
>>> def fn(self, name='world'): # 先定义函数
... print('Hello, %s.' % name)
...
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建 Hello class >>> h = Hello() 
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class '__main__.Hello'>
```

要创建一个 class 对象，type()函数依次传入 3 个参数：

1. class的名称; 
2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了 tuple 的单元素写法; 
3. class的方法名称与函数绑定，这里我们把函数fn绑定到方法名 hello 上。 

通过 type()函数创建的类和直接写 class 是完全一样的，因为 Python 解释器遇到 class 定义时，仅仅是扫描一下 class 定义的语法，然后调用 type() 函数创建出 class。 

正常情况下，我们都用 class Xxx...来定义类，但是，type()函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂。 

##### 11.9.2 `metaclass`

除了使用 type()动态创建类以外，要控制类的创建行为，还可以使用metaclass。 

metaclass，直译为元类，简单的解释就是: 

​	当我们定义了类以后，就可以根据这个类创建出实例，所以：先定义类， 然后创建实例。 

但是如果我们想创建出类呢?那就必须根据 metaclass 创建出类，所以: 

​	先定义 metaclass，然后创建类。

连接起来就是:先定义 metaclass，就可以创建类，最后创建实例。 

所以，metaclass 允许你创建类或者修改类。换句话说，你可以把类看成是 metaclass 创建出来的“实例”。 

metaclass 是 Python 面向对象里最难理解，也是最难使用的魔术代码。 正常情况下，你不会碰到需要使用 metaclass 的情况，所以，以下内容看不懂也没关系，因为基本上你不会用到。

我们先看一个简单的例子，这个 metaclass 可以给我们自定义的 MyList 增加一个 add 方法: 

定义 ListMetaclass，按照默认习惯，metaclass 的类名总是以 Metaclass 结尾，以便清楚地表示这是一个 metaclass: 

```python
# metaclass 是类的模板，所以必须从type类型派生:
class ListMetaclass(type): 
	def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)
```

有了 ListMetaclass，我们在定义类的时候还要指示使用 ListMetaclass 来定制类，传入关键字参数 metaclass：

```python
class MyList(list, metaclass=ListMetaclass):
    pass
```

当我们传入关键字参数 metaclass 时，魔术就生效了，它指示 Python 解 释器在创建 MyList 时，要通过 `ListMetaclass.__new__()`来创建，在此， 我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。 

`__new__()`方法接收到的参数依次是: 

1. 当前准备创建的类的对象; 
2. 类的名字;
3. 类继承的父类集合;
4. 类的方法集合。 

测试一下 MyList 是否可以调用 add()方法: 

```python 
>>> L = MyList()
>>> L.add(1)
>> L
[1]
```

而普通的 list 没有 add()方法: 

```python
>>> L2 = list()
>>> L2.add(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'list' object has no attribute 'add'
```

动态修改有什么意义？直接在 MyList 定义中写上 add()方法不是更简单吗?正常情况下，确实应该直接写，通过 metaclass 修改纯属变态。 

但是，总会遇到需要通过 metaclass 修改类定义的。ORM 就是一个典型的例子。 

ORM 全称“Object Relational Mapping”，即对象-关系映射，就是把关系数据库的一行映射为一个对象，也就是一个类对应一个表，这样，写代码更简单，不用直接操作 SQL 语句。  

要编写一个 ORM 框架，所有的类都只能动态定义，因为只有使用者才能根据表的结构定义出对应的类来。 

让我们来尝试编写一个 ORM 框架。 

编写底层模块的第一步，就是先把调用接口写出来。比如，使用者如果使用这个 ORM 框架，想定义一个 User 类来操作对应的数据库表 User， 我们期待他写出这样的代码: 

```python
class User(Model):
	# 定义类的属性到列的映射:
	id = IntegerField('id')
	name = StringField('username') 
	email = StringField('email') 
	password = StringField('password') 

# 创建一个实例:
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库:
u.save() 
```

其中，父类 Model 和属性类型 StringField、IntegerField 是由 ORM 框架 提供的，剩下的魔术方法比如 save()全部由 metaclass 自动完成。虽然 metaclass 的编写会比较复杂，但 ORM 的使用者用起来却异常简单。 

现在，我们就按上面的接口来实现该 ORM。 首先来定义 Field 类，它负责保存数据库表的字段名和字段类型：

```python
class Field(object):

	def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type
        
    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name) 
```

在 Field 的基础上，进一步定义各种类型的 Field，比如 StringField， IntegerField 等等：

```python
class StringField(Field):
    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')
        
class IntegerField(Field):
    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')
```

下一步，就是编写最复杂的 ModelMetaclass 了: 

```python
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model: %s' % name)
        mappings = dict()
        for k, v in attrs.items():
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        for k in mappings.keys():
            attrs.pop(k)
		attrs['__mappings__'] = mappings # 保存属性和列的映射关系 
        attrs['__table__'] = name # 假设表名和类名一致
		return type.__new__(cls, name, bases, attrs)
```

以及基类Model：

```python
class Model(dict, metaclass=ModelMetaclass):
	  def __init__(self, **kw):
        super(Model, self).__init__(**kw)
        
    def __getattr__(self, key):
        try:    
        	return self[key]
        except KeyError:
        	raise AttributeError(r"'Model' object has no attribute
'%s'" % key)     

	  def __setattr__(self, key, value):
        self[key] = value
        
    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.items():
        	fields.append(v.name)
            params.append('?')
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__,
','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))                  

```

当用户定义一个 class User(Model)时，Python 解释器首先在当前类 User 的定义中查找 metaclass，如果没有找到，就继续在父类 Model 中查找 metaclass，找到了，就使用 Model 中定义的 metaclass 的 ModelMetaclass 来创建 User 类，也就是说，metaclass 可以隐式地继承到子类，但子类自己却感觉不到。 

在 ModelMetaclass 中，一共做了几件事情: 

1. 排除掉对Model类的修改；
2. 在当前类(比如User)中查找定义的类的所有属性，如果找到一个 Field 属性，就把它保存到`__mappings__`的 dict 中，同时从类属性中删除该 Field 属性，否则，容易造成运行时错误(实例的属性会遮盖类的同名属性)；
3. 把表名保存到`__table__`中，这里简化为表名默认为类名。 

在 Model 类中，就可以定义各种操作数据库的方法，比如 save()， delete()，find()，update 等等。 

我们实现了 save()方法，把一个实例保存到数据库中。因为有表名，属性到字段的映射和属性值的集合，就可以构造出 INSERT 语句。 

编写代码试试: 

```python
u = User(id=12345, name='Michael', email='test@orm.org',
password='my-pwd')
u.save()
```

输出如下: 

```python 
Found model: User
Found mapping: email ==> <StringField:email>
Found mapping: password ==> <StringField:password>
Found mapping: id ==> <IntegerField:uid>
Found mapping: name ==> <StringField:username>
SQL: insert into User (password,email,username,id) values (?,?,?,?)
ARGS: ['my-pwd', 'test@orm.org', 'Michael', 12345]
```

可以看到，save()方法已经打印出了可执行的 SQL 语句，以及参数列表，只需要真正连接到数据库，执行该 SQL 语句，就可以完成真正的功能。 

不到 100 行代码，我们就通过 metaclass 实现了一个精简的 ORM 框架。

metaclass 是 Python 中非常具有魔术性的对象，它可以改变类创建时的行为。这种强大的功能使用起来务必小心。 

#### 11.10 类的初始化

当使用类名() 创建对象时，会自动执行以下操作：  

1. 为对象在内存中分配空间 —— 创建对象
2. 为对象的属性设置初始值—— 初始化方法(`__init__`)



### 12. 错误、调试和测试

在程序运行过程中，总会遇到各种各样的错误。

有的错误是程序编写有问题造成的，比如本来应该输出整数结果输出了字符串，这种错误我们通常称之为 bug，bug 是必须修复的。 

有的错误是用户输入造成的，比如让用户输入 email 地址，结果得到一个空字符串，这种错误可以通过检查用户输入来做相应的处理。

还有一类错误是完全无法在程序运行过程中预测的，比如写入文件的时候，磁盘满了，写不进去了，或者从网络抓取数据，网络突然断掉了。这类错误也称为异常，在程序中通常是必须处理的，否则，程序会因为各种问题终止并退出。 

Python 内置了一套异常处理机制，来帮助我们进行错误处理。 此外，我们也需要跟踪程序的执行，查看变量的值是否正确，这个过程称为调试。Python 的 pdb 可以让我们以单步方式执行代码。 

最后，编写测试也很重要。有了良好的测试，就可以在程序修改后反复运行，确保程序输出符合我们编写的测试。

#### 12.1 错误处理

在程序运行的过程中，如果发生了错误，可以事先约定返回一个错误代码，这样，就可以知道是否有错，以及出错的原因。在操作系统提供的调用中，返回错误码非常常见。比如打开文件的函数 open()，成功时返回文件描述符(就是一个整数)，出错时返回-1。 

用错误码来表示是否出错十分不便，因为函数本身应该返回的正常结果和错误码混在一起，造成调用者必须用大量的代码来判断是否出错：

```python
def foo():
	r = some_function()
    if r==(-1):
        return (-1)
    # do something
	return r
	
def bar():
    r = foo()
    if r==(-1):
        print('Error')
	else: pass
```

一旦出错，还要一级一级上报，直到某个函数可以处理该错误(比如，给用户输出一个错误信息)。所以高级语言通常都内置了一套 try...except...finally...的错误处理机制，Python 也不例外。 

```python
try:
    print('try...')
    r = 10 / int('2')
    print('result:', r)
except ValueError as e:
    print('ValueError:', e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
else:			#如果没有发生错误，自动执行else语句
    print('no error!')
finally:  # 一定会被调用
    print('finally...')
print('END')
```

Python 的错误其实也是 class，所有的错误类型都继承自 BaseException。

##### 12.1.1 记录错误

如果不捕获错误，自然可以让 Python 解释器来打印出错误堆栈，但程序也被结束了。既然我们能捕获错误，就可以把错误堆栈打印出来，然后分析错误原因，同时，让程序继续执行下去。 

Python 内置的 logging 模块可以非常容易地记录错误信息: 

```python
# err_logging.py
import logging
def foo(s):
    return 10 / int(s)
def bar(s):
    return foo(s) * 2
def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)
main()
print('END')
```

同样是出错，但程序打印完错误信息后会继续执行，并正常退出：

```python
$ python3 err_logging.py
ERROR:root:division by zero
Traceback (most recent call last):
  File "err_logging.py", line 13, in main
    bar('0')
  File "err_logging.py", line 9, in bar
    return foo(s) * 2
  File "err_logging.py", line 6, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
END
```

通过配置，logging 还可以把错误记录到日志文件里，方便事后排查。 

##### 12.1.2 抛出错误

因为错误是 class，捕获一个错误就是捕获到该 class 的一个实例。因此， 错误并不是凭空产生的，而是有意创建并抛出的。Python 的内置函数会抛出很多类型的错误，我们自己编写的函数也可以抛出错误。 

如果要抛出错误，首先根据需要，可以定义一个错误的 class，选择好继承关系，然后，用 raise 语句抛出一个错误的实例：

```python
# err_raise.py
class FooError(ValueError):
	pass 
	
def foo(s):
    n = int(s)
    if n==0:
        raise FooError('invalid value: %s' % s)	#错误抛出后，就可以被except捕捉到
    return 10 / n
foo('0')
```

raise 语句如果不带参数，就会把当前错误原样抛出。 

只有在必要的时候才定义我们自己的错误类型。如果可以选择Python已有的内置的错误类型，尽量使用Python内置的错误类型。

#### 12.2 调试

当程序有BUG且较为复杂时，我们需要知道出错时，哪些变量的值时正确的，哪些变量的值是错误的。因此，需要一整套调试程序的手段来修复BUG。有以下几种方法：

1. `print()`打印出信息；

   该方法使用后需要大量删除，运行结果有很多垃圾信息。

2. 断言：凡是用`print()`来辅助查看的地方，都可以用断言（assert）来替代：

   ```python
   def foo(s):
   		n = int(s)
   		assert n != 0, 'n is zero!'
   		return 10 / n
   
   def main():
   		foo('0')
   ```

   assert的意思是：表达式 n != 0应该是True，否则，根据程序运行的逻辑，后面的代码肯定会出错。

   如果断言失败，assert语句本身就会抛出AssertionError。。

   程序运行时可以用 -o 参数来关闭assert。`python -o xxx.py`

3. logging

   ```python
   import logging
   logging.basicConfig(level=logging.INFO)	#指定记录信息的级别（debug、info、warning、error）
   
   n = int(s)
   logging.info('n = %d' % n)		#类似Android的日志
   ```

4. pdb调试器

   pdb调试器可以让程序以单步方式运行，可以随时查看运行状态。

   ```python
   # err.py
   s = '0'
   n = int(s)
   print(10 / n)
   ```

   然后启动: 

   ```python 
   $ python3 -m pdb err.py
   >
   /Users/michael/Github/learn-python3/samples/debug/err.py(2)<module>() 
   -> s = '0'
   ```

   以参数-m pdb 启动后，pdb 定位到下一步要执行的代码-> s = '0'。

   - 输入命令l来查看代码；
   - 输入命令 n 可以单步执行代码；
   - 任何时候都可以输入命令 p 变量名来查看变量；`p s`
   - 输入命令 q 结束调试，退出程序。

   但是这种方法及其麻烦，如果程序较多，完全不适用。

5. pdb.set_trace()

   这个方法也是用 pdb，但是不需要单步执行，我们只需要 import pdb， 然后，在可能出错的地方放一个 pdb.set_trace()，就可以设置一个断点: 

   ```python 
   # err.py
   import pdb
   
   s = '0'
   n = int(s)
   pdb.set_trace() # 运行到这里会自动暂停
   print(10 / n) 
   ```

   运行代码，程序会自动在 pdb.set_trace()暂停并进入 pdb 调试环境，可以用命令 p 查看变量，或者用命令 c 继续运行。

   但是，相对来说也不是很方便。。。

6. IDE调试

   如果要比较爽地设置断点、单步执行，就需要一个支持调试功能的IDE。

   虽然IDE调试起来比较方便，但是最后你会发现，logging才是终极武器。

#### 12.3 单元测试

如果你听说过“测试驱动开发"(TDD:Test-Driven Development)，单元测试就不陌生。 单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。 这种以测试为驱动的开发模式最大的好处就是确保一个程序模块的行为符合我们设计的测试用例。在将来修改的时候，可以极大程度地保证该模块行为仍然是正确的。

我们来编写一个 Dict 类，这个类的行为和 dict 一致，但是可以通过属性来访问，用起来就像下面这样: 

```python
>>> d = Dict(a=1, b=2)
>>> d['a']
1
>>> d.a 1
```

mydict.py 代码如下：

```python
class Dict(dict):
    def init(self, **kw):
    	super().init(**kw)
    
    def getattr(self, key):
      try:
        return self[key]
      except KeyError:
        raise AttributeError(r"'Dict' object has no attribute'%s'" % key)

    def setattr(self, key, value):
		self[key] = value
```

为了编写单元测试，我们需要引入 Python 自带的 unittest 模块，编写 mydict_test.py 如下：

```python
import unittest
from mydict import Dict

class TestDict(unittest.TestCase):	#测试类需要从unittest.TestCase继承，以test开头的方法就是测试方法，不以test开头的方法不被认为是测试方法，测试的时候不会执行。
    
    #对每一类测试都需要编写一个 test_xxx()方法。由于 unittest.TestCase 提供了很多内置的条件判断，我们只需要调用这些方法就可以断言输出是否是我们所期望的。最常用的断言就是 assertEqual()
    def test_init(self):
        d = Dict(a=1, b='test')
        self.assertEqual(d.a, 1)  #断言相等
        self.assertEqual(d.b, 'test')
        self.assertTrue(isinstance(d, dict))	
    def test_key(self):
        d = Dict()
        d['key'] = 'value'
        self.assertEqual(d.key, 'value')
    def test_attr(self):
        d = Dict()
        d.key = 'value'
        self.assertTrue('key' in d)
        self.assertEqual(d['key'], 'value')
    def test_keyerror(self):
        d = Dict()
        with self.assertRaises(KeyError):
            value = d['empty']	#另一种重要的断言就是期待抛出指定类型的 Error，比如通过 d['empty']访问不存在的 key 时，断言会抛出 KeyError
    def test_attrerror(self):
        d = Dict()
        with self.assertRaises(AttributeError):
            value = d.empty		#而通过 d.empty 访问不存在的 key 时，我们期待抛出AttributeError
```

##### 12.3.1 运行单元测试

一旦编写好单元测试，我们就可以运行单元测试。最简单的运行方式是在 mydict_test.py 的最后加上两行代码: 

```python
if __name__ == '__main__':
    unittest.main()
```

这样就可以把 mydict_test.py 当做正常的 python 脚本运行:

```
$ python3 mydict_test.py
```

 另一种方法是在命令行通过参数-m unittest 直接运行单元测试: 

```
$ python3 -m unittest mydict_test
.....
---------------------------------------------------------------------
-
Ran 5 tests in 0.000s
OK
```

这是推荐的做法，因为这样可以一次批量运行很多单元测试，并且，有很多工具可以自动来运行这些单元测试。 

##### 12.3.2 setUp与tearDown

可以在单元测试中编写两个特殊的 setUp()和 tearDown()方法。这两个方法会分别在每调用一个测试方法的前后分别被执行。

setUp()和 tearDown()方法有什么用呢?设想你的测试需要启动一个数据库，这时，就可以在 setUp()方法中连接数据库，在 tearDown()方法中关闭数据库，这样，不必在每个测试方法中重复相同的代码。

**小结**

单元测试可以有效地测试某个程序模块的行为，是未来重构代码的信心保证；

单元测试的测试用例要覆盖常用的输入组合、边界条件和异常。

单元测试代码要非常简单，如果测试代码太复杂，那么测试代码本身就可能有Bug。

单元测试通过了并不意味着程序就没有Bug了，但是不通过程序肯定有Bug。

#### 12.4 文档测试

Python 内置的“文档测试”(doctest)模块可以直接提取注释中的代码并执行测试。 

doctest 严格按照 Python 交互式命令行的输入和输出来判断测试结果是否正确。 

```python
def fact(n):
    '''  # 文档测试模块可以直接提取注释中的代码并执行测试
    Function to get n!
    Example:
    >>> fact(1)	#中间有个空格
    1
    >>> fact(2)
    2
    >>> fact(3)
    6
    >>> fact('a')
    Traceback(most recent call last)
        ...
    KeyError: 'a'
    '''
    if n < 1 :
        raise ValueError()
    if n == 1 :
        return 1
    return n * fact(n - 1)

if __name__ == 'main' :	#正常导入时，不会被运行，只有在命令行直接运行时，才执行doctest。
    import doctest
    doctest.testmod()
print(fact(5))
```

doctest 非常有用，不但可以用来测试，还可以直接作为示例代码。通过某些文档生成工具，就可以自动把包含 doctest 的注释提取出来。用户看文档的时候，同时也看到了 doctest。 

当模块正常导入时，doctest不会被执行。只有在命令后直接运行时，才执行doctest。所以，不必担心doctest会在非测试环境下执行。

### 13. IO编程

IO 在计算机中指 Input/Output，也就是输入和输出。由于程序和运行时数据是在内存中驻留，由 CPU 这个超快的计算核心来执行，涉及到数据交换的地方，通常是磁盘、网络等，就需要 IO 接口。 

同步和异步的区别就在于是否等待 IO 执行的结果。异步IO需要进行完成任务后通知。操作I/O的能力都是由操作系统提供的，每一种编程语言都会把操作系统提供的低级C接口封装起来方便使用。

#### 13.1 文件读写

在磁盘上读写文件的功能都是由操作系统提供的，现在操作系统不允许普通的程序直接操作磁盘。所以，读写文件就是请求操作系统打开一个文件对象（通常称为文件描述符），然后，通过操作系统提供的接口从这个文件对象中读写文件。

##### 13.1.1 读文件

要以读文件的模式打开一个文件对象，使用 Python 内置的 open()函数，传入文件名和标示符： 

```python
>>> f = open('/Users/michael/test.txt', 'r')	#如果不存在，会抛出IOError错误
>>> f.read()  #文件打开成功后，调用read()方法可以一次性读取文件的全部内容，Python 把内容读到内存，用一个 str 对象表示
>>> f.close() #关闭文件，释放资源
```

由于文件读写时都有可能产生 IOError，一旦出错，后面的 f.close()就不会调用。所以，为了保证无论是否出错都能正确地关闭文件，我们可 以使用 try ... finally 来实现：

```python
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
    	f.close() 
```

但是每次都这么写实在太繁琐，所以，Python 引入了 with 语句来自动帮我们调用 close()方法: 

```python
with open('/path/to/file', 'r') as f:
    print(f.read())
```

这和前面的 try ... finally 是一样的，但是代码更佳简洁，并且不必调用 `f.close()`方法。 

调用 read()会一次性读取文件的全部内容，如果文件有 10G，内存就爆 ，所以，要保险起见，可以反复调用 read(size)方法，每次最多读取 size 个字节的内容。另外，调用 `readline()`可以每次读取一行内容，调用 `readlines()`一次读取所有内容并按行返回 list。因此，要根据需要决定怎么调用。 

如果文件很小，read()一次性读取最方便;如果不能确定文件大小，反复调用 read(size)比较保险;如果是配置文件，调用 `readlines()`最方便: 

```python
for line in f.readlines(): 
	print(line.strip()) # 把末尾的'\n'删掉 
```

**file-like Object**

像 open()函数返回的这种有个 read()方法的对象，在 Python 中统称为 file-like Object。 

除了 file 外，还可以是内存的字节流，网络流，自定义流等等。file-like Object 不要求从特定类继承，只要写个 read()方法就行。 

StringIO 就是在内存中创建的 file-like Object，常用作临时缓冲。 

**二进制文件**

要读取二进制文件，比如图片、视频等等，用'rb'模式打开文件即可。

**字符编码**

要读取非 UTF-8 编码的文本文件，需要给 open()函数传入 encoding 参数。

例如，读取 GBK 编码的文件: 

```python
>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk') 
>>> f.read()
'测试' 
```

遇到有些编码不规范的文件，你可能会遇到 UnicodeDecodeError，因为在文本文件中可能夹杂了一些非法编码的字符。遇到这种情况，open()函数还接收一个 errors 参数，表示如果遇到编码错误后如何处理。最简单的方式是直接忽略: 

```python
>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk', errors='ignore')
```

##### 13.1.2 写文件

写文件和读文件是一样的，唯一区别是调用 open()函数时，传入标识符'w'或者'wb'表示写文本文件/二进制文件。如果使用'wa'，则表示追加写入。

你可以反复调用 write()来写入文件，但是务必要调用 f.close()来关闭文件。当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用 close()方法时， 操作系统才保证把没有写入的数据全部写入磁盘。忘记调用 close()的 后果是数据可能只写了一部分到磁盘，剩下的丢失了。所以，还是用 with 语句来得保险: 

```python
with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
```

要写入特定编码的文本文件，请给 open()函数传入 encoding 参数，将字符串自动转换成指定编码。 

##### 13.1.3 总结

在Python中，文件读写都是通过`open()`函数打开的文件对象完成的，使用With语句操作文件IO是个好习惯。

#### 13.2 StringIO和BytesIO

##### 13.2.1 StringIO

很多时候，数据读写不一定是文件，也可以在内存中读写。StringIO 顾名思义就是在内存中读写 str。 

要把 str 写入 StringIO，我们需要先创建一个 StringIO，然后，像文件一样写入即可：

```python
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue())		#getValue()用于获取写入后的 str
hello world!
```

要读取 StringIO，可以用一个 str 初始化 StringIO，然后，像读文件一样读取：

```python
>>> from io import StringIO
>>> f = StringIO('Hello!\nHi!\nGoodbye!')
>>> while True:
...     s = f.readline()
... if s == '':
... break
...     print(s.strip())
...
Hello!
Hi!
Goodbye!
```

##### 13.2.2 BytesIO

StringIO 操作的只能是 str，如果要操作二进制数据，就需要使用 BytesIO。 BytesIO 实现了在内存中读写 bytes，我们创建一个 BytesIO，然后写入 一些 bytes: 

```python
>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中文'.encode('utf-8')) 
6
>>> print(f.getvalue()) 
b'\xe4\xb8\xad\xe6\x96\x87'
```

请注意，写入的不是 str，而是经过 UTF-8 编码的 bytes。  

和 StringIO 类似，可以用一个 bytes 初始化 BytesIO，然后，像读文件一样读取: 

```python
>>> from io import StringIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'
```

#### 13.3 操作文件和目录

如果我们要操作文件、目录，可以在命令行下面输入操作系统提供的各种命令来完成。比如 dir、cp 等命令。 

如果要在 Python 程序中执行这些目录和文件的操作怎么办?其实操作系统提供的命令只是简单地调用了操作系统提供的接口函数，Python 内置的 os 模块也可以直接调用操作系统提供的接口函数。 

所以，**os模块的默写函数是跟操作系统相关的**。

```python
>>> import os
>>> os.name # 获取操作系统类型，值是posix代表Linux、Unix或Mac OS X，如果是nt，就是Windows系统。
>>> os.uname()	# 获取详细的系统信息，该函数在Windows上不提供。
>>> os.environ	# 获取系统中定义的环境变量
>>> os.environ.get('PATH')	# 获取某个环境变量的值
```

另外，操作文件和目录的函数一部分放在 os 模块中，一部分放在 os.path 模块中。要特别注意。

```python
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/michael'

>>> os.path.join('/Users/michael', 'testdir') # 把两个路径合成一个时，不要直接拼字符串，而要通过join()函数，这样可以正确处理不同操作系统的路径分隔符。
'/Users/michael/testdir'
>>> os.mkdir('/Users/michael/testdir') 		# 然后创建一个目录。
>>> os.rmdir('/Users/michael/testdir')		# 删掉一个目录。

>>> os.path.split('/Users/michael/testdir/file.txt')	# 拆分路径为两部分，后一部分总是最后级别的目录或文件名。
('/Users/michael/testdir', 'file.txt')

>>> os.path.splitext('/path/to/file.txt')	#直接得到文件扩展名
('/path/to/file', '.txt')
#这些合并、拆分路径的函数并不要求目录和文件要真实存在，它们只对字符串进行操作。
```

文件操作：

```python
#文件操作使用下面的函数
>>> os.rename('test.txt', 'test.py')    # 对文件重命名
>>> os.remove('test.py')				# 删掉文件
>>> os.walk()  #用于通过在目录树中游走输出在目录中的文件名，向上或者向下。
```

但是，你会发现。复制文件的函数在os模块中不存在，原因是复制文件并非由操作系统系统的系统调用。幸运的是 shutil 模块提供了 copyfile() 的函数，你还可以在 shutil 模块中找到很多实用函数，它们可以看做是 os 模块的补充。 

#### 13.4 序列化

在程序运行的过程中，所有的变量都是在内存中，比如，定义一个 dict：

```python
d = dict(name='Bob', age=20, score=88) 
```

可以随时修改变量，比如把 name 改成'Bill'，但是一旦程序结束，变量所占用的内存就被操作系统全部回收。如果没有把修改后的'Bill'存储 到磁盘上，下次重新运行程序，变量又被初始化为'Bob'。 

我们把变量从内存中变成可存储或传输的过程称之为序列化，在 Python 中叫 pickling，在其他语言中也被称之为 serialization，marshalling， flattening 等等，都是一个意思。 

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化， 即 unpickling。 

Python 提供了 pickle 模块来实现序列化。 

首先，我们尝试把一个对象序列化并写入文件: 

```python
>>> import pickle
>>> d = dict(name='Bob', age=20, score=88)
>>> pickle.dumps(d)
b'\x80\x03}q\x00(X\x03\x00\x00\x00ageq\x01K\x14X\x05\x00\x00\x00scoreq\x02KXX\x04\x00\x00\x00nameq\x03X\x03\x00\x00\x00Bobq\x04u.'
```

`pickle.dumps()`方法把任意对象序列化成一个 bytes，然后，就可以把这 个 bytes 写入文件。或者用另一个方法 `pickle.dump()`直接把对象序列化后写入一个 file-like Object：

```python
>>> f = open('dump.txt', 'wb')
>>> pickle.dump(d, f)
>>> f.close()
```

当我们要把对象从磁盘读到内存时，可以先把内容读到一个 bytes，然后用 pickle.loads()方法反序列化出对象，也可以直接用 pickle.load() 方法从一个 file-like Object 中直接反序列化出对象。我们打开另一个 Python 命令行来反序列化刚才保存的对象: 

```python
>>> f = open('dump.txt', 'rb')
>>> d = pickle.load(f)
>>> f.close()
>>> d
{'age': 20, 'score': 88, 'name': 'Bob'} 
```

**反序列化之后的变量与原来的变量完全不相干，它们只是内容相同而已。**

Pickle 的问题和所有其他编程语言特有的序列化问题一样，就是它只能用于 Python，并且可能不同版本的Python 彼此都不兼容，因此，只能用 Pickle 保存那些不重要的数据，不能成功地反序列化也没关系。 

##### 13.4.1 JSON

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。JSON不仅是标准格式，并且比XML更快，而且可以直接在Web页面中读取，非常方便。

JSON表示的对象就是标准的JavaScript语言的对象，JSON 和 Python 内置的数据类型对应如下: 

| JSON 类型  | Python类型 |
| ---------- | ---------- |
| {}         | dict       |
| []         | list       |
| "string"   | str        |
| 1234.56    | int或float |
| true/false | True/False |
| null       | None       |

Python 对象变成一个 JSON: 

```python
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)	#该方法返回一个str，内容就是标准的JSON。类似的，dump()方法可以直接把JSON写入一个file-like Object
'{"age": 20, "score": 88, "name": "Bob"}'
#要把JSON反序列化为Python对象，用loads()或者对应的load()方法，前者把JSON的字符串反序列化，后者从 file-like Object中读取字符串并反序列化:
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

由于 JSON 标准规定 JSON 编码是 UTF-8，所以我们总是能正确地在 Python 的 str 与 JSON 的字符串之间转换。 

##### 13.4.2 JSON进阶

Python的dict对象可以直接序列化为JSON的{}，不过，很多时候，我们更喜欢用class表示对象，然后序列化。

class必须是可序列化的对象才能被序列化。

默认情况下，`dumps()`方法不知道如何将类实例变为一个JSON的对象。我们只需要为类专门写一个转换函数，再把函数传进去即可：

```python
import json
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }
>>> print(json.dumps(s, default=student2dict))
{"age": 20, "name": "Bob", "score": 88}

#反序列化
def dict2student(d):
	return Student(d['name'], d['age'], d['score']) 

>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> print(json.loads(json_str, object_hook=dict2student))
<__main__.Student object at 0x10cd3c190>  # 打印出的就是Student对象
```

小细节：

对于一些类，我们可以偷个懒，把任意的class的实例变为dict，然后进行然后进行转变JSON。

```python
print(json.dumps(s, default=lambda obj: obj.__dict__))
```

#### 13.5 总结

Python 语言特定的序列化模块是 pickle，但如果要把序列化搞得更通用、 更符合 Web 标准，就可以使用 json 模块。 

### 14. 进程和线程

并发：指的是任务数多余cpu核数，通过操作系统的各种任务调度算法，实现用多个任务“一起”执行（实际上总有一些任务不在执行，因为切换任务的速度相当快，看上去一起执行而已）。

并行：指的是任务数小于等于cpu核数，即任务真的是一起执行的。

多任务的实现有3种方式：

1. 多进程模式；
2. 多线程模式；
3. 多进程+多线程模式。

#### 14.1 多进程 

Unix/Linux 操作系统提供了一个 `fork()` 系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是 `fork()`调用一次，返回两次，因为操作系统自动把当前进程(称为父进程)复制了一份(称为子进程)， 然后，分别在父进程和子进程内返回。 

子进程永远返回 0，而父进程返回子进程的 ID。这样做的理由是，一个 父进程可以 fork 出很多子进程，所以，父进程要记下每个子进程的 ID， 而子进程只需要调用 `getppid()`就可以拿到父进程的 ID。 

Python 的 os 模块封装了常见的系统调用，其中就包括 fork，可以在 Python 程序中轻松创建子进程: 

```python
import os
print('Process (%s) start...' % os.getpid())
# Only works on Unix/Linux/Mac:
pid = os.fork()
if pid == 0:
    print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
else:
    print('I (%s) just created a child process (%s).' % (os.getpid(),pid))
```

运行结果如下: 

```python
Process (876) start...
I (876) just created a child process (877).
I am child process (877) and my parent is 876.
```

Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

##### 14.1.1 multiprocessing

但是使用`fork()`只能在Linux/Unix创建多进程。如果要跨平台，需要使用multiprocessing模块。

multiprocessing模块提供了一个Process类来代表一个进程对象，下面演示了启动一个子进程并等待其结束：

```python
from multiprocessing import Process
import os
# 子进程要执行的代码 
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))
    
if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))	#target=运行的目标函数，args=函数的参数
    print('Child process will start.')
    p.start()		# 开始子进程的执行
    p.join()		# 等待子进程结束后再继续往下运行，通常用于进程间的同步
		print('Child process end.')

# 执行结果如下
Parent process 928.
Process will start.
Run child process test (929)...
Process end.
```

**`Process`语法结构**：

`Process([group [, target [, name [, args [, kwargs]]]]])`

1. target：如果传递了函数的引用，可以认为这个子进程就执行这里的代码
2. args：给target指定的函数传递的参数，以元组的方式传递
3. kwargs：给target指定的函数传递命名参数
4. name：给进程设定一个名字，可以不设定
5. group：指定进程组，大多数情况下用不到

Process创建的实例对象的常用方法：

- start()：启动子进程实例（创建子进程）
- is_alive()：判断进程子进程是否还在活着
- join([timeout])：是否等待子进程执行结束，或等待多少秒
- terminate()：不管任务是否完成，立即终止子进程

Process创建的实例对象的常用属性：

- name：当前进程的别名，默认为Process-N，N为从1开始递增的整数
- pid：当前进程的pid（进程号）

##### 14.1.2 Pool

如果要启动大量的子进程，可以用进程池的方式批量创建子进程。

```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))
    
if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)		# 限制同时执行的进程数，默认大小是CPU的核数
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()		# 等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后不能继续添加新的Process。
    print('All subprocesses done.')
    
# 执行结果如下
Parent process 669.
Waiting for all subprocesses done...
Run task 0 (671)...
Run task 1 (672)...
Run task 2 (673)...
Run task 3 (674)...
Task 2 runs 0.14 seconds.
Run task 4 (673)...
Task 1 runs 0.27 seconds.
Task 3 runs 0.86 seconds.
Task 0 runs 1.41 seconds.
Task 4 runs 1.91 seconds.
All subprocesses done.
```

Pool()的默认大小是CPU的核数，所以，如果你不幸拥有8核CPU，那要提交至少9个子进程才能看到上面的等待效果。

**`multiprocessing.Pool`常用函数解析：**

1. `apply_async(func[, args[, kwds]])` ：使用非阻塞方式调用func（并行执行，堵塞方式必须等待上一个进程退出才能执行下一个进程），args为传递给func的参数列表，kwds为传递给func的关键字参数列表；
2. `close()`：关闭Pool，使其不再接受新的任务；
   1. `terminate()`：不管任务是否完成，立即终止；
3. `join()`：主进程阻塞，等待子进程的退出， 必须在close或terminate之后使用；

##### 14.1.3 子进程

很多时候，子进程并不是自身，而是一个外部进程。我们创建了子进程后，还需要控制子进程的输入和输出。 

subprocess 模块可以让我们非常方便地启动一个子进程，然后控制其输入和输出。 

下面的例子演示了如何在 Python 代码中运行命令 nslookup  www.python.org，这和命令行直接运行的效果是一样的：

```python
import subprocess

print('$ nslookup www.python.org')
r = subprocess.call(['nslookup', 'www.python.org'])
print('Exit code:', r)

#运行结果:
$ nslookup www.python.org
Server:        192.168.19.4
Address:    192.168.19.4#53
Non-authoritative answer:
www.python.org    canonical name = python.map.fastly.net.
Name:    python.map.fastly.net
Address: 199.27.79.223
Exit code: 0
```

如果子进程还需要输入，则可以通过 communicate()方法输入：

```python
import subprocess

print('$ nslookup')
p = subprocess.Popen(['nslookup'], stdin=subprocess.PIPE, stdout=subprocess.PIPE,  stderr=subprocess.PIPE)
output, err = p.communicate(b'set q=mx\npython.org\nexit\n')
print(output.decode('utf-8'))
print('Exit code:', p.returncode)
```

上面的代码相当于在命令行执行命令 nslookup，然后手动输入：

```python
set q=mx
python.org
exit
#运行结果如下：

$ nslookup
Server:        192.168.19.4
Address:    192.168.19.4#53
Non-authoritative answer:
python.org    mail exchanger = 50 mail.python.org.
Authoritative answers can be found from:
mail.python.org    internet address = 82.94.164.166
mail.python.org    has AAAA address 2001:888:2000:d::a6
Exit code: 0
```

##### 14.1.4 进程间通信

Process 之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python 的 multiprocessing 模块包装了底层的机制，提供了 Queue、Pipes 等多种方式来交换数据。 

我们以 Queue 为例，在父进程中创建两个子进程，一个往 Queue 里写数据，一个从 Queue 里读数据：

```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码：
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码：
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)
        
if __name__=='__main__':
    # 父进程创建 Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程 pw，写入:
    pw.start()
    # 启动子进程 pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr 进程里是死循环，无法等待其结束，只能强行终止
    pr.terminate()
    
# 运行结果
Process to write: 50563
Put A to queue...
Process to read: 50564
Get A from queue.
Put B to queue...
Get B from queue.
Put C to queue...
Get C from queue.
```

在 Unix/Linux 下，multiprocessing 模块封装了 `fork()` 调用，使我们不需要关注 fork()的细节。由于 Windows 没有 fork 调用，因此， multiprocessing 需要“模拟”出 fork 的效果，父进程所有 Python 对象都必须通过 pickle 序列化再传到子进程去，所有，如果 multiprocessing 在 Windows 下调用失败了，要先考虑是不是 pickle 失败了。 

初始化Queue()对象时（例如：q=Queue()），若括号中没有指定最大可接收的消息数量，或数量为负值，那么就代表可接受的消息数量没有上限（直到内存的尽头）。

1. `Queue.qsize()`：返回当前队列包含的消息数量；
2. `Queue.empty()`：如果队列为空，返回True，反之False ；
3. `Queue.full()`：如果队列满了，返回True,反之False；
4. `Queue.get([block[, timeout]])`：获取队列中的一条消息，然后将其从列队中移除，block默认值为True；
   1. 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果为空，此时程序将被阻塞（停在读取状态），直到从消息列队读到消息为止，如果设置了timeout，则会等待timeout秒，若还没读取到任何消息，则抛出"Queue.Empty"异常；
   2. 如果block值为False，消息列队如果为空，则会立刻抛出"Queue.Empty"异常；
5. `Queue.get_nowait()`：相当`Queue.get(False)`；
6. `Queue.put(item,[block[, timeout]])`：将item消息写入队列，block默认值为True；
   1. 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果已经没有空间可写入，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止，如果设置了timeout，则会等待timeout秒，若还没空间，则抛出"Queue.Full"异常；
   2. 如果block值为False，消息列队如果没有空间可写入，则会立刻抛出"Queue.Full"异常；
7. `Queue.put_nowait(item)`：相当`Queue.put(item, False)`；

##### 14.1.5 进程的状态

工作中，任务数往往大于cpu的核数，即一定有一些任务正在执行，而另外一些任务在等待cpu进行执行，因此导致了有了不同的状态。	

![进程的状态](./assets/进程的状态.png)

1. 就绪态：运行的条件都已经满足，正在等在cpu执行
2. 执行态：cpu正在执行其功能
3. 等待态：等待某些条件满足，例如一个程序sleep了，此时就处于等待态

#### 14.2 多线程

多任务可以由多进程完成，也可以由一个进程内的多线程完成。

由于线程是操作系统直接支持的执行单元，因此，高级语言通常都内置多线程的支持，Python 也不例外，并且，Python 的线程是真正的 Posix Thread，而不是模拟出来的线程。 

Python 的标准库提供了两个模块:`_thread` 和` threading`，`_thread` 是低级 模块，`threading` 是高级模块，对`_thread` 进行了封装。绝大多数情况下， 我们只需要使用 `threading` 这个高级模块。 

启动一个线程就是把一个函数传入并创建 Thread 实例，然后调用 start() 开始执行：

```python
import time, threading

# 新线程执行的代码：
def loop():
	print('thread %s is running...' % threading.current_thread().name) 
	n =0
	while n < 5:	
		n = n + 1
		print('thread %s >>> %s' % (threading.current_thread().name, n)) 
		time.sleep(1)
    	print('thread %s ended.' % threading.current_thread().name)
    
print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)

# 执行结果如下：
thread MainThread is running...
thread LoopThread is running...
thread LoopThread >>> 1
thread LoopThread >>> 2
thread LoopThread >>> 3
thread LoopThread >>> 4
thread LoopThread >>> 5
thread LoopThread ended.
thread MainThread ended.
```

由于任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程，Python 的 threading 模块有个 current_thread() 函数，它永远返回当前线程的实例。主线程实例的名字叫 MainThread， 子线程的名字在创建时指定，我们用 LoopThread 命名子线程。名字仅仅在打印时用来显示，完全没有其他意义，如果不起名字 Python 就自动给线程命名为 Thread-1，Thread-2...... 

##### 14.2.1 Lock

多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。 

可以通过`threading.Lock()`来实现锁机制。在使用需要锁的变量时，首先获取锁`lock.acquire()`，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止。

获得锁的线程用完后一定要释放锁，否则那些苦苦等待锁的线程将永远等待下去，成为死线程。 可以使用try…finally来确保锁一定会被释放。

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行，坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。 

##### 14.2.2 多核CPU

在其他语言中，可以使用很多核，但是Python 的线程虽然是真正的线程，但解释器执行代码时，有一个 GIL 锁：Global Interpreter Lock，任何 Python 线程执行前，必须先获得 GIL 锁，然后，每执行 100 条字节码，解释器就自动释放 GIL 锁，让别的线程有机会执行。这个 GIL 全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在 Python 中只能交替执行，即使 100 个线程跑在 100 核 CPU 上，也只能用到 1 个核。 

GIL 是 Python 解释器设计的历史遗留问题，通常我们用的解释器是官方实现的 CPython，要真正利用多核，除非重写一个不带 GIL 的解释器。 

所以，在 Python 中，可以使用多线程，但不要指望能有效利用多核。如果一定要通过多线程利用多核，那只能通过 C 扩展来实现，不过这样就失去了 Python 简单易用的特点。 

不过，也不用过于担心，Python 虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个 Python 进程有各自独立的 GIL 锁，互不影响。 

##### 14.2.3 线程代码封装

通过上一小节，能够看出，通过使用threading模块能完成多任务的程序开发，为了让每个线程的封装性更完美，所以使用threading模块时，往往会定义一个新的子类class，只要继承`threading.Thread`就可以了，然后重写`run`方法

```python
#coding=utf-8
import threading
import time

class MyThread(threading.Thread):
    def run(self):
    	pass

if __name__ == '__main__':
    t = MyThread()
    t.start()
```

##### 14.2.4 小结

多线程编程，模型复杂，容易发生冲突，必须用锁加以隔离，同时，又要小心死锁的发生。 

Python 解释器由于设计时有 GIL 全局锁，导致了多线程无法利用多核。多线程的并发在 Python 中就是一个美丽的梦。 

#### 14.3 ThreadLocal

在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。 

但是局部变量也有问题，就是在函数调用的时候，传递起来很麻烦。需要传递很多次的参数。ThreadLocal因此应运而生。

```python
import threading

# 创建全局 ThreadLocal 对象：
local_school = threading.local()

def process_student():
	# 获取当前线程关联的 student
	std = local_school.student
	print('Hello, %s (in %s)' % (std, threading.current_thread().name))
    
def process_thread(name):
# 绑定 ThreadLocal 的 student
	local_school.student = name 
    process_student()
    
t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()

#执行结果
Hello, Alice (in Thread-A)
Hello, Bob (in Thread-B)
```

全局变量 local_school 就是一个 ThreadLocal 对象，每个 Thread 对它都可以读写 student 属性，但互不影响。你可以把 local_school 看成全局变量，但每个属性如 local_school.student 都是线程的局部变量，可以任意读写而互不干扰，也不用管理锁的问题，ThreadLocal 内部会处理。可以理解为全局变量 local_school 是一个 dict，不但可以用 local_school.student，还可以绑定其他变量，如 local_school.teacher 等等。 

ThreadLocal 最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。 

一个 ThreadLocal 变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。ThreadLocal 解决了参数在一个线程中各个函数之间互相传递的问题。 

#### 14.4 进程 VS 线程

首先，要实现多任务，通常我们会设计 Master-Worker 模式，Master 负责分配任务，Worker 负责执行任务，因此，多任务环境下，通常是一个 Master，多个 Worker。 

多进程模式最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程。(当然主进程挂了所有进程就全挂了，但是 Master 进程只负责分配任务，挂掉的概率低）。著名的 Apache 最早就是采用多进程模式。 

多进程模式的缺点是创建进程的代价大，在 Unix/Linux 系统下，用 fork 调用还行，在 Windows 下创建进程开销巨大。另外，操作系统能同时运行的进程数也是有限的，在内存和 CPU 的限制下，如果有几千个进程同时运行，操作系统连调度都会成问题。 

多线程模式通常比多进程快一点，但是也快不到哪去，而且，多线程模式致命的缺点就是任何一个线程挂掉都可能直接造成整个进程崩溃，因为所有线程共享进程的内存。在 Windows 上，如果一个线程执行的代码出了问题，你经常可以看到这样的提示：“该程序执行了非法操作，即将关闭”，其实往往是某个线程出了问题，但是操作系统会强制结束整个进程。 

在 Windows 下，多线程的效率比多进程要高，所以微软的 IIS 服务器默认采用多线程模式。由于多线程存在稳定性的问题，IIS 的稳定性就不如 Apache。为了缓解这个问题，IIS 和 Apache 现在又有多进程+多线程的混合模式，真是把问题越搞越复杂。

1. 进程是系统进行资源分配和调度的一个独立单位.
2. 线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位。线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.
3. 一个程序至少有一个进程，一个进程至少有一个线程。
4. 线程的划分尺度小于进程(资源比进程少)，使得多线程程序的并发性高。
5. 进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率
6. 线程和进程在使用上各有优缺点：线程执行开销小，但不利于资源的管理和保护；而进程正相反。

##### 14.4.1 线程切换

无论是多进程还是多线程，只要数量一多，效率肯定上不去。

因为操作系统在切换时也会浪费资源进行旧任务状体的保存以及新任务状态的准备。切换过程虽然很快，但是也需要耗费时间。如果有几千个任务同时进行，操作系统可能就主要忙着切换任务，根本没有多少时间去执行任务了。

所以，多任务一旦多到一个限度，就会消耗掉系统所有的资源，结果效率急剧下降，所有任务都做不好。 

##### 14.4.2 计算密集型 VS IO密集型

是否采用多任务的第二个考虑是任务的类型。我们可以把任务分为计算密集型和 IO 密集型。 

计算密集型任务的特点是要进行大量的计算，消耗 CPU 资源，比如计算圆周率、对视频进行高清解码等等，全靠 CPU 的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU 执行任务的效率就越低，所以，要最高效地利用 CPU，计算密集型任务同时进行的数量应当等于 CPU 的核心数。 

计算密集型任务由于主要消耗 CPU 资源，因此，代码运行效率至关重要。Python 这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用 C 语言编写。 

第二种任务的类型是 IO 密集型，涉及到网络、磁盘 IO 的任务都是IO密集型任务，这类任务的特点是 CPU 消耗很少，任务的大部分时间都在等待 IO 操作完成(因为 IO 的速度远远低于 CPU 和内存的速度)。 对于 IO 密集型任务，任务越多，CPU 效率越高，但也有一个限度。常见的大部分任务都是 IO 密集型任务，比如 Web 应用。 

IO 密集型任务执行期间，99%的时间都花在 IO 上，花在 CPU 上的时间很少，因此，用运行速度极快的 C 语言替换用 Python 这样运行速度极低的脚本语言，完全无法提升运行效率。对于 IO 密集型任务，最合适的语言就是开发效率最高(代码量最少)的语言，脚本语言是首选，C语言最差。 

##### 14.4.3 异步IO

考虑到 CPU 和 IO 之间巨大的速度差异，一个任务在执行的过程中大部分时间都在等待 IO 操作，单进程单线程模型会导致别的任务无法并行执行，因此，我们才需要多进程模型或者多线程模型来支持多任务并发执行。 

现代操作系统对 IO 操作已经做了巨大的改进，最大的特点就是支持异步 IO。如果充分利用操作系统提供的异步 IO 支持，就可以用单进程单线程模型来执行多任务，这种全新的模型称为事件驱动模型，Nginx 就是支持异步 IO 的 Web 服务器，它在单核 CPU 上采用单进程模型就可以高效地支持多任务。在多核 CPU 上，可以运行多个进程(数量与 CPU 核心数相同)，充分利用多核 CPU。由于系统总的进程数量十分有限， 因此操作系统调度非常高效。用异步 IO 编程模型来实现多任务是一个 主要的趋势。 

对应到 Python 语言，单进程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。我们会在后面讨论如何编写协程。 

#### 14.5 分布式进程

在 Thread 和 Process 中，应当优选 Process，因为 Process 更稳定，而且，Process 可以分布到多台机器上，而 Thread 最多只能分布到同一台机器的多个 CPU 上。 

Python 的 multiprocessing 模块不但支持多进程，其中 managers 子模块还支持把多进程分布到多台机器上。一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。由于 managers 模块封装很好，不必了解网络通信的细节，就可以很容易地编写分布式多进程程序。 

举个例子:如果我们已经有一个通过 Queue 通信的多进程程序在同一台机器上运行，现在，由于处理任务的进程任务繁重，希望把发送任务的进程和处理任务的进程分布到两台机器上。怎么用分布式进程实现? 

原有的 Queue 可以继续使用，但是，通过 managers 模块把 Queue 通过网络暴露出去，就可以让其他机器的进程访问 Queue 了。 

我们先看服务进程，服务进程负责启动 Queue，把 Queue 注册到网络上，然后往 Queue 里面写入任务: 

```python
# task_master.py

import random, time, queue
from multiprocessing.managers import BaseManager

# 发送任务的队列
task_queue = queue.Queue()
# 接收结果的队列
result_queue = queue.Queue()

# 从 BaseManager 继承的 QueueManager
class QueueManager(BaseManager)
	pass
	
# 把两个 Queue 都注册到网络上, callable 参数关联了 Queue 对象
QueueManager.register('get_task_queue', callable=lambda: task_queue) QueueManager.register('get_result_queue', callable=lambda: result_queue)

# 绑定端口 5000, 设置验证码'abc'
manager = QueueManager(address=('', 5000), authkey=b'abc') 
# 启动 Queue:
manager.start()
# 获得通过网络访问的 Queue 对象
task = manager.get_task_queue()
result = manager.get_result_queue()
# 放几个任务进去
for i in range(10):
    n = random.randint(0, 10000)
    print('Put task %d...' % n)
    task.put(n)
# 从 result 队列读取结果
print('Try get results...') 
for i in range(10):
    r = result.get(timeout=10)
	print('Result: %s' % r) 
# 关闭:
manager.shutdown()
print('master exit.')
```

请注意，当我们在一台机器上写多进程程序时，创建的 Queue 可以直接拿来用，但是，在分布式多进程环境下，添加任务到 Queue 不可以直接对原始的 task_queue 进行操作，那样就绕过了 QueueManager 的封装，必须通过 manager.get_task_queue()获得的 Queue 接口添加。 

然后，在另一台机器上启动任务进程(本机上启动也可以)：

```python
# task_worker.py

import time, sys, queue
from multiprocessing.managers import BaseManager

# 创建类似的 QueueManager:
class QueueManager(BaseManager):
	pass
# 由于这个 QueueManager 只从网络上获取 Queue，所以注册时只提供名字
QueueManager.register('get_task_queue') 
QueueManager.register('get_result_queue')
# 连接到服务器，也就是运行 task_master.py 的机器:
server_addr = '127.0.0.1'
print('Connect to server %s...' % server_addr)
# 端口和验证码注意保持与 task_master.py 设置的完全一致:
m = QueueManager(address=(server_addr, 5000), authkey=b'abc') 
# 从网络连接:
m.connect()
# 获取 Queue 的对象:
task = m.get_task_queue()
result = m.get_result_queue()
# 从 task 队列取任务,并把结果写入 result 队列: 
for i in range(10):
    try:
        n = task.get(timeout=1)
        print('run task %d * %d...' % (n, n))
        r = '%d * %d = %d' % (n, n, n*n)
        time.sleep(1)
        result.put(r)
    except Queue.Empty:
        print('task queue is empty.')
# 处理结束: 
print('worker exit.')
```

任务进程要通过网络连接到服务进程，所以要指定服务进程的 IP。 

现在，可以试试分布式进程的工作效果了。先启动 task_master.py 服务进程: 

```
$ python3 task_master.py
Put task 3411...
Put task 1605...
Put task 1398...
Put task 4729...
Put task 5300...
Put task 7471...
Put task 68...
Put task 4219...
Put task 339...
Put task 7866...
Try get results...
```

task_master.py 进程发送完任务后，开始等待 result 队列的结果。现在启动 task_worker.py 进程: 

```
$ python3 task_worker.py
Connect to server 127.0.0.1...
run task 3411 * 3411...
run task 1605 * 1605...
run task 1398 * 1398...
run task 4729 * 4729...
run task 5300 * 5300...
run task 7471 * 7471...
run task 68 * 68...
run task 4219 * 4219...
run task 339 * 339...
run task 7866 * 7866...
worker exit.
```

task_worker.py 进程结束，在 task_master.py 进程中会继续打印出结果: 

```
Result: 3411 * 3411 = 11634921
Result: 1605 * 1605 = 2576025
Result: 1398 * 1398 = 1954404
Result: 4729 * 4729 = 22363441
Result: 5300 * 5300 = 28090000
Result: 7471 * 7471 = 55815841
Result: 68 * 68 = 4624
Result: 4219 * 4219 = 17799961
Result: 339 * 339 = 114921
Result: 7866 * 7866 = 61873956
```

这个简单的 Master/Worker 模型有什么用?其实这就是一个简单但真正的分布式计算，把代码稍加改造，启动多个 worker，就可以把任务分布到几台甚至几十台机器上，比如把计算 n*n 的代码换成发送邮件，就实现了邮件队列的异步发送。 

Queue 对象存储在哪？注意到 task_worker.py 中根本没有创建 Queue 的代码，所以，Queue 对象存储在 task_master.py 进程中: 

而 Queue 之所以能通过网络访问，就是通过 QueueManager 实现的。由于 QueueManager 管理的不止一个 Queue，所以，要给每个 Queue 的网络调用接口起个名字，比如 get_task_queue。 

authkey 有什么用?这是为了保证两台机器正常通信，不被其他机器恶意干扰。如果 task_worker.py 的 authkey 和 task_master.py 的 authkey 不一致，肯定连接不上。 

**小结**

Python 的分布式进程接口简单，封装良好，适合需要把繁重任务分布到多台机器的环境下。 

注意 Queue 的作用是用来传递任务和接收结果，每个任务的描述数据量要尽量小。比如发送一个处理日志文件的任务，就不要发送几百兆的日志文件本身，而是发送日志文件存放的完整路径，由 Worker 进程再去共享的磁盘上读取文件。 

### 15. 正则表达式

正则表达式是一种用来匹配字符串的强有力的武器。它的设计思想是用一种描述性的语言来给字符串定义一个规则，凡是符合规则的字符串，我们就认为它“匹配”了，否则，该字符串就是不合法的。 

在正则表达式中，匹配规则：

- 如果直接给出字符，就是精确匹配。
- `\d`： 匹配一 个数字；
- `\D`：匹配非数字，即不是数字；
- `\w` ：匹配一个字母或数字；
- `\W`：匹配非单词字符；
- `\s`：匹配一个空格（也包括Tab等空白符）
- `\S`：匹配非空白
- `.`：匹配任意字符

匹配变长的字符：

- `*`：表示任意个字符（包括0个）；
- `+`：表示至少一个字符；
- `?`：表示0个或1个字符；
- `{n}`：表示n个字符；
- `{n, m}`：表示n-m个字符

用[]表示范围：

- `[0-9a-zA-Z\_]`：可以匹配一个数字、字母或者下划线；
- `[0-9a-zA-Z\_]+`：可以匹配至少由一个数字、字母或者下划线组成的字符串，如'a100'，'0_Z'，'Py3000'等等；
- `[a-zA-Z\_][0-9a-zA-Z\_]*`：可以匹配由字母或下划线开头，后接任意个由一个数字、字母或者下划线组成的字符串，也就是 Python 合法的变量；
- `[a-zA-Z\_][0-9a-zA-Z\_]{0, 19}`：更精确地限制了变量的长度是 1-20 个字符(前面 1 个字符+后面最多 19 个字符)。 

- `A|B`：可以匹配 A 或 B ；

- `^`：表示行的开头，`^\d`表示必须以数字开头；
- `$`：表示行的结束，`\d$`表示必须以数字结束；

#### 15.1 re模块 

Python 提供 re 模块，包含所有正则表达式的功能。

```python
>>> import re
>>> re.match(r'^\d{3}\-\d{3,8}$', '010-12345')	#match()方法判断是否匹配，如果匹配成功，返回一个Match对象，否则返回None。这里-是特殊字符。
<_sre.SRE_Match object; span=(0, 9), match='010-12345'>
>>> re.match(r'^\d{3}\-\d{3,8}$', '010 12345')
>>>
```

##### 15.1.1 切分字符串

用正则表达式切分字符串比用固定的字符更灵活，请看正常的切分代码:

```python
>>> 'a b   c'.split(' ')
['a', 'b', '', '', 'c']

```

嗯，无法识别连续的空格，用正则表达式试试：

```python
>>> re.split(r'\s+', 'a b   c')
['a', 'b', 'c']
```

##### 15.1.2 分组

除了简单地判断是否匹配之外，正则表达式还有提取子串的强大功能。 用()表示的就是要提取的分组(Group)。 

`^(\d{3})-(\d{3,8})$`分别定义了两个组，可以直接从匹配的字符串中提取出区号和本地号码: 

```python 
>>> m = re.match(r'^(\d{3})-(\d{3,8})$', '010-12345')	#如果定义了组，就可以在Match对象上用group()方法提取出子串来。
>>> m
<_sre.SRE_Match object; span=(0, 9), match='010-12345'>
>>> m.group(0)	# group(0)永远是原始字符串
'010-12345'
>>> m.group(1)	# 1表示第1个子串
'010'
>>> m.group(2)	# 2表示第2个子串
'12345'
```

|     字符     | 功能                             |
| :----------: | :------------------------------- |
|      \|      | 匹配左右任意一个表达式           |
|     (ab)     | 将括号中字符作为一个分组         |
|    `\num`    | 引用分组num匹配到的字符串        |
| `(?P<name>)` | 分组起别名                       |
|  (?P=name)   | 引用别名为name分组匹配到的字符串 |

```python
# 需求：匹配出0-100之间的数字
import re

ret = re.match("[1-9]?\d","8")  # 匹配0-99
print(ret.group())  # 8

ret = re.match("[1-9]?\d","78")  # 匹配0-99
print(ret.group())  # 78

# 不正确的情况
ret = re.match("[1-9]?\d","08")  # 匹配0-99，边界情况
print(ret.group())  # 0

# 修正之后的
ret = re.match("[1-9]?\d$","08")  # 匹配0-99，修正后
if ret:
    print(ret.group())
else:
    print("不在0-100之间")

# 添加|
ret = re.match("[1-9]?\d$|100","8")  # | 用来匹配左右任意一个表达式
print(ret.group())  # 8

ret = re.match("[1-9]?\d$|100","78")
print(ret.group())  # 78

ret = re.match("[1-9]?\d$|100","08")
# print(ret.group())  # 不是0-100之间

ret = re.match("[1-9]?\d$|100","100")
print(ret.group())  # 100
```

```python
# 需求：匹配出163、126、qq邮箱
import re

ret = re.match("\w{4,20}@163\.com", "test@163.com")
print(ret.group())  # test@163.com

ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@126.com")  # 这里为什么用\.来匹配.呢，因为.有自己的含义，需要转义后才能精确匹配
print(ret.group())  # test@126.com

ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@qq.com")
print(ret.group())  # test@qq.com

ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@gmail.com")
if ret:
    print(ret.group())
else:
    print("不是163、126、qq邮箱")  # 不是163、126、qq邮箱
```

```python
# 匹配不是以4、7结尾的手机号码(11位)
import re

tels = ["13100001234", "18912344321", "10086", "18800007777"]

for tel in tels:
    ret = re.match("1\d{9}[0-35-68-9]", tel)
    if ret:
        print(ret.group())
    else:
        print("%s 不是想要的手机号" % tel)
```

```python
# 提取区号和电话号码
ret = re.match("([^-]*)-(\d+)","010-12345678")
ret.group()  # '010-12345678'
ret.group(1)  # '010'
ret.group(2)  # '12345678'    所以这是分组的一个很实用的用途？
```

```python
# 需求：匹配出<html>hh</html>
import re

# 能够完成对正确的字符串的匹配
ret = re.match("<[a-zA-Z]*>\w*</[a-zA-Z]*>", "<html>hh</html>")
print(ret.group())  # <html>hh</html>

# 如果遇到非正常的html格式字符串，匹配出错
ret = re.match("<[a-zA-Z]*>\w*</[a-zA-Z]*>", "<html>hh</htmlbalabala>")
print(ret.group())  # <html>hh</htmlbalabala>

# 正确的理解思路：如果在第一对<>中是什么，按理说在后面的那对<>中就应该是什么

# 通过引用分组中匹配到的数据即可，但是要注意是元字符串，即类似 r""这种格式
ret = re.match(r"<([a-zA-Z]*)>\w*</\1>", "<html>hh</html>")
print(ret.group())  # <html>hh</html>

# 因为2对<>中的数据不一致，所以没有匹配出来
test_label = "<html>hh</htmlbalabala>"
ret = re.match(r"<([a-zA-Z]*)>\w*</\1>", test_label)  # 这是一对不正确的标签
if ret:
    print(ret.group())  
else:
    print("%s 这是一对不正确的标签" % test_label)
```

```python
# 需求：匹配出<html><h1>www.itcast.cn</h1></html>，使用\num
import re

labels = ["<html><h1>www.itcast.cn</h1></html>", "<html><h1>www.itcast.cn</h2></html>"]

for label in labels:
    ret = re.match(r"<(\w*)><(\w*)>.*</\2></\1>", label)
    if ret:
        print("%s 是符合要求的标签" % ret.group())  
    else:
        print("%s 不符合要求" % label)
# 结果：
# <html><h1>www.itcast.cn</h1></html> 是符合要求的标签
# <html><h1>www.itcast.cn</h2></html> 不符合要求
```

```python
# 需求：匹配出<html><h1>www.itcast.cn</h1></html>，使用 (?P<name>) (?P=name)
import re

ret = re.match(r"<(?P<name1>\w*)><(?P<name2>\w*)>.*</(?P=name2)></(?P=name1)>", "<html><h1>www.itcast.cn</h1></html>")
ret.group()  # <html><h1>www.itcast.cn</h1></html>

ret = re.match(r"<(?P<name1>\w*)><(?P<name2>\w*)>.*</(?P=name2)></(?P=name1)>", "<html><h1>www.itcast.cn</h2></html>")
ret.group()  # 匹配失败
# 注意：这里的p是大写的。
```

##### 15.1.3 贪婪匹配

正则匹配默认是贪婪匹配，也就是匹配尽可能多的字符。举例如下，匹配出数字后面的 0：

```python
>>> re.match(r'^(\d+)(0*)$', '102300').groups()
('102300', '')
```

由于`\d+`采用贪婪匹配，直接把后面的 0 全部匹配了，结果`0*`只能匹配空字符串了。 

必须让`\d+`采用非贪婪匹配(也就是尽可能少匹配)，才能把后面的0匹配出来，加个`?`就可以让`\d+`采用非贪婪匹配：

```python
>>> re.match(r'^(\d+?)(0*)$', '102300').groups()
('1023', '00')
```

可以在`*`,`?`,`+`,`{m,n}`后面加上`?`，使贪婪变成非贪婪。

##### 15.1.4 预编译

当我们在Python中使用正则表达式时，re模块内部会干两件事情：

1. 编译正则表达式，如果正则表达式的字符串本身不合法，会报错；
2. 用编译后的正则表达式去匹配字符串。

如果一个正则表达式要重复使用几千次，出于效率的考虑，我们可以预编译该正则表达式，接下来重复使用时就不需要编译这个步骤了，直接匹配：

```python
>>> import re
# 编译:
>>> re_telephone = re.compile(r'^(\d{3})-(\d{3,8})$') # 使用:
>>> re_telephone.match('010-12345').groups()
('010', '12345')
>>> re_telephone.match('010-8086').groups()
('010', '8086')
```

编译后生成 Regular Expression 对象，由于该对象自己包含了正则表达式，所以调用对应的方法时不用给出正则字符串。 

#### 15.2 re的高级用法

##### 15.2.1 `search`

在字符串中查找可以匹配的字符串。

```python
# 需求：匹配出文章阅读的次数
import re

ret = re.search(r"\d+", "阅读次数为 9999")
ret.group()  # 9999
```

##### 15.2.2 `findall`

查询出字符串中所有符合匹配条件的子字符串。

```python
# 统计出python、c、c++相应文章阅读的次数
import re

ret = re.findall(r"\d+", "python = 9999, c = 7890, c++ = 12345")
print(ret)  # ['9999', '7890', '12345']
```

##### 15.2.3 `sub`

将匹配到的数据进行替换

```python
# 需求：将匹配到的阅读次数加1
# 方法1
import re

ret = re.sub(r"\d+", '998', "python = 997")
print(ret)
# 方法2
def add(temp):
    strNum = temp.group()
    num = int(strNum) + 1
    return str(num)

ret = re.sub(r"\d+", add, "python = 997")
print(ret)  # python = 998

ret = re.sub(r"\d+", add, "python = 99")
print(ret)  # python = 100
```

示例：在下列字符串中取出文本

```html
<div>
        <p>岗位职责：</p>
<p>完成推荐算法、数据统计、接口、后台等服务器端相关工作</p>
<p><br></p>
<p>必备要求：</p>
<p>良好的自我驱动力和职业素养，工作积极主动、结果导向</p>
<p>&nbsp;<br></p>
<p>技术要求：</p>
<p>1、一年以上 Python 开发经验，掌握面向对象分析和设计，了解设计模式</p>
<p>2、掌握HTTP协议，熟悉MVC、MVVM等概念以及相关WEB开发框架</p>
<p>3、掌握关系数据库开发设计，掌握 SQL，熟练使用 MySQL/PostgreSQL 中的一种<br></p>
<p>4、掌握NoSQL、MQ，熟练使用对应技术解决方案</p>
<p>5、熟悉 Javascript/CSS/HTML5，JQuery、React、Vue.js</p>
<p>&nbsp;<br></p>
<p>加分项：</p>
<p>大数据，数理统计，机器学习，sklearn，高性能，大并发。</p>

        </div>
```

答案：正则表达式：`re.sub(r"<[^>]*>|&nbsp;|\n", " ", str)`   ????为什么，没看懂。

##### 15.2.4 `split`

可以根据匹配进行切割字符串，并返回一个列表

```python
import re

ret = re.split(r":| ","info:xiaoZhang 33 shandong")
print(ret)  # ['info', 'xiaoZhang', '33', 'shandong']
```

#### 15.3 `r`的作用

与大多数编程语言相同，`正则表达式里使用"\"作为转义字符`，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，那么使用编程语言表示的正则表达式里将需要4个反斜杠"\\"：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。

Python里的原生字符串很好地解决了这个问题，有了原生字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观。`Python中字符串前面加上 r 表示原生字符串`。

示例：

```python
# 不适用r的情况
>>> mm = "c:\\a\\b\\c"
>>> mm
'c:\\a\\b\\c'
>>> print(mm)
c:\a\b\c
>>> re.match("c:\\\\",mm).group()
'c:\\'
>>> ret = re.match("c:\\\\",mm).group()
>>> print(ret)
c:\
>>> ret = re.match("c:\\\\a",mm).group()
>>> print(ret)
c:\a
>>> ret = re.match(r"c:\\a",mm).group()  # 原生字符串其实就是不进行转义处理的字符串，这里有两个是因为需要在正则表达式中转义
>>> print(ret)
c:\a
>>> ret = re.match(r"c:\a",mm).group()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'NoneType' object has no attribute 'group'
```

### 16. 常用内建模块

Python 之所以自称“batteries included”，就是因为内置了许多非常有用的模块，无需额外安装和配置，即可直接使用。 本章将介绍一些常用的内建模块。

#### 16.1 `datetime`

`datetime`是Python处理日期和时间的标准库。

```python
>>> from datetime import datetime	# datetime模块还包含了一个datetime类

# 获取当前 datetime
>>> now = datetime.now()	
>>> print(now)
2015-05-18 16:28:07.198690
        
# 用指定日期时间创建 datetime
>>> dt = datetime(2015, 4, 19, 12, 20)	
>>> print(dt)
2015-04-19 12:20:00

# 把 datetime 转换为 timestamp
>>> dt.timestamp() 
1429417200.0	# Python 的 timestamp是一个浮点数。如果有小数位，小数位表示毫秒数
# timestamp的值与时区毫无关系，因为timestamp一旦确定，期UTC时间就确定了，转换到任意时区的时间也是完全确定的，这就是为什么计算机存储的当前时间是以timestamp表示的，因为全球各地的计算机在任意时刻的timestamp都是完全相同的。

# 把 timestamp 转换为 datetime 
>>> t = 1429417200.0
>>> print(datetime.fromtimestamp(t))    # 转换为本地时间
2015-04-19 12:20:00
>>> print(datetime.utcfromtimestamp(t)) # 转换为UTC时间
2015-04-19 04:20:00
     
# 把 str 转换为 datetime
>>> cday = datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')  
>>> print(cday)
2015-06-01 18:19:59		# 通过字符串转换后的datetime是没有时区信息的
       
# 把 datetime 转换为 str
>>> now = datetime.now()
>>> print(now.strftime('%a, %b %d %H:%M'))	# a b d等是日期和时间的格式
Mon, May 05 16:28
    
# datetime 加减，timedelta用于加减
>>> from datetime import datetime, timedelta	
>>> now = datetime.now()
>>> now
datetime.datetime(2015, 5, 18, 16, 57, 3, 540997)
>>> now + timedelta(hours=10)		# +
datetime.datetime(2015, 5, 19, 2, 57, 3, 540997)
>>> now - timedelta(days=1)			# -
datetime.datetime(2015, 5, 17, 16, 57, 3, 540997)

# 本地时间转换成UTC时间
# 本地时间是指系统设定时区的时间，例如北京时间是UTC+8:00时区的时间，而UTC时间指UTC+0:00时区的时间。
# 一个 datetime 类型有一个时区属性 tzinfo，但是默认为 None，所以无法区分这个 datetime 到底是哪个时区，除非强行给 datetime 设置一个时区:
>>> from datetime import datetime, timedelta, timezone
>>> tz_utc_8 = timezone(timedelta(hours=8)) # 创建时区 UTC+8:00 
>>> now = datetime.now()
>>> now
datetime.datetime(2015, 5, 18, 17, 2, 10, 871012)
>>> dt = now.replace(tzinfo=tz_utc_8) # 强制设置为 UTC+8:00
>>> dt
datetime.datetime(2015, 5, 18, 17, 2, 10, 871012, tzinfo=datetime.timezone(datetime.timedelta(0, 28800)))
# 如果系统时区恰好是 UTC+8:00，那么上述代码就是正确的，否则，不能强制设置为 UTC+8:00 时区。

# 时区转换
#我们可以先通过 utcnow()拿到当前的 UTC 时间，再转换为任意时区的时间:
# 拿到 UTC 时间，并强制设置时区为 UTC+0:00:
>>> utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc) 
>>> print(utc_dt)
2015-05-18 09:05:12.377316+00:00
# astimezone()将转换时区为北京时间:
>>> bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8))) 
>>> print(bj_dt)
2015-05-18 17:05:12.377316+08:00
# astimezone()将转换时区为东京时间:
>>> tokyo_dt = utc_dt.astimezone(timezone(timedelta(hours=9))) 
>>> print(tokyo_dt)
2015-05-18 18:05:12.377316+09:00
# astimezone()将 bj_dt 转换时区为东京时间:
>>> tokyo_dt2 = bj_dt.astimezone(timezone(timedelta(hours=9))) 
>>> print(tokyo_dt2)
2015-05-18 18:05:12.377316+09:00
# 时区转换的关键在于，拿到一个 datetime 时，要获知其正确的时区，然后强制设置时区，作为基准时间。利用带时区的 datetime，通过 astimezone()方法，可以转换到任意时区。
# 注:不是必须从 UTC+0:00 时区转换到其他时区，任何带时区的 datetime 都可以正确转换，例如上述 bj_dt 到 tokyo_dt 的转换。
```

`datetime` 表示的时间需要时区信息才能确定一个特定的时间，否则只能视为本地时间。 

如果要存储 `datetime`，最佳方法是将其转换为 `timestamp` 再存储，因为 `timestamp` 的值与时区完全无关。 

#### 16.2 `collections`

collections 是 Python 内建的一个集合模块，提供了许多有用的集合类。 

##### 16.2.1 `namedtuple`

tuple表示不变集合，例如，一个点的二维坐标就可以表示成：`>>> p = (1, 2) `，但是，看到(1, 2)，很难看出这个 tuple 是用来表示一个坐标的。 定义一个 class 又小题大做了，这时，`namedtuple` 就派上了用场：

```python
>>> from collections import namedtuple
# namedtuple('名称', [属性 list]):
>>> Point = namedtuple('Point', ['x', 'y'])  # 类名 Point， 变量名Point
>>> p = Point(1, 2)
>>> p.x
1
>>> p.y
2
```

`namedtuple` 是一个函数，它用来创建一个自定义的 tuple 对象，并且规定了 tuple 元素的个数，并可以用属性而不是索引来引用 tuple 的某个元素。 

这样一来，我们用 `namedtuple` 可以很方便地定义一种数据类型，它具备 tuple 的不变性，又可以根据属性来引用，使用十分方便。

可以验证创建的 Point 对象是 tuple 的一种子类: 

```python
>>> isinstance(p, Point)
True
>>> isinstance(p, tuple)
True
```

##### 16.2.2 `deque`

使用 list 存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为 list 是线性存储，数据量大的时候，插入和删除效率很低。 

`deque` 是为了高效实现插入和删除操作的双向列表，适合用于队列和栈: 

```python
>>> from collections import deque
>>> q = deque(['a', 'b', 'c'])
>>> q.append('x')			# 添加到右边	    pop()
>>> q.appendleft('y')		# 添加到左边		popleft()
>>> q
deque(['y', 'a', 'b', 'c', 'x'])
```

##### 16.2.3 `defaultdict `

使用 dict 时，如果引用的 Key 不存在，就会抛出 KeyError。如果希望key不存在时，返回一个默认值，就可以用 defaultdict：

```python
>>> from collections import defaultdict 
>>> dd = defaultdict(lambda: 'N/A') # 除了Key不存在时返回默认值，其他行为跟dict完全一样
>>> dd['key1'] = 'abc'
>>> dd['key1'] # key1 存在
'abc'
>>> dd['key2'] # key2 不存在，返回默认值 
'N/A'
```

##### 16.2.4 `OrderedDict`

使用 dict 时，Key 是无序的。在对dict做迭代时，我们无法确定 Key 的顺序。 OrderedDict可以保持Key的顺序。

```python
>>> from collections import OrderedDict
>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
>>> d # dict 的 Key 是无序的
{'a': 1, 'c': 3, 'b': 2}
>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)]) 
>>> od # OrderedDict 的 Key 是有序的，Key按照插入的顺序排序，而不是Key本身排序。
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
```

##### 16.2.5 `Counter`

Counter是一个简单的计数器，例如，统计字符出现的个数：

```python
>>> from collections import Counter
>>> c = Counter()
>>> for ch in 'programming':
...     c[ch] = c[ch] + 1
...
>>> c
Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})
```

Counter实际上是dict的一个子类。

#### 16.3 `base64`

Base64是一种用64个字符来表示任意二进制数据的方法。是一种常见的二进制编码方法。

Base64的原理很简单，首先，准备一个包含64个字符的数组：

```
['A', 'B', 'C', ... 'a', 'b', 'c', ... '0', '1', ... '+', '/']
```

然后，对二进制数据进行处理，每3个字节一组，一共是3*8=24bit，化为4组，每组正好6个bit。这样我们就得到4个数字作为索引，然后查表，获得相应的 4 个字符，就是编码后的字符串。 	

所以，Base64 编码会把 3 字节的二进制数据编码为 4 字节的文本数据，长度增加 33%，好处是编码后的文本数据可以在邮件正文、网页等直接显示。 

如果要编码的二进制数据不是 3 的倍数，最后会剩下 1 个或 2 个字节怎么办？Base64 用\x00 字节在末尾补足后，再在编码的末尾加上 1 个或 2个=号，表示补了多少字节，解码的时候，会自动去掉。 

Python 内置的 base64 可以直接进行 base64 的编解码: 

```python
>>> import base64
>>> base64.b64encode(b'binary\x00string')
b'YmluYXJ5AHN0cmluZw=='
>>> base64.b64decode(b'YmluYXJ5AHN0cmluZw==')
b'binary\x00string'
```

由于标准的 Base64 编码后可能出现字符+和/，在 URL 中就不能直接作为参数，所以又有一种“url safe”的 base64 编码，其实就是把字符+和/分 别变成-和_：

```python
>>> base64.b64encode(b'i\xb7\x1d\xfb\xef\xff')
b'abcd++//'
>>> base64.urlsafe_b64encode(b'i\xb7\x1d\xfb\xef\xff')
b'abcd--__'
>>> base64.urlsafe_b64decode('abcd--__')
b'i\xb7\x1d\xfb\xef\xff'
```

还可以自己定义 64 个字符的排列顺序，这样就可以自定义 Base64 编码，不过，通常情况下完全没有必要。 

Base64 是一种通过查表的编码方法，不能用于加密，即使使用自定义的编码表也不行。 

Base64 适用于小段内容的编码，比如数字证书签名、Cookie 的内容等。 

由于=字符也可能出现在 Base64 编码中，但=用在 URL、Cookie 里面会造成歧义，所以，很多 Base64 编码后会把=去掉: 

```python
# 标准 Base64
'abcd' -> 'YWJjZA==' 
# 自动去掉=
'abcd' -> 'YWJjZA'
```

去掉=后怎么解码呢？因为 Base64 是把 3 个字节变为 4 个字节，所以， Base64 编码的长度永远是 4 的倍数，因此，需要加上=把 Base64 字符串的长度变为 4 的倍数，就可以正常解码了。 

Base64 是一种任意二进制到文本字符串的编码方法，常用于在 URL、 Cookie、网页中传输少量二进制数据。 

#### 16.4 `struct`

准确地讲，Python 没有专门处理字节的数据类型。但由于 b'str' 既是字符串，又可以表示字节，所以，字节数组=二进制str。 

在 Python 中，比方说要把一个32位无符号整数变成字节，也就是 4 个长度的 bytes，你得配合位运算符这么写：

```python
>>> n = 10240099
>>> b1 = (n & 0xff000000) >> 24
>>> b2 = (n & 0xff0000) >> 16
>>> b3 = (n & 0xff00) >> 8
>>>  
>>> bs = bytes([b1, b2, b3, b4], encoding='utf8')
>>> bs
b'\x00\x9c@c'
```

非常麻烦。如果换成浮点数就无能为力了。

好在 Python 提供了一个 struct 模块来解决 bytes 和其他二进制数据类型的转换。 

struct 的 pack 函数把任意数据类型变成 bytes：

```python
>>> import struct
>>> struct.pack('>I', 10240099) # pack 的第一个参数是处理指令，'>I'的意思是: >表示字节顺序是 big-endian，也就是网络序，I 表示 4 字节无符号整数。后面的参数个数要和处理指令一致。
b'\x00\x9c@c'
>>> struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80') # unpack 把 bytes 变成相应的数据类型
(4042322160, 32896)	#根据>IH的说明，后面的bytes依次变为I:4字节无符号整数和H:2字节无符号整数。
```

所以，尽管 Python 不适合编写底层操作字节流的代码，但在对性能要求不高的地方，利用 struct 就方便多了。 

#### 16.5 `hashlib`

Python的`hashlib`提供了常见的摘要算法，如MD5、SHA1等。

摘要算法又称哈希算法、散列算法。它通过一个函数，把任意长度的数据转换为一个长度固定的数据串（通常用16进制的字符串表示）。

摘要算法就是通过摘要函数`f()`对任意长度的数据data计算出固定长度的摘要digest，目的是为了发现原始数据是否被人篡改过。

摘要算法之所以能支出数据是否被篡改过，就是因为摘要函数是一个单向函数，计算`f(data)`很容易，但通过digest反推data却非常困难。而且，对原始数据一个bit的修改，都会导致计算出的摘要完全不同。

MD5是最常见的摘要算法，速度很快，生成结果是固定的128bit字节，通常用一个32位的16进制字符串表示；SHA1的结果是160bit，通常用一个40位的16进制字符串表示。

比SHA1更安全的算法是SHA256和SHA512，不过越安全的算法不仅越慢，而且摘要长度更长。

有没有可能两个不同的数据通过某个摘要算法得到了相同的摘要？

答：完全有可能，因为任何摘要算法都是把无限多的数据集合映射到一个有限的集合中。这种情况称为碰撞，但是非常非常困难。

实例应用：

可以在数据库中存储密码的摘要，防止明文密码被泄露。

由于常用口令的MD5值很容易被计算出来，所以，要确保存储的用户口令不是那些已经被计算出来的常用口令的MD5，这一方法通过对原始口令加一个复杂字符串来实现，俗称“加盐”。

摘要算法在很多地方都有广泛的应用。要注意摘要算法不是加密算法，不能用于加密（因为无法通过摘要反推明文），只能用于防篡改，但是它的单向计算特性决定了可以在不存储明文口令的情况下验证用户口令。

#### 16.6 `hmac`

如果Salt是我们自己随机生成的，通常我们计算MD5时采用`md5(message + salt)`。但实际上，把salt看做一个“口令”。加Salt的哈希就是：计算一段message的哈希时，根据不同口令计算出不同的哈希。要验证哈希值，必须同时提供正确的口令。

这实际上就是Hmac算法：Keyed-Hashing for Message Authentication。它通过一个标准算法，在计算哈希的过程中，把key混入计算过程中。

Hmac算法针对所有哈希算法都通用，无论是MD5还是SHA-1。采用Hmac替代我们自己的halt算法，可以使程序更标准化，也更安全。

Python自带的Hmac模块实现了标准的Hmac算法。

Python内置的hmac模块实现了标准的Hmac算法，它利用一个key对message计算“杂凑”后的hash，使用hmac算法比标准hash算法更安全，因为针对相同的message，不同的key会产生不同的hash。

#### 16.7 `itertools`

Python的内建模块`itertools`提供了非常有用的用于操作迭代对象的函数。

```python
import itertools
natuals = itertools.count(1)  # 会创建一个无限的迭代器
	for n in natuals:
		print(n)
cs = itertools.cycle('ABC')  # 会把传入的序列无限重复下去
ns = itertools.repeat('A', 3)  # 负责把一个元素无限重复下去，第二个参数可以限定重复次数

ns1 = itertools.takewhile(lambda x: x <= 10, natuals)  # 无限序列虽然可以无限迭代下去，但是通常我们会通过takewhile()等函数根据条件判断来截取出一个有限的序列。
```

无限序列只在for迭代时才会无限地迭代下去，如果只是创建了一个迭代对象，它不会事先把无限个元素生成出来，事实上也不可能在内存中创建无限多个元素。

##### 16.7.1 `chain()`

`chain()`可以把一组迭代对象串联起来，形成一个更大的迭代器。

```python
import itertools
for c in itertools.chain('ABC', 'XYZ'):
	print(c)

# 结果A	B	C	X	Y	Z
```

16.7.2 `groupby()`

`groupby()`函数把迭代器中相邻的重复元素挑出来放在一起。

```python
for key, group in itertools.groupby('AAABBBCCAAA'):
	print(key, list(group))
    
# 结果
A ['A', 'A', 'A']
B ['B', 'B', 'B']
C ['C', 'C']
A ['A', 'A', 'A']
```

`itertools`模块提供的全部是处理迭代功能的函数，它们的返回值不是list，而是Iterator，只有用for循环迭代的时候才真正计算。

#### 16.8 `contextlib`

在Python中，读写文件这样的资源要特别注意，必须在使用完毕后正确关闭它们。通常，使用with 语句会比`try...finally...`更加方便。事实上，并不是只有`open()`函数返回的fp对象才能使用with语句。任何对象，只要正确实现了上下文管理，就可以用于with语句。

实现上下文管理是通过`__enter__`和`__exit__`这两个方法实现的。如下：

```python
class Query(object):
	
	def __init__(self, name):
		self.name = name
		
	def __enter__(self):
		print('Begin')
		return self
	
	def __exit__(self, exc_type, exc_value, traceback):
		if exc_type:
			print('Error')
		else:
			print('End')
	
	def query(self):
		print('Query info about %s...' % self.name)
```

调用：

```python
with Query('Bob') as q:
	q.query()
```



##### 16.8.1`@contextmanager`

编写`__enter__`和`__exit__`仍然很繁琐，因此Python的标准库contextlib提供了更简单的写法，上面的代码可以改写为：

```python
form contextlib import contextmanager

class Query(object):
	
	def __init__(self, name):
		self.name = name
	
	def query(self):
		print('Query info about %s...' % self.name)
		
@contextmanager
def create_query(name):
	print('Begin')
	q = Query(name)
	yield q
	print('End')
```

调用：

```python
with create_query(name):
	q.query()
```

`@contextmanager`让我们通过编写generator来简化上下文管理。

##### 16.8.2 `@closing`

如果一个对象没有实现上下文，我们就不能把它用于with语句。这个时候，可以用`closing()`来把该对象变为上下文对象。例如，用with语句使用`urlopen()`：

```python
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.python.org')) as page:
	for line in page:
		print(line)
```

`closing`也是一个经过`@contextmanager`装饰的generator，这个generator编写起来非常简单：

```python
@contextmanager
def closing(thing):
	try:
		yield thing
	finally:
		thing.close()
```

它的作用就是把任意对象变为上下文对象，并支持with语句。

16.9 `urllib`

`urllib`提供了一系列用于操作URL的功能。

`urllib`的request模块可以非常方便地抓取URL内容，也就是发送一个GET请求到指定的页面，然后返回HTTP的响应。

`urllib`提供的功能就是利用程序去执行各种HTTP请求。如果要模拟浏览器完成特定功能，需要把请求伪装成浏览器。伪装的方法是先监控浏览器发出的请求，再根据浏览器的请求头来伪装，User-Agent就是来标识浏览器的。

#### 16.9 XML

操作XML有两种方法：DOM和SAX。DOM会把整个XML读入内存，解析为树，因此占用内存大，解析慢，优点是可以任意遍历树的节点。SAX是流模式，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件。

正常情况下，优先考虑SAX，因为DOM实在太占内存。

在Python中使用SAX解析XML非常简洁，通常我们关心的事件是`start_element`、`end_element`、`char_data`，准备好这3个函数，然后就可以解析xml了。

### 17.  异常

#### 17.1 异常定义

程序在运行时，如果Python解释器遇到一个错误，会停止程序的执行，并且提示一些错误信息，这就是异常。

程序停止执行并且提示错误信息这个动作，我们通常称之为：抛出（raise）异常。

程序开发时，很难将所有的特殊情况都处理的面面俱到，通过异常捕捉可以针对突发事件做集中的处理，从而保证程序的稳定性和健壮性。

#### 17.2 异常处理

```python
try:
    # 尝试执行的代码
    pass
except 错误类型1:
    # 针对错误类型1，对应的代码处理
    pass
except 错误类型2:
    # 针对错误类型2，对应的代码处理
    pass
except (错误类型3, 错误类型4):
    # 针对错误类型3 和 4，对应的代码处理
    pass
except Exception as result:
    # 打印错误信息
    print(result)
else:
    # 没有异常才会执行的代码
    pass
finally:
    # 无论是否有异常，都会执行的代码
    print("无论是否有异常，都会执行的代码")
```

#### 17.3 异常传递

当函数/方法执行出现异常，会将异常传递给函数/方法的调用一方。如果传递到主程序，仍然没有异常处理，程序会被终止。

在开发中，可以在主函数中增加异常捕获，而在主函数中调用的其他函数，只要出现异常，都会传递到主函数的异常捕获中，这样就不需要再代码中，增加大量的异常捕获，能够保证代码的清洁。

#### 17.4 异常抛出

在Python中提供了一个Exception异常类，在开发中，如果满足特定业务需求时，希望抛出异常，可以：

1. 创建一个`Exception`的对象
2. 使用`raise`关键字抛出异常对象

```python
ex = Exception("密码长度不够")
raise ex
```

### 18. 文件操作

在终端/文件浏览器中可以执行常规的文件/目录管理操作，例如：创建、重命名、删除、改变路径、查看目录内容、......

在Python中，如果希望通过程序实现上述功能，需要导入`os`模块。

文件操作：

| 序号 | 方法名 | 说明       | 示例                            |
| ---- | ------ | ---------- | ------------------------------- |
| 01   | rename | 重命名文件 | os.rename(源文件名, 目标文件名) |
| 02   | remove | 删除文件   | os.remove(文件名)               |

目录操作：

| 序号 | 方法名     | 说明           | 示例                    |
| ---- | ---------- | -------------- | ----------------------- |
| 01   | listdir    | 目录列表       | os.listdir(目录名)      |
| 02   | mkdir      | 创建目录       | os.mkdir(目录名)        |
| 03   | rmdir      | 删除目录       | os.rmdir(目录名)        |
| 04   | getcwd     | 获取当前目录   | os.getcwd()             |
| 05   | chdir      | 修改工作目录   | os.chdir(目标目录)      |
| 06   | path.isdir | 判断是否是文件 | os.path.isdir(文件路径) |

文件或目录操作都支持相对路径和绝对路径

### 19. `eval()`

`eval()`函数十分强大——将字符串当成有效的表达式来求值并返回计算结果。

```python
# 基本的数学计算
In [1]: eval("1 + 1")
Out[1]: 2

# 字符串重复
In [2]: eval("'*' * 10")
Out[2]: '**********'

# 将字符串转换成列表
In [3]: type(eval("[1, 2, 3, 4, 5]"))
Out[3]: list

# 将字符串转换成字典
In [4]: type(eval("{'name': 'xiaoming', 'age': 18}"))
Out[4]: dict
```

代码执行终端命令

```python
eval(__import__('os').system('ls'))
# 等价于
import os
os.system("终端命令")  # 执行成功，返回0,；执行失败，返回错误信息
```

### 20. 协程

#### 20.1 协程定义

协程，又称微线程、纤程。英文名Coroutine。

协程是python个中另外一种实现多任务的方式，只不过比线程更小占用更小执行单元（理解为需要的资源）。 为啥说它是一个执行单元，因为它自带CPU上下文。这样只要在合适的时机， 我们可以把一个协程 切换到另一个协程。 只要这个过程中保存或恢复 CPU上下文那么程序还是可以运行的。

通俗的理解：在一个线程中的某个函数，可以在任何地方保存当前函数的一些临时变量等信息，然后切换到另外一个函数中执行，注意不是通过调用函数的方式做到的，并且切换的次数以及什么时候再切换到原来的函数都由开发者自己确定。

#### 20.2 协程与线程的差异

在实现多任务时, 线程切换从系统层面远不止保存和恢复 CPU上下文这么简单。 操作系统为了程序运行的高效性每个线程都有自己缓存Cache等等数据，操作系统还会帮你做这些数据的恢复操作。 所以线程的切换非常耗性能。但是协程的切换只是单纯的操作CPU的上下文，所以一秒钟切换个上百万次系统都抗的住。

#### 20.3 协程的简单实现（yield）

```python
import time

def work1():
    while True:
        print("----work1---")
        yield
        time.sleep(0.5)

def work2():
    while True:
        print("----work2---")
        yield
        time.sleep(0.5)

def main():
    w1 = work1()
    w2 = work2()
    while True:
        next(w1)
        next(w2)

if __name__ == "__main__":
    main()
```

运行结果：

```python
----work1---
----work2---
----work1---
----work2---
----work1---
----work2---
----work1---
----work2---
----work1---
----work2---
----work1---
----work2---
...省略...
```

#### 20.4 协程的实现（greenlet）

为了更好使用协程来完成多任务，python中的greenlet模块对其封装，从而使得切换任务变的更加简单。

greenlet模块需要单独安装。

```python
#coding=utf-8

from greenlet import greenlet
import time

def test1():
    while True:
        print "---A--"
        gr2.switch()
        time.sleep(0.5)

def test2():
    while True:
        print "---B--"
        gr1.switch()
        time.sleep(0.5)

gr1 = greenlet(test1)
gr2 = greenlet(test2)

#切换到gr1中运行
gr1.switch()
```

运行效果：

```
---A--
---B--
---A--
---B--
---A--
---B--
---A--
---B--
...省略...
```

#### 20.5 协程的实现（gevent）

greenlet已经实现了协程，但是这个还的人工切换，是不是觉得太麻烦了，不要捉急，python还有一个比greenlet更强大的并且能够自动切换任务的模块`gevent`

其原理是当一个greenlet遇到IO(指的是input output 输入输出，比如网络、文件操作等)操作时，比如访问网络，就自动切换到其他的greenlet，等到IO操作完成，再在适当的时候切换回来继续执行。

由于IO操作非常耗时，经常使程序处于等待状态，有了gevent为我们自动切换协程，就保证总有greenlet在运行，而不是等待IO。

gevent也需要单独安装。

```python
import gevent

def f(n):
    for i in range(n):
        print(gevent.getcurrent(), i)

g1 = gevent.spawn(f, 5)
g2 = gevent.spawn(f, 5)
g3 = gevent.spawn(f, 5)
g1.join()
g2.join()
g3.join()
```

运行结果：

```
<Greenlet at 0x10e49f550: f(5)> 0
<Greenlet at 0x10e49f550: f(5)> 1
<Greenlet at 0x10e49f550: f(5)> 2
<Greenlet at 0x10e49f550: f(5)> 3
<Greenlet at 0x10e49f550: f(5)> 4
<Greenlet at 0x10e49f910: f(5)> 0
<Greenlet at 0x10e49f910: f(5)> 1
<Greenlet at 0x10e49f910: f(5)> 2
<Greenlet at 0x10e49f910: f(5)> 3
<Greenlet at 0x10e49f910: f(5)> 4
<Greenlet at 0x10e49f4b0: f(5)> 0
<Greenlet at 0x10e49f4b0: f(5)> 1
<Greenlet at 0x10e49f4b0: f(5)> 2
<Greenlet at 0x10e49f4b0: f(5)> 3
<Greenlet at 0x10e49f4b0: f(5)> 4
```

可以看到，3个greenlet是依次运行而不是交替运行

##### 6.7.2 切换执行

```python
import gevent

def f(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        #用来模拟一个耗时操作，注意不是time模块中的sleep
        gevent.sleep(1)  #用来模拟一个耗时操作，注意不是time模块中的sleep

g1 = gevent.spawn(f, 5)
g2 = gevent.spawn(f, 5)
g3 = gevent.spawn(f, 5)
g1.join()
g2.join()
g3.join()
```

运行结果

```python
<Greenlet at 0x7fa70ffa1c30: f(5)> 0
<Greenlet at 0x7fa70ffa1870: f(5)> 0
<Greenlet at 0x7fa70ffa1eb0: f(5)> 0
<Greenlet at 0x7fa70ffa1c30: f(5)> 1
<Greenlet at 0x7fa70ffa1870: f(5)> 1
<Greenlet at 0x7fa70ffa1eb0: f(5)> 1
<Greenlet at 0x7fa70ffa1c30: f(5)> 2
<Greenlet at 0x7fa70ffa1870: f(5)> 2
<Greenlet at 0x7fa70ffa1eb0: f(5)> 2
<Greenlet at 0x7fa70ffa1c30: f(5)> 3
<Greenlet at 0x7fa70ffa1870: f(5)> 3
<Greenlet at 0x7fa70ffa1eb0: f(5)> 3
<Greenlet at 0x7fa70ffa1c30: f(5)> 4
<Greenlet at 0x7fa70ffa1870: f(5)> 4
<Greenlet at 0x7fa70ffa1eb0: f(5)> 4
```

切换执行中sleep不能使用time.sleep，需要使用gevent.sleep。

在gevent中，使用sleep、socket等都要换成gevent的内置函数，效果很差。所以需要打补丁解决。打补丁可以将系统的耗时模块更换为gevent自己实现的模块。

##### 6.7.3  切换执行（打补丁）

```python
from gevent import monkey
import gevent
import random
import time

# 有耗时操作时需要
monkey.patch_all()  # 将程序中用到的耗时操作的代码，换为gevent中自己实现的模块

def coroutine_work(coroutine_name):
    for i in range(10):
        print(coroutine_name, i)
        time.sleep(random.random())  # 打补丁后，可以自动将time.sleep更改为gevent.sleep

gevent.joinall([  # 使用joinall()可以等待列表中的所有协程执行完。
        gevent.spawn(coroutine_work, "work1"),
        gevent.spawn(coroutine_work, "work2")
])
```

运行结果

```python
work1 0
work2 0
work1 1
work1 2
work1 3
work2 1
work1 4
work2 2
work1 5
work2 3
work1 6
work1 7
work1 8
work2 4
work2 5
work1 9
work2 6
work2 7
work2 8
work2 9
```

#### 20.6 进程、线程、协程对比

1. 进程是资源分配的单位
2. 线程是操作系统调度的单位
3. 进程切换需要的资源很最大，效率很低
4. 线程切换需要的资源一般，效率一般（当然了在不考虑GIL的情况下）
5. 协程切换任务资源很小，效率高
6. 多进程、多线程根据cpu核数不一样可能是并行的，但是协程是在一个线程中，所以是并发

### 20.  Tips

1. Python的解释器可以由多个语言来实现，常用的有CPython（C语言实现）、PyPy（Python实现）等。

2. Python中注释使用`#`或者三个单引号```或者三个双引号“”“来实现。

3. 在Python 2中需要使用`#coding=utf-8`或者`#-*- coding:utf-8 -*-`（推荐）来声明使用汉字，否则会报错。

4. 输入使用`input("xxxxx")`（Python3）或者`raw_input("xxxxx")`（Python2）。

5. 输出可以使用`print(“age的值%d%s%d” % (age,name,add))`

   `print("*",end="")打印的时候不换号`。

6. input获取的所有数据都当作字符串类型。

7. `//`是取商，`**`是求次方。

8. 逻辑运算符：and or not。

9. `if...elif...elif...else...`