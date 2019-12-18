## Django数据库查询

Django主要通过**过滤器**查询数据库数据：

1. 返回查询集的过滤器如下：
   1. all()：返回所有数据。
   2. filter()：返回满足条件的数据。
   3. exclude()：返回满足条件之外的数据，相当于sql语句中where部分的not关键字。
   4. order_by()：对结果进行排序。
2. 返回单个值的过滤器如下：
   1. get()：返回单个满足条件的对象，如果未找到会引发"模型类.DoesNotExist"异常。如果多条被返回，会引发"模型类.MultipleObjectsReturned"异常。
   2. count()：返回当前查询结果的总条数。
   3. aggregate()：聚合，返回一个字典。

判断某一个查询集中是否有数据：exists()：判断查询集中是否有数据，如果有则返回True，没有则返回False。

格式：

```python
属性名称__比较运算符=值  # 通过"属性名_id"表示外键对应对象的id值
```

### 1. 基本操作

```python
list=模型.objects.过滤器(属性__比较运算符=1)  
"""
过滤器包括：filter、exclude、get
比较运算符包括：
	1. exact：表示判等
	2. contains：是否包含
	3. startswith、endswith：开头或结尾
	4. isnull：是否为null。
	5. in：是否包含在范围内
	6. gt、gte、lt、lte：大于、大于等于、小于、小于等于
	7. year、month、day、week_day、hour、minute、second：对日期时间类型的属性进行查询
"""
```

### 2. 比较两个属性

使用F对象对两个属性进行比较。

```
F(属性名)
```

```
from django.db.models import F
...
list = BookInfo.objects.filter(bread__gte=F('bcomment'))
```

可以在F对象上使用算数运算。

### 3. 与或非

多个过滤器逐个调用表示逻辑与关系，同sql语句中where部分的and关键字。

```
list=BookInfo.objects.filter(bread__gt=20,id__lt=3)
或
list=BookInfo.objects.filter(bread__gt=20).filter(id__lt=3)
```

如果需要实现逻辑或or的查询，需要使用Q()对象结合|运算符，Q对象被义在django.db.models中。

```
Q(属性名__运算符=值)
```

Q对象可以使用&、|连接，&表示逻辑与，|表示逻辑或。

例：查询阅读量大于20，或编号小于3的图书，只能使用Q对象实现

```
list = BookInfo.objects.filter(Q(bread__gt=20) | Q(pk__lt=3))
```

Q对象前可以使用~操作符，表示非not。

例：查询编号不等于3的图书。

```
list = BookInfo.objects.filter(~Q(pk=3))
```

### 4. 聚合函数

使用aggregate()过滤器调用聚合函数。聚合函数包括：Avg，Count，Max，Min，Sum，被定义在django.db.models中。

例：查询图书的总阅读量。

```
from django.db.models import Sum
...
list = BookInfo.objects.aggregate(Sum('bread'))
```

注意aggregate的返回值是一个字典类型，格式如下：

```
  {'聚合类小写__属性名':值}
  如:{'sum__bread':3}
```

使用count时一般不使用aggregate()过滤器。

例：查询图书总数。

```
list = BookInfo.objects.count()
```

注意count函数的返回值是一个数字。

### 5. 两大特性

- 惰性执行：创建查询集不会访问数据库，直到调用数据时，才会访问数据库，调用数据的情况包括迭代、序列化、与if合用。
- 缓存：使用**同一个查询集**，第一次使用时会发生数据库的查询，然后把结果缓存下来，再次使用这个查询集时会使用缓存的数据。

### 6. 限制查询集

可以对查询集进行取下标或切片操作，等同于sql中的limit和offset子句。**注意：不支持负数索引。**

对查询集进行切片后返回一个新的查询集，不会立即执行查询。

如果获取一个对象，直接使用[0]，等同于[0:1].get()，但是如果没有数据，[0]引发IndexError异常，[0:1].get()如果没有数据引发DoesNotExist异常。

示例：获取第1、2项，运行查看。

```
list=BookInfo.objects.all()[0:2]
```





