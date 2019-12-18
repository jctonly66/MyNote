一、Python高级

### 1. GIL

描述Python GIL的概念， 以及它对python多线程的影响？编写一个多线程抓取网页的程序，并阐明多线程抓取程序是否可比单线程性能有提升，并解释原因。

GIL其实对多进程没有影响

Guido的声明：<http://www.artima.com/forums/flat.jsp?forum=106&thread=214235>

he language doesn't require the GIL -- it's only the CPython virtual machine that has historically been unable to shed it.

答：

1. Python语言和GIL没有半毛钱关系。仅仅是由于历史原因在Cpython虚拟机(解释器)，难以移除GIL。
2. GIL：全局解释器锁。每个线程在执行的过程都需要先获取GIL，保证同一时刻只有一个线程可以执行代码。
3. 线程释放GIL锁的情况： 在IO操作等可能会引起阻塞的system call之前,可以暂时释放GIL,但在执行完毕后,必须重新获取GIL。Python 3.x使用计时器（执行时间达到阈值后，当前线程释放GIL）或Python 2.x，tickets计数达到100
4. Python使用多进程是可以利用多核的CPU资源的。
5. 多线程爬取比单线程性能有提升，因为遇到IO阻塞会自动释放GIL锁

总结：对于IO密集型的程序，可以使用协程，因为IO阻塞可以释放GIL锁，对于计算密集型的程序，最好使用进程，线程不能发挥多核CPU的威力。

### 3. 私有化

在Python中，使用变量的命名不同来区分不同的属性、方法。

1. `xx`: 公有变量
2. `_x`: 单前置下划线,私有化属性或方法，`from somemodule import *`禁止导入，类对象和子类可以访问
3. `__xx`：双前置下划线，私有，避免与子类中的属性命名冲突，无法在外部直接访问(名字重整所以访问不到)，子类不能继承，不能访问。
4. `__xx__`:双前后下划线,用户名字空间的魔法对象或属性。例如:`__init__` ，不要自己发明这样的名字。
5. `xx_`:单后置下划线,用于避免与Python关键词的冲突

通过name mangling（名字重整(目的就是以防子类意外重写基类的方法或者属性)如：_Class__object）机制就可以访问private了。

```python
#coding=utf-8

class Person(object):
    def __init__(self, name, age, taste):
        self.name = name
        self._age = age 
        self.__taste = taste

    def showperson(self):  # 公有
        print(self.name)
        print(self._age)
        print(self.__taste)

    def dowork(self):  # 公有
        self._work()
        self.__away()


    def _work(self):  # 私有，不可导入，本模块可用
        print('my _work')

    def __away(self):  # 私有，本模块可用
        print('my __away')

class Student(Person):
    def construction(self, name, age, taste):
        self.name = name
        self._age = age 
        self.__taste = taste

    def showstudent(self):
        print(self.name)
        print(self._age)
        print(self.__taste)

    @staticmethod
    def testbug():
        _Bug.showbug()

# 模块内可以访问，当from  cur_module import *时，不导入
class _Bug(object):
    @staticmethod
    def showbug():
        print("showbug")

s1 = Student('jack', 25, 'football')
s1.showperson()
print('*'*20)

# 无法访问__taste,导致报错
# s1.showstudent() 
s1.construction('rose', 30, 'basketball')
s1.showperson()
print('*'*20)

s1.showstudent()
print('*'*20)

Student.testbug()
```

#### 3.1 TIPS

1. 父类中属性名为`__名字`的，子类不继承，子类不能访问
2. 如果在子类中向`__名字`赋值，那么会在子类中定义的一个与父类相同名字的属性
3. `_名`的变量、函数、类在使用`from xxx import *`时都不会被导入







### 6. 静态方法、类方法/属性

类属性在内存中只保存一份，实例属性在每个对象中都要保存一份。

实例方法、静态方法和类方法在内存中都归属于类，区别在于调用方式不同。它们根据装饰符来区分。

- 实例方法：由对象调用；至少一个self参数；执行实例方法时，自动将调用该方法的对象赋值给self；
- 类方法：由类调用； 至少一个cls参数；执行类方法时，自动将调用该方法的类赋值给cls；
- 静态方法：由类调用；无默认参数；

实例对象只能调用实例方法和类方法和静态方法。类对象只能调用类方法和静态方法。

### 7. property属性

property属性是一种用起来像是使用的实例属性一样的特殊属性，可以对应于某个方法。

```python
# ############### 定义 ###############
class Foo:
    def func(self):
        pass

    # 定义property属性
    @property
    def prop(self):
        pass

# ############### 调用 ###############
foo_obj = Foo()
foo_obj.func()  # 调用实例方法
foo_obj.prop  # 调用property属性
```

注意：

1. 定义时，在实例方法的基础上添加 @property 装饰器；并且仅有一个self参数
2. 调用时，无需括号

```python
  方法：foo_obj.func()
  property属性：foo_obj.prop
```

Python的property属性的功能是：property属性内部进行一系列的逻辑计算，最终将计算结果返回。

#### 7.1 property属性的两种方式

1. 装饰器 即：在方法上应用装饰器
2. 类属性 即：在类中定义值为property对象的类属性

##### 7.1.1 装饰器方式

在类的实例方法上应用@property装饰器

Python中的类有`经典类`和`新式类`，`新式类`的属性比`经典类`的属性丰富。（ 如果类继object，那么该类是新式类 ）

###### 1）经典类，具有一种@property装饰器

```python
# ############### 定义 ###############    
class Goods:
    @property
    def price(self):
        return "laowang"
# ############### 调用 ###############
obj = Goods()
result = obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
print(result)
```

###### 2）新式类，具有三种@property装饰器

```python
#coding=utf-8
# ############### 定义 ###############
class Goods:
    """python3中默认继承object类
        以python2、3执行此程序的结果不同，因为只有在python3中才有@xxx.setter  @xxx.deleter
    """
    @property
    def price(self):
        print('@property')

    @price.setter
    def price(self, value):
        print('@price.setter')

    @price.deleter
    def price(self):
        print('@price.deleter')

# ############### 调用 ###############
obj = Goods()
obj.price          # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
obj.price = 123    # 自动执行 @price.setter 修饰的 price 方法，并将  123 赋值给方法的参数
del obj.price      # 自动执行 @price.deleter 修饰的 price 方法
```

经典类中的属性只有一种访问方式，其对应被 @property 修饰的方法，新式类中的属性有三种访问方式，并分别对应了三个被@property、@方法名.setter、@方法名.deleter修饰的方法

由于新式类中具有三种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

```python
class Goods(object):

    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    @price.setter
    def price(self, value):
        self.original_price = value

    @price.deleter
    def price(self):
        del self.original_price

obj = Goods()
obj.price         # 获取商品价格
obj.price = 200   # 修改商品原价
del obj.price     # 删除商品原价
```

##### 7.1.2 类属性方式

类属性方式可以创建值为property对象的类属性，当使用类属性的方式创建property属性时，`经典类`和`新式类`无区别。

```python
class Foo:
    def get_bar(self):
        return 'laowang'

    BAR = property(get_bar)

obj = Foo()
reuslt = obj.BAR  # 自动调用get_bar方法，并获取方法的返回值
print(reuslt)
```

property方法中有个四个参数

- 第一个参数是方法名，调用 `对象.属性` 时自动触发执行方法
- 第二个参数是方法名，调用 `对象.属性 ＝ XXX` 时自动触发执行方法
- 第三个参数是方法名，调用 `del 对象.属性` 时自动触发执行方法
- 第四个参数是字符串，调用 `对象.属性.__doc__` ，此参数是该属性的描述信息

```python
#coding=utf-8
class Foo(object):
    def get_bar(self):
        print("getter...")
        return 'laowang'

    def set_bar(self, value): 
        """必须两个参数"""
        print("setter...")
        return 'set value' + value

    def del_bar(self):
        print("deleter...")
        return 'laowang'

    BAR = property(get_bar, set_bar, del_bar, "description...")

obj = Foo()

obj.BAR  # 自动调用第一个参数中定义的方法：get_bar
obj.BAR = "alex"  # 自动调用第二个参数中定义的方法：set_bar方法，并将“alex”当作参数传入
desc = Foo.BAR.__doc__  # 自动获取第四个参数中设置的值：description...
print(desc)
del obj.BAR  # 自动调用第三个参数中定义的方法：del_bar方法
```

由于`类属性方式`创建property属性具有3种访问方式，我们可以根据它们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除

WEB框架 Django 的视图中 request.POST 就是使用的类属性的方式创建的属性。

#### 7.2 property属性的作用

通过使用property属性，能够简化调用者在获取数据的流程。

##### 7.2.1 私有属性添加getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
```

这是JAVA中常用的方法，但是在Python中不推荐使用。可以使用property来升级。

##### 7.2.2 使用property升级getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    def getMoney(self):
        return self.__money

    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")

    # 定义一个属性，当对这个money设置值时调用setMoney,当获取值时调用getMoney
    money = property(getMoney, setMoney)  

a = Money()
a.money = 100  # 调用setMoney方法
print(a.money)  # 调用getMoney方法
#100
```

##### 7.2.3 使用property取代getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0

    # 使用装饰器对money进行装饰，那么会自动添加一个叫money的属性，当调用获取money的值时，调用装饰的方法
    @property
    def money(self):
        return self.__money

    # 使用装饰器对money进行装饰，当对money设置值时，调用装饰的方法
    @money.setter
    def money(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")

a = Money()
a.money = 100
print(a.money)
```

### 8. 魔法属性

无论人或事物往往都有不按套路出牌的情况，Python的类属性也是如此，存在着一些具有特殊含义的属性，详情如下：

#### 8.1 `__doc__`

用来表示类的描述信息。

```python
class Foo:
    """ 描述类信息，这是用于看片的神奇 """
    def func(self):
        pass

print(Foo.__doc__)
#输出：类的描述信息
```

#### 8.2 `__module__ `和` __class__`

`__module__ `表示当前操作的对象在那个模块；`__class__` 表示当前操作的对象的类是什么

```python
test.py
# -*- coding:utf-8 -*-
class Person(object):
    def __init__(self):
        self.name = 'laowang'
        
main.py
from test import Person

obj = Person()
print(obj.__module__)  # 输出 test 即：输出模块
print(obj.__class__)  # 输出 test.Person 即：输出类
```

#### 8.3 `__init__`

初始化方法，通过类创建对象时，自动触发执行

```python
class Person:
    def __init__(self, name):
        self.name = name
        self.age = 18


obj = Person('laowang')  # 自动执行类中的 __init__ 方法
```

#### 8.4 `__del__`

当对象在内存中被释放时，自动触发执行。此方法一般无须定义，因为Python是一门高级语言，程序员在使用时无需关心内存的分配和释放，因为此工作都是交给Python解释器来执行，所以，`__del__`的调用是由解释器在进行垃圾回收时自动触发执行的。

```python
class Foo:
    def __del__(self):
        pass
```

#### 8.5 `__call__`

对象后面加括号，触发执行。`__init__`方法的执行是由创建对象触发的，即：`对象 = 类名()` ；而对于 `__call__` 方法的执行是由对象后加括号触发的，即：`对象()` 或者 `类()()`

```python
class Foo:
    def __init__(self):
        pass

    def __call__(self, *args, **kwargs):
        print('__call__')


obj = Foo()  # 执行 __init__
obj()  # 执行 __call__
```

#### 8.6 `__dict__`

类或对象中的所有属性，类的实例属性属于对象；类中的类属性和方法等属于类，即：

```python
class Province(object):
    country = 'China'

    def __init__(self, name, count):
        self.name = name
        self.count = count

    def func(self, *args, **kwargs):
        print('func')

# 获取类的属性，即：类属性、方法、
print(Province.__dict__)
# 输出：{'__dict__': <attribute '__dict__' of 'Province' objects>, '__module__': '__main__', 'country': 'China', '__doc__': None, '__weakref__': <attribute '__weakref__' of 'Province' objects>, 'func': <function Province.func at 0x101897950>, '__init__': <function Province.__init__ at 0x1018978c8>}

obj1 = Province('山东', 10000)
print(obj1.__dict__)
# 获取 对象obj1 的属性
# 输出：{'count': 10000, 'name': '山东'}

obj2 = Province('山西', 20000)
print(obj2.__dict__)
# 获取 对象obj1 的属性
# 输出：{'count': 20000, 'name': '山西'}
```

#### 8.7 `__str__`

如果一个类中定义了__str__方法，那么在打印 对象 时，默认输出该方法的返回值。

```python
class Foo:
    def __str__(self):
        return 'laowang'

obj = Foo()
print(obj)
# 输出：laowang
```

#### 8.8 `__getitem__`、`__setitem__`、`__delitem__`

用于索引操作，如字典。以上分别表示获取、设置、删除数据

```python
# -*- coding:utf-8 -*-

class Foo(object):

    def __getitem__(self, key):
        print('__getitem__', key)

    def __setitem__(self, key, value):
        print('__setitem__', key, value)

    def __delitem__(self, key):
        print('__delitem__', key)


obj = Foo()

result = obj['k1']      # 自动触发执行 __getitem__
obj['k2'] = 'laowang'   # 自动触发执行 __setitem__
del obj['k1']           # 自动触发执行 __delitem__
```

#### 8.9 `__getslice__`、`__setslice__`、`__delslice__`

该三个方法用于分片操作，如：列表

```python
# -*- coding:utf-8 -*-

class Foo(object):

    def __getslice__(self, i, j):
        print('__getslice__', i, j)

    def __setslice__(self, i, j, sequence):
        print('__setslice__', i, j)

    def __delslice__(self, i, j):
        print('__delslice__', i, j)

obj = Foo()

obj[-1:1]                   # 自动触发执行 __getslice__
obj[0:1] = [11,22,33,44]    # 自动触发执行 __setslice__
del obj[0:2]                # 自动触发执行 __delslice__
```

### 9. with与上下文管理器

对于系统资源如文件、数据库连接、socket 而言，应用程序打开这些资源并执行完业务逻辑之后，必须做的一件事就是要关闭（断开）该资源。

比如 Python 程序打开一个文件，往文件中写内容，写完之后，就要关闭该文件，否则会出现什么情况呢？极端情况下会出现 "Too many open files" 的错误，因为系统允许你打开的最大文件数量是有限的。

同样，对于数据库，如果连接数过多而没有及时关闭的话，就可能会出现 "Can not connect to MySQL server Too many connections"，因为数据库连接是一种非常昂贵的资源，不可能无限制的被创建。

来看看如何正确关闭一个文件。

普通版：

```python
def m1():
    f = open("output.txt", "w")
    f.write("python之禅")
    f.close()
# 这样写有一个潜在的问题，如果在调用 write 的过程中，出现了异常进而导致后续代码无法继续执行，close 方法无法被正常调用，因此资源就会一直被该程序占用者释放。那么该如何改进代码呢？
```

进阶版：

```python
def m2():
    f = open("output.txt", "w")
    try:
        f.write("python之禅")
    except IOError:
        print("oops error")
    finally:
        f.close()
# 改良版本的程序是对可能发生异常的代码处进行 try 捕获，使用 try/finally 语句，该语句表示如果在 try 代码块中程序出现了异常，后续代码就不再执行，而直接跳转到 except 代码块。而无论如何，finally 块的代码最终都会被执行。因此，只要把 close 放在 finally 代码中，文件就一定会关闭。
```

高级版：

```python
def m3():
    with open("output.txt", "r") as f:
        f.write("Python之禅")
# 一种更加简洁、优雅的方式就是用 with 关键字。open 方法的返回值赋值给变量 f，当离开 with 代码块的时候，系统会自动调用 f.close() 方法， with 的作用和使用 try/finally 语句是一样的。那么它的实现原理是什么？在讲 with 的原理前要涉及到另外一个概念，就是上下文管理器（Context Manager）。
```

#### 9.1 上下文

上下文在不同的地方表示不同的含义，要感性理解。context其实说白了，和文章的上下文是一个意思，在通俗一点，我觉得叫环境更好。....

林冲大叫一声“啊也！”....

问:这句话林冲的“啊也”表达了林冲怎样的心里？

答:啊你妈个头啊！

看，一篇文章，给你摘录一段，没前没后，你读不懂，因为有语境，就是语言环境存在，一段话说了什么，要通过上下文(文章的上下文)来推断。

app点击一个按钮进入一个新的界面，也要保存你是在哪个屏幕跳过来的等等信息，以便你点击返回的时候能正确跳回，如果不存肯定就无法正确跳回了。

看这些都是上下文的典型例子，理解成环境就可以，(而且上下文虽然叫上下文，但是程序里面一般都只有上文而已，只是叫的好听叫上下文。。进程中断在操作系统中是有上有下的，不过不这个高深的问题就不要深究了。。。)

#### 9.2 上下文管理器

任何实现了` __enter__()` 和 `__exit__() `方法的对象都可称之为上下文管理器，上下文管理器对象可以使用 with 关键字。显然，文件（file）对象也实现了上下文管理器。

那么文件对象是如何实现这两个方法的呢？我们可以模拟实现一个自己的文件类，让该类实现 `__enter__()` 和 `__exit__()` 方法。

```python
class File():

    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        print("entering")
        self.f = open(self.filename, self.mode)
        return self.f

    def __exit__(self, *args):
        print("will exit")
        self.f.close()
```

`__enter__() `方法返回资源对象，这里就是你将要打开的那个文件对象，`__exit__()` 方法处理一些清除工作。

因为 File 类实现了上下文管理器，现在就可以使用 with 语句了。

```python
with File('out.txt', 'w') as f:
    print("writing")
    f.write('hello, python')
```

这样，你就无需显示地调用 close 方法了，由系统自动去调用，哪怕中间遇到异常 close 方法也会被调用。

#### 9.3 使用装饰符实现上下文

Python 还提供了一个 contextmanager 的装饰器，更进一步简化了上下文管理器的实现方式。通过 yield 将函数分割成两部分，yield 之前的语句在 `__enter__ `方法中执行，yield 之后的语句在 `__exit__ `方法中执行。紧跟在 yield 后面的值是函数的返回值。

```python
from contextlib import contextmanager

@contextmanager
def my_open(path, mode):
    f = open(path, mode)
    yield f
    f.close()
```

调用

```
with my_open('out.txt', 'w') as f:
    f.write("hello , the simplest context manager")
```

Python 提供了 with 语法用于简化资源操作的后续清除操作，是 try/finally 的替代方法，实现原理建立在上下文管理器之上。此外，Python 还提供了一个 contextmanager 装饰器，更进一步简化上下管理器的实现方式。

### 10. 元类

#### 10.1 类也是对象

在大多数编程语言中，类就是一组用来描述如何生成一个对象的代码段。在Python中这一点仍然成立：

```python
>>> class ObjectCreator(object):
…       pass
…
>>> my_object = ObjectCreator()
>>> print(my_object)
<__main__.ObjectCreator object at 0x8974f2c>
```

但是，Python中的类还远不止如此。类同样也是一种对象。是的，没错，就是对象。只要你使用关键字class，Python解释器在执行的时候就会创建一个对象。

下面的代码段：

```python
>>> class ObjectCreator(object):
…       pass
…
```

将在内存中创建一个对象，名字就是ObjectCreator。这个对象（类对象ObjectCreator）拥有创建对象（实例对象）的能力。但是，它的本质仍然是一个对象，于是乎你可以对它做如下的操作：

1. 你可以将它赋值给一个变量
2. 你可以拷贝它
3. 你可以为它增加属性
4. 你可以将它作为函数参数进行传递

下面是示例：

```python
>>> print(ObjectCreator)  # 你可以打印一个类，因为它其实也是一个对象
<class '__main__.ObjectCreator'>
>>> def echo(o):
…       print(o)
…
>>> echo(ObjectCreator)  # 你可以将类做为参数传给函数
<class '__main__.ObjectCreator'>
>>> print(hasattr(ObjectCreator, 'new_attribute'))
Fasle
>>> ObjectCreator.new_attribute = 'foo'  # 你可以为类增加属性
>>> print(hasattr(ObjectCreator, 'new_attribute'))
True
>>> print(ObjectCreator.new_attribute)
foo
>>> ObjectCreatorMirror = ObjectCreator  # 你可以将类赋值给一个变量
>>> print(ObjectCreatorMirror())
<__main__.ObjectCreator object at 0x8997b4c>
```

#### 10.2 动态地创建类

因为类也是对象，你可以在运行时动态的创建它们，就像其他任何对象一样。首先，你可以在函数中创建类，使用class关键字即可。

```python
>>> def choose_class(name):
…       if name == 'foo':
…           class Foo(object):
…               pass
…           return Foo     # 返回的是类，不是类的实例
…       else:
…           class Bar(object):
…               pass
…           return Bar
…
>>> MyClass = choose_class('foo')
>>> print(MyClass)  # 函数返回的是类，不是类的实例
<class '__main__'.Foo>
>>> print(MyClass())  # 你可以通过这个类创建类实例，也就是对象
<__main__.Foo object at 0x89c6d4c>
```

但这还不够动态，因为你仍然需要自己编写整个类的代码。由于类也是对象，所以它们必须是通过什么东西来生成的才对。

当你使用class关键字时，Python解释器自动创建这个对象。但就和Python中的大多数事情一样，Python仍然提供给你手动处理的方法。

还记得内建函数type吗？这个古老但强大的函数能够让你知道一个对象的类型是什么，就像这样：

```python
>>> print(type(1))  # 数值的类型
<type 'int'>
>>> print(type("1"))  # 字符串的类型
<type 'str'>
>>> print(type(ObjectCreator()))  # 实例对象的类型
<class '__main__.ObjectCreator'>
>>> print(type(ObjectCreator))  # 类的类型
<type 'type'>
```

仔细观察上面的运行结果，发现使用type对ObjectCreator查看类型是，答案为type， 是不是有些惊讶。。。看下面

#### 10.3 使用type动态创建类

type还有一种完全不同的功能，动态的创建类。

type可以接受一个类的描述作为参数，然后返回一个类。（要知道，根据传入参数的不同，同一个函数拥有两种完全不同的用法是一件很傻的事情，但这在Python中是为了保持向后兼容性）

type可以像这样工作：

`type(类名, 由父类名称组成的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)`

比如下面的代码：

```python
In [2]: class Test: #定义了一个Test类
   ...:     pass
   ...:
In [3]: Test() # 创建了一个Test类的实例对象
Out[3]: <__main__.Test at 0x10d3f8438>
```

可以手动像这样创建：

```python
Test2 = type("Test2", (), {}) # 定了一个Test2类
In [5]: Test2() # 创建了一个Test2类的实例对象
Out[5]: <__main__.Test2 at 0x10d406b38>
```

我们使用"Test2"作为类名，并且也可以把它当做一个变量来作为类的引用。类和变量是不同的，这里没有任何理由把事情弄的复杂。即type函数中第1个实参，也可以叫做其他的名字，这个名字表示类的名字

```python
In [23]: MyDogClass = type('MyDog', (), {})

In [24]: print(MyDogClass)
<class '__main__.MyDog'>
```

使用help来测试这2个类

```python
In [10]: help(Test) # 用help查看Test类

Help on class Test in module __main__:

class Test(builtins.object)
 |  Data descriptors defined here:
 |
 |  __dict__
 |      dictionary for instance variables (if defined)
 |
 |  __weakref__
 |      list of weak references to the object (if defined)
In [8]: help(Test2) #用help查看Test2类

Help on class Test2 in module __main__:

class Test2(builtins.object)
 |  Data descriptors defined here:
 |
 |  __dict__
 |      dictionary for instance variables (if defined)
 |
 |  __weakref__
 |      list of weak references to the object (if defined)
```

#### 10.4 使用type创建带有属性的类

type 接受一个字典来为类定义属性，因此

```python
>>> Foo = type('Foo', (), {'bar': True})
```

可以翻译为：

```python
>>> class Foo(object):
…       bar = True
```

并且可以将Foo当成一个普通的类一样使用：

```python
>>> print(Foo)
<class '__main__.Foo'>
>>> print(Foo.bar)
True
>>> f = Foo()
>>> print(f)
<__main__.Foo object at 0x8a9b84c>
>>> print(f.bar)
True
```

当然，你可以继承这个类，代码如下：

```python
>>> class FooChild(Foo):
…       pass
```

就可以写成：

```python
>>> FooChild = type('FooChild', (Foo,), {})
>>> print(FooChild)
<class '__main__.FooChild'>
>>> print(FooChild.bar)  # bar属性是由Foo继承而来
True
```

注意：

- type的第2个参数，元组中是父类的名字，而不是字符串
- 添加的属性是类属性，并不是实例属性

#### 10.5 使用type创建带有方法的类

最终你会希望为你的类增加方法。只需要定义一个有着恰当签名的函数并将其作为属性赋值就可以了。

##### 10.5.1 添加实例方法

```python
In [46]: def echo_bar(self):  # 定义了一个普通的函数
    ...:     print(self.bar)
    ...:

In [47]: FooChild = type('FooChild', (Foo,), {'echo_bar': echo_bar})  # 让FooChild类中的echo_bar属性，指向了上面定义的函数

In [48]: hasattr(Foo, 'echo_bar')  # 判断Foo类中 是否有echo_bar这个属性
Out[48]: False

In [49]: hasattr(FooChild, 'echo_bar')  # 判断FooChild类中 是否有echo_bar这个属性
Out[49]: True

In [50]: my_foo = FooChild()

In [51]: my_foo.echo_bar()
True
```

##### 10.5.2 添加静态方法

```python
In [36]: @staticmethod
    ...: def test_static():
    ...:     print("static method ....")
    ...:

In [37]: Foochild = type('Foochild', (Foo,), {"echo_bar": echo_bar, "test_static": test_static})

In [38]: fooclid = Foochild()

In [39]: fooclid.test_static
Out[39]: <function __main__.test_static>

In [40]: fooclid.test_static()
static method ....

In [41]: fooclid.echo_bar()
True
```

##### 10.5.3 添加类方法

```python
In [42]: @classmethod
    ...: def test_class(cls):
    ...:     print(cls.bar)
    ...:

In [43]:

In [43]: Foochild = type('Foochild', (Foo,), {"echo_bar":echo_bar, "test_static": test_static, "test_class": test_class})

In [44]:

In [44]: fooclid = Foochild()

In [45]: fooclid.test_class()
True
```

你可以看到，在Python中，类也是对象，你可以动态的创建类。这就是当你使用关键字class时Python在幕后做的事情，而这就是通过元类来实现的。

较为完整的使用type创建类的方式:

```python
class A(object):
    num = 100

def print_b(self):
    print(self.num)

@staticmethod
def print_static():
    print("----haha-----")

@classmethod
def print_class(cls):
    print(cls.num)

B = type("B", (A,), {"print_b": print_b, "print_static": print_static, "print_class": print_class})
b = B()
b.print_b()
b.print_static()
b.print_class()
# 结果
# 100
# ----haha-----
# 100
```

####  10.6 到底什么是元类

元类就是用来创建类的“东西”。你创建类就是为了创建类的实例对象，不是吗？但是我们已经学习到了Python中的类也是对象。

元类就是用来创建这些类（对象）的，元类就是类的类，你可以这样理解为：

```python
MyClass = MetaClass() # 使用元类创建出一个对象，这个对象称为“类”
my_object = MyClass() # 使用“类”来创建出实例对象
```

你已经看到了type可以让你像这样做：

```python
MyClass = type('MyClass', (), {})
```

这是因为函数type实际上是一个元类。type就是Python在背后用来创建所有类的元类。现在你想知道那为什么type会全部采用小写形式而不是Type呢？好吧，我猜这是为了和str保持一致性，str是用来创建字符串对象的类，而int是用来创建整数对象的类。type就是创建类对象的类。你可以通过检查`__class__`属性来看到这一点。Python中所有的东西，注意，我是指所有的东西——都是对象。这包括整数、字符串、函数以及类。它们全部都是对象，而且它们都是从一个类创建而来，这个类就是type。

```python
>>> age = 35
>>> age.__class__
<type 'int'>
>>>
>>> name = 'bob'
>>> name.__class__
<type 'str'>
>>>
>>> def foo(): pass
>>>foo.__class__
<type 'function'>
>>>
>>> class Bar(object): pass
>>> b = Bar()
>>> b.__class__
<class '__main__.Bar'>
>>>
```

现在，对于任何一个`__class__`的`__class__`属性又是什么呢？

```python
>>> a.__class__.__class__
<type 'type'>
>>> age.__class__.__class__
<type 'type'>
>>> foo.__class__.__class__
<type 'type'>
>>> b.__class__.__class__
<type 'type'>
```

因此，元类就是创建类这种对象的东西。type就是Python的内建元类，当然了，你也可以创建自己的元类。

#### 10.7 `__metaclass__`属性

你可以在定义一个类的时候为其添加`__metaclass__`属性。

```python
class Foo(object):
    __metaclass__ = something…
    ...省略...
```

如果你这么做了，Python就会用元类来创建类Foo。小心点，这里面有些技巧。你首先写下class Foo(object)，但是类Foo还没有在内存中创建。Python会在类的定义中寻找`__metaclass__`属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。把下面这段话反复读几次。当你写如下代码时 :

```python
class Foo(Bar):
    pass
```

Python做了如下的操作：

1. Foo中有`__metaclass__`这个属性吗？如果是，Python会通过`__metaclass__`创建一个名字为Foo的类(对象)
2. 如果Python没有找到`__metaclass__`，它会继续在Bar（父类）中寻找`__metaclass__`属性，并尝试做和前面同样的操作。
3. 如果Python在任何父类中都找不到`__metaclass__`，它就会在模块层次中去寻找`__metaclass__`，并尝试做同样的操作。
4. 如果还是找不到`__metaclass__`,Python就会用内置的type来创建这个类对象。

现在的问题就是，你可以在`__metaclass__`中放置些什么代码呢？答案就是：可以创建一个类的东西。那么什么可以用来创建一个类呢？type，或者任何使用到type或者子类化type的东东都可以。

#### 10.8 自定义元类

元类的主要目的就是为了当创建类时能够自动地改变类。

假想一个很傻的例子，你决定在你的模块里所有的类的属性都应该是大写形式。有好几种方法可以办到，但其中一种就是通过在模块级别设定`__metaclass__`。采用这种方法，这个模块中的所有类都会通过这个元类来创建，我们只需要告诉元类把所有的属性都改成大写形式就万事大吉了。

幸运的是，`__metaclass__`实际上可以被任意调用，它并不需要是一个正式的类。所以，我们这里就先以一个简单的函数作为例子开始。

python2中

```python
#-*- coding:utf-8 -*-
def upper_attr(class_name, class_parents, class_attr):

    # class_name 会保存类的名字 Foo
    # class_parents 会保存类的父类 object
    # class_attr 会以字典的方式保存所有的类属性

    # 遍历属性字典，把不是__开头的属性名字变为大写
    new_attr = {}
    for name, value in class_attr.items():
        if not name.startswith("__"):
            new_attr[name.upper()] = value

    # 调用type来创建一个类
    return type(class_name, class_parents, new_attr)

class Foo(object):
    __metaclass__ = upper_attr # 设置Foo类的元类为upper_attr
    bar = 'bip'

print(hasattr(Foo, 'bar'))
print(hasattr(Foo, 'BAR'))

f = Foo()
print(f.BAR)
```

python3中

```python
#-*- coding:utf-8 -*-
def upper_attr(class_name, class_parents, class_attr):

    #遍历属性字典，把不是__开头的属性名字变为大写
    new_attr = {}
    for name,value in class_attr.items():
        if not name.startswith("__"):
            new_attr[name.upper()] = value

    #调用type来创建一个类
    return type(class_name, class_parents, new_attr)

class Foo(object, metaclass=upper_attr):
    bar = 'bip'

print(hasattr(Foo, 'bar'))
print(hasattr(Foo, 'BAR'))

f = Foo()
print(f.BAR)
```

现在让我们再做一次，这一次用一个真正的class来当做元类。

```python
#coding=utf-8

class UpperAttrMetaClass(type):
    # __new__ 是在__init__之前被调用的特殊方法
    # __new__是用来创建对象并返回之的方法
    # 而__init__只是用来将传入的参数初始化给对象
    # 你很少用到__new__，除非你希望能够控制对象的创建
    # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
    # 如果你希望的话，你也可以在__init__中做些事情
    # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
    def __new__(cls, class_name, class_parents, class_attr):
        # 遍历属性字典，把不是__开头的属性名字变为大写
        new_attr = {}
        for name, value in class_attr.items():
            if not name.startswith("__"):
                new_attr[name.upper()] = value

        # 方法1：通过'type'来做类对象的创建
        return type(class_name, class_parents, new_attr)

        # 方法2：复用type.__new__方法
        # 这就是基本的OOP编程，没什么魔法
        # return type.__new__(cls, class_name, class_parents, new_attr)

# python3的用法
class Foo(object, metaclass=UpperAttrMetaClass):
    bar = 'bip'

# python2的用法
# class Foo(object):
#     __metaclass__ = UpperAttrMetaClass
#     bar = 'bip'


print(hasattr(Foo, 'bar'))
# 输出: False
print(hasattr(Foo, 'BAR'))
# 输出:True

f = Foo()
print(f.BAR)
# 输出:'bip'
```

就是这样，除此之外，关于元类真的没有别的可说的了。但就元类本身而言，它们其实是很简单的：

1. 拦截类的创建
2. 修改类
3. 返回修改之后的类

#### 10.9 为什么要使用元类

现在回到我们的大主题上来，究竟是为什么你会去使用这样一种容易出错且晦涩的特性？好吧，一般来说，你根本就用不上它：

“元类就是深度的魔法，99%的用户应该根本不必为此操心。如果你想搞清楚究竟是否需要用到元类，那么你就不需要它。那些实际用到元类的人都非常清楚地知道他们需要做什么，而且根本不需要解释为什么要用元类。” —— Python界的领袖 Tim Peters

#### 10.10 元类实现ORM

##### 10.10.1 ORM是什么

ORM 是 python编程语言后端web框架 Django的核心思想，“Object Relational Mapping”，即对象-关系映射，简称ORM。

一个句话理解就是：创建一个实例对象，用创建它的类名当做数据表名，用创建它的类属性对应数据表的字段，当对这个实例对象操作时，能够对应MySQL语句

```python
class User(父类省略):
    uid = ('uid', "int unsigned")
    name = ('username', "varchar(30)")
    email = ('email', "varchar(30)")
    password = ('password', "varchar(30)")
    ...省略...


u = User(uid=12345, name='Michael', email='test@orm.org', password='my-pwd')
u.save()
# 对应如下sql语句
# insert into User (username,email,password,uid)
# values ('Michael','test@orm.org','my-pwd',12345)
```

说明

1. 所谓的ORM就是让开发者在操作数据库的时候，能够像操作对象时通过`xxxx.属性=yyyy`一样简单，这是开发ORM的初衷
2. 只不过ORM的实现较为复杂，Django中已经实现了很复杂的操作，本节知识主要通过完成一个 insert相类似的ORM，理解其中的道理就就可以了

##### 10.10.2 通过元类简单实现ORM中的insert功能

```python
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        mappings = dict()
        # 判断是否需要保存
        for k, v in attrs.items():
            # 判断是否是指定的StringField或者IntegerField的实例对象
            if isinstance(v, tuple):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v

        # 删除这些已经在字典中存储的属性
        for k in mappings.keys():
            attrs.pop(k)

        # 将之前的uid/name/email/password以及对应的对象引用、类名字
        attrs['__mappings__'] = mappings  # 保存属性和列的映射关系
        attrs['__table__'] = name  # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)


class User(metaclass=ModelMetaclass):
    uid = ('uid', "int unsigned")
    name = ('username', "varchar(30)")
    email = ('email', "varchar(30)")
    password = ('password', "varchar(30)")
    # 当指定元类之后，以上的类属性将不在类中，而是在__mappings__属性指定的字典中存储
    # 以上User类中有 
    # __mappings__ = {
    #     "uid": ('uid', "int unsigned")
    #     "name": ('username', "varchar(30)")
    #     "email": ('email', "varchar(30)")
    #     "password": ('password', "varchar(30)")
    # }
    # __table__ = "User"
    def __init__(self, **kwargs):
        for name, value in kwargs.items():
            setattr(self, name, value)

    def save(self):
        fields = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v[0])
            args.append(getattr(self, k, None))

        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join([str(i) for i in args]))
        print('SQL: %s' % sql)


u = User(uid=12345, name='Michael', email='test@orm.org', password='my-pwd')
# print(u.__dict__)
u.save()
```

执行的效果:

```python
Found mapping: password ==> ('password', 'varchar(30)')
Found mapping: email ==> ('email', 'varchar(30)')
Found mapping: uid ==> ('uid', 'int unsigned')
Found mapping: name ==> ('username', 'varchar(30)')
SQL: insert into User (uid,password,username,email) values (12345,my-pwd,Michael,test@orm.org)
```

##### 10.10.3 完善对数据类型的检测

```python
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        mappings = dict()
        # 判断是否需要保存
        for k, v in attrs.items():
            # 判断是否是指定的StringField或者IntegerField的实例对象
            if isinstance(v, tuple):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v

        # 删除这些已经在字典中存储的属性
        for k in mappings.keys():
            attrs.pop(k)

        # 将之前的uid/name/email/password以及对应的对象引用、类名字
        attrs['__mappings__'] = mappings  # 保存属性和列的映射关系
        attrs['__table__'] = name  # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)


class User(metaclass=ModelMetaclass):
    uid = ('uid', "int unsigned")
    name = ('username', "varchar(30)")
    email = ('email', "varchar(30)")
    password = ('password', "varchar(30)")
    # 当指定元类之后，以上的类属性将不在类中，而是在__mappings__属性指定的字典中存储
    # 以上User类中有 
    # __mappings__ = {
    #     "uid": ('uid', "int unsigned")
    #     "name": ('username', "varchar(30)")
    #     "email": ('email', "varchar(30)")
    #     "password": ('password', "varchar(30)")
    # }
    # __table__ = "User"
    def __init__(self, **kwargs):
        for name, value in kwargs.items():
            setattr(self, name, value)

    def save(self):
        fields = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v[0])
            args.append(getattr(self, k, None))

        args_temp = list()
        for temp in args:
            # 判断入如果是数字类型
            if isinstance(temp, int):
                args_temp.append(str(temp))
            elif isinstance(temp, str):
                args_temp.append("""'%s'""" % temp)
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(args_temp))
        print('SQL: %s' % sql)


u = User(uid=12345, name='Michael', email='test@orm.org', password='my-pwd')
# print(u.__dict__)
u.save()
```

运行效果如下:

```python
Found mapping: uid ==> ('uid', 'int unsigned')
Found mapping: password ==> ('password', 'varchar(30)')
Found mapping: name ==> ('username', 'varchar(30)')
Found mapping: email ==> ('email', 'varchar(30)')
SQL: insert into User (email,uid,password,username) values ('test@orm.org',12345,'my-pwd','Michael')
```

##### 10.10.4 抽取到基类中

```python
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        mappings = dict()
        # 判断是否需要保存
        for k, v in attrs.items():
            # 判断是否是指定的StringField或者IntegerField的实例对象
            if isinstance(v, tuple):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v

        # 删除这些已经在字典中存储的属性
        for k in mappings.keys():
            attrs.pop(k)

        # 将之前的uid/name/email/password以及对应的对象引用、类名字
        attrs['__mappings__'] = mappings  # 保存属性和列的映射关系
        attrs['__table__'] = name  # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)


class Model(object, metaclass=ModelMetaclass):
    def __init__(self, **kwargs):
        for name, value in kwargs.items():
            setattr(self, name, value)

    def save(self):
        fields = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v[0])
            args.append(getattr(self, k, None))

        args_temp = list()
        for temp in args:
            # 判断入如果是数字类型
            if isinstance(temp, int):
                args_temp.append(str(temp))
            elif isinstance(temp, str):
                args_temp.append("""'%s'""" % temp)
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(args_temp))
        print('SQL: %s' % sql)


class User(Model):
    uid = ('uid', "int unsigned")
    name = ('username', "varchar(30)")
    email = ('email', "varchar(30)")
    password = ('password', "varchar(30)")


u = User(uid=12345, name='Michael', email='test@orm.org', password='my-pwd')
# print(u.__dict__)
u.save()
```

