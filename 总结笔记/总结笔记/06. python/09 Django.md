---
typora-copy-images-to: assets
---

## 一、Django使用

### 1. 创建项目

我们以一个电商网站为例，网站上有跟用户有关的页面，有跟商品有关的页面，还有跟订单有关的页面，这样的一块内容其实就是网站的一个功能模块。

在django中，项目的组织结构为**一个项目包含多个应用，一个应用对应一个业务模块**。

```bash
django-admin startproject # 项目名称
```

创建项目后，会自动生成一些文件，含义如下：

![1563355459185](assets\1563355459185.png)

- manage.py是项目管理文件，通过它管理项目。
- 与项目同名的目录，此处为test1。
- `__init__.py`是一个空文件，作用是这个目录test1可以被当作包使用。
- settings.py是项目的整体配置文件。
- urls.py是项目的URL配置文件。
- wsgi.py是项目与WSGI兼容的Web服务器入口，详细内容会在布署中讲到。

### 2. 创建应用

使用一个应用开发一个业务模块。

```bash
python manage.py startapp app_name
```

![1563355583662](assets\1563355583662.png)

- `__init.py__`是一个空文件，表示当前目录booktest可以当作一个python包使用。
- tests.py文件用于开发测试用例，在实际开发中会有专门的测试人员，这个事情不需要我们来做。
- models.py文件跟数据库操作相关。
- views.py文件跟接收浏览器请求，进行处理，返回页面相关。
- admin.py文件跟网站的后台管理相关。
- migrations文件夹之后给大家介绍。

### 3. 关联应用

应用创建成功后，需要关联才可以使用，也就是建立应用和项目之间的关联，在test1/settings.py中INSTALLED_APPS下添加应用的名称就可以完成安装。

初始项目的INSTALLED_APPS如下图：

![1563355779730](assets\1563355779730.png)

### 4. 开发测试服务器

在开发阶段，为了能够快速预览到开发的效果，django提供了一个纯python编写的轻量级web服务器，仅在开发阶段使用。

运行服务器命令如下：

```
python manage.py runserver ip:端口
例：
python manage.py runserver
```

如果增加、修改、删除文件，服务器会自动重启。

### 5. 设计模型

使用django进行数据库开发的步骤如下：

1. 在models.py中定义模型类
2. 迁移
3. 通过类和对象完成数据增删改查操作

#### 5.1 定义模型类

模型类定义在models.py文件中，继承自models.Model类。

注：不需要定义主键列，在生成时会自动添加，并且值为自动增长。

示例：

```python
from django.db import models

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateField()
    
    # 如果有外键
    hbook = models.ForeignKey('BookInfo')
```

#### 5.2 迁移

迁移由两步完成：

1. 生成迁移文件：根据模型类生成创建表的迁移文件。 `python manage.py makemigrations`

   执行生成迁移文件命令后，会在应用booktest目录下的migrations目录中生成迁移文件。

2. 执行迁移：根据第一步生成的迁移文件在数据库中创建表。`python manage.py migrate`

   当执行迁移命令后，Django框架会读取迁移文件自动帮我们在数据库中生成对应的表格。

#### 5.3 默认数据库

Django默认采用sqlite3数据库，文件中的db.sqlite3就是Django框架帮我们自动生成的数据库文件。 sqlite3是一个很小的数据库，通常用在手机中，它跟mysql一样，我们也可以通过sql语句来操作它。

##### 5.3.1 更改默认数据库

更改setting文件中的DATABASES。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'test2', #数据库名字，
        'USER': 'root', #数据库登录用户名
        'PASSWORD': 'mysql', #数据库登录密码
        'HOST': 'localhost', #数据库所在主机
        'PORT': '3306', #数据库端口
    }
}
```

然后在应用的`__init__`文件中将数据库切换成MySQL

```python 
import pymysql
pymysql.install_as_MySQLdb()
```

#### 5.4 数据操作

完成数据表的迁移之后，下面就可以通过进入项目的shell，进行简单的API操作。如果需要退出项目，可以使用ctrl+d快捷键或输入quit()。

```python
python manage.py shell	# 进入项目shell的命令

from booktest.models import BookInfo,HeroInfo  # 首先引入models中的类
BookInfo.objects.all()  # 查询所有图书信息

# 插入实体
b=BookInfo()
b.btitle="射雕英雄传"
from datetime import date
b.bpub_date=date(1991,1,31)
b.save()

# 查找实体
b=BookInfo.objects.get(id=1)

# 修改实体
b.bpub_date=date(2017,1,1)
b.save()

# 删除实体
b.delete()

# 获得关联集合：返回当前实体被关联的所有实体，其中Book是Hero的外键
b.heroinfo_set.all()
```

默认值并不在数据库层面生效，而是在django创建对象时生效。

#### 5.5 后台管理

Django能够根据定义的模型类自动地生成管理页面。使用Django的管理模块，需要按照如下步骤操作：

1. 管理界面本地化
2. 创建管理员
3. 注册模型类
4. 自定义管理页面

##### 5.5.1 管理界面本地化

本地化是将显示的语言、时间等使用本地的习惯，这里的本地化就是进行中国化，中国大陆地区使用简体中文，时区使用亚洲/上海时区，注意这里不使用北京时区表示。

打开settings.py文件，找到语言编码、时区的设置项，将内容改为如下：

```python
LANGUAGE_CODE = 'zh-hans' #使用中国语言
TIME_ZONE = 'Asia/Shanghai' #使用中国上海时间
```

##### 5.5.2 创建管理员

创建管理员的命令如下，按提示输入用户名、邮箱、密码。

```
python manage.py createsuperuser
```

接下来启动服务器。

```
python manage.py runserver
```

打开浏览器，在地址栏中输入如下地址后回车。

```
http://127.0.0.1:8000/admin/
```

##### 5.5.3 注册模型类

登录后台管理后，默认没有我们创建的应用中定义的模型类，需要在自己应用中的admin.py文件中注册，才可以在后台管理中看到，并进行增删改查操作。

打开booktest/admin.py文件，编写如下代码：

```python
from django.contrib import admin
from booktest.models import BookInfo,HeroInfo

admin.site.register(BookInfo)
admin.site.register(HeroInfo)
```

到浏览器中刷新页面，可以看到模型类BookInfo和HeroInfo的管理了。

##### 5.5.4 自定义管理页面

在列表页只显示出了BookInfo object，对象的其它属性并没有列出来，查看非常不方便。 Django提供了自定义管理页面的功能，比如列表页要显示哪些值。

打开admin.py文件，自定义类，继承自admin.ModelAdmin类。

```python
class BookInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'btitle', 'bpub_date']  # 属性list_display表示要显示哪些属性
   
admin.site.register(BookInfo, BookInfoAdmin)  # 修改模型类BookInfo的注册代码
```

### 6. 视图

对于django的设计框架MVT，用户在URL中请求的是视图，视图接收请求后进行处理，并将处理的结果返回给请求者。

使用视图时需要进行两步操作：

1. 定义视图函数
2. 配置URLconf

#### 6.1 定义视图

视图就是一个Python函数，被定义在views.py中。

视图的必须有一个参数，一般叫request，视图必须返回HttpResponse对象，HttpResponse中的参数内容会显示在浏览器的页面上。

示例：

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("index")
```

#### 6.2 配置URLconf

##### 6.2.1 查找视图的过程

请求者在浏览器地址栏中输入url，请求到网站后，获取url信息，然后与编写好的URLconf逐条匹配，如果匹配成功则调用对应的视图函数，如果所有的URLconf都没有匹配成功，则返回404错误。

一条URLconf包括url规则、视图两部分；其中，url规则使用正则表达式定义，视图就是在views.py中定义的视图函数。

对于URLconf配置需要两步完成：

1. 在应用中定义URLconf

   在应用下创建urls.py文件，定义代码如下：

   ```python
   from django.conf.urls import url
   from booktest import views
   urlpatterns = [
       url(r'^严格匹配开头结尾$', views.index),
   ]
   ```

2. 包含到项目的URLconf中

   打开test1/urls.py文件，为urlpatterns列表增加项如下：

   ```python
   url(r'^', include('booktest.urls')),
   ```

### 7. 模板

在Django中，将前端的内容定义在模板中，然后再把模板交给视图调用，各种漂亮、炫酷的效果就出现了。

#### 7.1 创建模板

在项目目录下创建模板文件夹 templates 及应用文件夹 booktest，然后创建模板，比如index.html。

![1563502300885](assets\1563502300885.png)

设置查找模板的路径：打开test1/settings.py文件，设置TEMPLATES的DIRS值

```
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```

#### 7.2 定义模板

示例：

```html
<html>
<head>
    <title>图书列表</title>
</head>
<body>
<h1>{{title}}</h1>
{%for i in list%}
{{i}}<br>
{%endfor%}
</body>
</html>
```

在模板中输出变量语法如下，变量可能是从视图中传递过来的，也可能是在模板中定义的。

```
{{变量名}}
```

在模板中编写代码段语法如下：

```
{%代码段%}
```

#### 7.3 视图调用模板

Django提供了一个函数render。 方法render包含3个参数：

1. 第一个参数为request对象
2. 第二个参数为模板文件路径
3. 第三个参数为字典，表示向模板中传递的上下文数据

打开booktst/views.py文件，调用render的代码如下：

```python
from django.shortcuts import render

def index(request):
    context={'title':'图书列表','list':range(10)}
    return render(request,'booktest/index.html',context)
```