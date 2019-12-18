---
typora-copy-images-to: assets
---

## 一、设计模式

设计模式是前人工作的总结和提炼，通常，被人们广泛流传的设计模式都是针对某一特定问题的成熟的解决方案。

使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。

### 1. 单例

#### 1.1 目的

让类创建的对象，在系统中只有唯一的一个实例，每一次执行`类名()`返回的对象，内存地址是相同的。

#### 1.2 实现

在Python中，单例的实现方式：

1. 定义一个 类属性，初始值是 `None`，用于记录单例对象的引用
2. 重写 `__new__` 方法
3. 如果类属性 `is None`，调用父类方法分配空间，并在类属性中记录结果
4. 返回类属性中记录的对象引用

```python
class MusicPlayer(object):
    # 定义类属性记录单例对象引用
    instance = None
    # 记录是否执行过初始化动作
    init_flag = false
    
    def __new__(cls, *args, **kwargs):
        # 1. 判断类属性是否已经被赋值
        if cls.instance is None:
            cls.instance = super().__new__(cls)
        # 2. 返回类属性的单例引用
        return cls.instance
    
    def __init__(self):
        if not MusicPlayer.init_flag:
            MusicPlayer.init_flag = True
```



### 17. eval函数

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

