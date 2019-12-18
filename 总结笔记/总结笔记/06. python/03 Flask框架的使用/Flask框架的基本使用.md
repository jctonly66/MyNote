---
typora-copy-images-to: assets
---

## 一、HTTP通信与Web框架

### 1. HTTP通信的流程

1. 客户端将请求打包成HTTP请求报文，采用TCP传输发送给服务器端；
2. 服务器接收到HTTP请求报文后，按照HTTP协议对报文进行解析并根据解析后获知的客户端请求进行逻辑执行，将执行的结果封装成HTTP响应报文，然后通过TCP连接将响应报文发送给客户端；
3. 客户端按照HTTP协议解析响应报文获取结果数据。

> 这里的客户端不一定是浏览器，可以使任何使用HTTP的软件、设备。

### 2. Web框架

对于服务器来说，收到客户端的请求后，要进行路由以及业务逻辑处理。

## 二、Flask框架

Flask是基于Werkzeug工具箱编写的轻量级的Web开发框架，主要面向需求简单的小应用。

Flask的核心是Werkzeug（路由）和jinja2（模板引擎）

### 2.1 Flask扩展包

Flask扩展包包括：

- Flask-SQLalchemy：操作数据库；
- Flask-migrate：管理迁移数据库；
- Flask-Mail：邮件；
- Flask-WTF：表单；
- Flask-script：插入脚本；
- Flask-Login：认证用户状态；
- Flask-RESTful：开发REST API的工具；
- Flask-Bootstrap：集成前端Twitter Bootstrap框架；
- Flask-Moment：本地化日期和时间；

### 2.2 运行框架

```python
# 导入Flask类
from flask import Flask
#Flask类接收一个参数__name__
app = Flask(__name__)

# Flask应用程序实例的run方法启动WEB服务器
if __name__ == '__main__':
    app.run()
```

### 2.2 路由

#### 2.2.1 正常路由

```python
# 装饰器的作用是将路由映射到视图函数index
@app.route('/')
def index():
    return 'Hello World'
```

#### 2.2.2 带有参数的路由

```python
# 路由传递的参数默认当做string处理，这里指定int，尖括号中冒号后面的内容是动态的
@app.route('/user/<int:id>')
def hello_itcast(id):
    return 'hello itcast %d' %id
```

参数也可以是正则表达式，达到限制访问，以及优化访问路径的目的。

```python
from werkzeug.routing import BaseConverter

# 自定义转换器
class Regex_url(BaseConverter):
    def __init__(self,url_map,*args):
        super(Regex_url,self).__init__(url_map)
        self.regex = args[0]     
 
# 注册一个转换器
app.url_map.converters['re'] = Regex_url

@app.route('/user/<re("[a-z]{3}"):id>')
def hello_itcast(id):
    return 'hello %s' %id
```

> 以下纯属猜测：url_map存储路由映射，url_map.converters存储路由的转换规则，只有用route或其他添加路由后才会在map中进行注册。

#### 2.2.3 返回状态码

```python
# return后面可以自主定义状态码(即使这个状态码不存在)。当客户端的请求已经处理完成，由视图函数决定返回给客户端一个状态码，告知客户端这次请求的处理结果。
@app.route('/')
def hello_itcast():
    return 'hello itcast',999
```

#### 2.2.4 异常路由

如果在视图函数执行过程中，出现了异常错误，我们可以使用abort函数立即终止视图函数的执行。使用abort抛出一个HTTP标准中不存在的自定义的状态码，没有实际意义。如果abort函数被触发，其后面的语句将不会执行。其类似于python中raise。

```python
# 通过abort函数，可以向前端返回一个http标准中存在的错误状态码，表示出现的错误信息。
@app.route('/')
def hello_itcast():
    abort(404)
    return 'hello itcast',999

# 在Flask中通过装饰器来实现捕获异常，errorhandler()接收的参数为异常状态码。视图函数的参数，返回的是错误信息。
@app.errorhandler(404)
def error(e):
    return '您请求的页面不存在了，请确认后再次访问！%s'%e
```

#### 2.2.5 重定向

```python
from flask import redirect
@app.route('/')
def hello_itcast():
    return redirect('http://www.itcast.cn')
```

### 2.3 设置和获取cookie

```python
from flask import Flask, make_response
# 响应中设置cookie
@app.route('/cookie')
def set_cookie():
    resp = make_response('this is to set cookie')
    resp.set_cookie('username', 'itcast')
    return resp
```

```python
from flask import Flask,request
# 请求中获取cookie
@app.route('/request')
def resp_cookie():
    resp = request.cookies.get('username')
    return resp
```

### 2.4 Flask上下文

Flask中有两种上下文，请求上下文和应用上下文。

#### 2.4.1 请求上下文（request context）

request和session都属于请求上下文对象。

request：封装了HTTP请求的内容，针对的是HTTP请求。举例：`user = request.args.get('user')`，获取的是get请求的参数。

session：用来记录请求会话中的信息，针对的是用户信息。举例：`session['name'] = user.id`，可以记录用户信息。还可以通过`session.get('name')`获取用户信息。

#### 2.4.2 应用上下文（application context）

current_app和g都属于应用上下文对象。

current_app：表示当前运行程序文件的程序实例。我们可以通过`current_app.name`打印出当前应用程序实例的名字。

g：处理请求时，用于临时存储的对象，每次请求都会重设这个变量。比如：我们可以获取一些临时请求的用户信息。

#### 2.4.3 上下文特性

1. 当调用`app = Flask(__name__)`的时候，创建了程序应用对象app；
2. request 在每次HTTP请求发生时，WSGI server调用`Flask.call()`；然后在Flask内部创建的request对象；
3. app的生命周期大于request和g，一个app存活期间，可能发生多次http请求，所以就会有多个request和g。
4. 最终传入视图函数，通过return、redirect或render_template生成response对象，返回给客户端。

**区别：** 请求上下文：保存了客户端和服务器交互的数据。 应用上下文：在flask程序运行过程中，保存的一些配置信息，比如程序文件名、数据库的连接、用户信息等。

### 2.5 请求钩子

在客户端和服务器交互的过程中，有些准备工作或扫尾工作需要处理，比如：在请求开始时，建立数据库连接；在请求结束时，指定数据的交互格式。为了让每个视图函数避免编写重复功能的代码，Flask提供了通用设施的功能，即请求钩子。

请求钩子是通过装饰器的形式实现，Flask支持如下四种请求钩子：

- before_first_request：在处理第一个请求前运行。
- before_request：在每次请求前运行。
- after_request：如果没有未处理的异常抛出，在每次请求后运行。
- teardown_request：在每次请求后运行，即使有未处理的异常抛出。

### 2.6 Flask装饰器路由

Flask有两大核心：Werkzeug和Jinja2。Werkzeug实现路由、调试和Web服务器网关接口。Jinja2实现了模板。

Werkzeug是一个遵循WSGI协议的python函数库。其内部实现了很多Web框架底层的东西，比如request和response对象；与WSGI规范的兼容；支持Unicode；支持基本的会话管理和签名Cookie；集成URL请求路由等。

Werkzeug库的routing模块负责实现URL解析。不同的URL对应不同的视图函数，routing模块会对请求信息的URL进行解析，匹配到URL对应的视图函数，以此生成一个响应信息。

routing模块内部有Rule类（用来构造不同的URL模式的对象）、Map类（存储所有的URL规则）、MapAdapter类（负责具体URL匹配的工作）；

### 2.7 Flask-Script扩展命令行

通过使用Flask-Script扩展，我们可以在Flask服务器启动的时候，通过命令行的方式传入参数。而不仅仅通过app.run()方法中传参，比如我们可以通过python hello.py runserver --host ip地址，告诉服务器在哪个网络接口监听来自客户端的连接。默认情况下，服务器只监听来自服务器所在计算机发起的连接，即localhost连接。

我们可以通过python hello.py runserver --help来查看参数。

### 2.8 Jinja2模板

Flask提供的render_template函数封装了该模板引擎，render_template函数的第一个参数是模板的文件名，后面的参数都是键值对，表示模板中变量对应的真实值。

#### 2.8.1 变量

在模板中{{ variable }}结构表示变量，是一种特殊的占位符，告诉模板引擎这个位置的值，从渲染模板时使用的数据中获取；Jinja2除了能识别基本类型的变量，还能识别字典；

#### 2.8.2 过滤器

过滤器的本质就是函数。有时候我们不仅仅只是需要输出变量的值，我们还需要修改变量的显示，甚至格式化、运算等等，这就用到了过滤器。 过滤器的使用方式为：变量名 | 过滤器。 

过滤器名写在变量名后面，中间用 | 分隔。如：{{variable | capitalize}}，这个过滤器的作用：把变量variable的值的首字母转换为大写，其他字母转换为小写。 其他常用过滤器如下：

1. 字符串操作

   - `safe`：禁用转义；

     ```
     <p>{{ '<em>hello</em>' | safe }}</p>
     ```

   - capitalize：把变量值的首字母转成大写，其余字母转小写；

     ```
     <p>{{ 'hello' | capitalize }}</p>
     ```

   - lower：把值转成小写；

     ```
     <p>{{ 'HELLO' | lower }}</p>
     ```

   - upper：把值转成大写；

     ```
     <p>{{ 'hello' | upper }}</p>
     ```

   - title：把值中的每个单词的首字母都转成大写；

     ```
     <p>{{ 'hello' | title }}</p>
     ```

   - trim：把值的首尾空格去掉；

     ```
     <p>{{ ' hello world ' | trim }}</p>
     ```

   - reverse:字符串反转；

     ```
     <p>{{ 'olleh' | reverse }}</p>
     ```

   - format:格式化输出；

     ```
     <p>{{ '%s is %d' | format('name',17) }}</p>
     ```

   - striptags：渲染之前把值中所有的HTML标签都删掉；

     ```
     <p>{{ '<em>hello</em>' | striptags }}</p>
     ```

2. 列表操作

   - first：取第一个元素

     ```
     <p>{{ [1,2,3,4,5,6] | first }}</p>
     ```

   - last：取最后一个元素

     ```
     <p>{{ [1,2,3,4,5,6] | last }}</p>
     ```

   - length：获取列表长度

     ```
     <p>{{ [1,2,3,4,5,6] | length }}</p>
     ```

   - sum：列表求和

     ```
     <p>{{ [1,2,3,4,5,6] | sum }}</p>
     ```

   - sort：列表排序

     ```
     <p>{{ [6,2,3,1,5,4] | sort }}</p>
     ```

3. 语句块过滤(不常用)：

   ```
   {% filter upper %}
       this is a Flask Jinja2 introduction
   {% endfilter %}
   ```

4. 自定义过滤器

   过滤器的本质是函数。当模板内置的过滤器不能满足需求，可以自定义过滤器。自定义过滤器有两种实现方式：一种是通过Flask应用对象的add_template_filter方法。还可以通过装饰器来实现自定义过滤器。

   **自定义的过滤器名称如果和内置的过滤器重名，会覆盖内置的过滤器。**

   1. 通过调用应用程序实例的add_template_filter方法实现自定义过滤器。该方法第一个参数是函数名，第二个参数是自定义的过滤器名称。

      ```python
      def filter_double_sort(ls):
          return ls[::2]
      app.add_template_filter(filter_double_sort,'double_2')
      ```

   2. 用装饰器来实现自定义过滤器。装饰器传入的参数是自定义的过滤器名称。

      ```python
      @app.template_filter('db3')
      def filter_double_sort(ls):
          return ls[::-3]
      ```

#### 2.8.3 Web表单

web表单是web应用程序的基本功能。

它是HTML页面中负责数据采集的部件。表单有三个部分组成：表单标签、表单域、表单按钮。表单允许用户输入数据，负责HTML页面数据采集，通过表单将用户输入的数据提交给服务器。

在Flask中，为了处理web表单，我们一般使用Flask-WTF扩展，它封装了WTForms，并且它有验证表单数据的功能。

**WTForms支持的HTML标准字段**

|      字段对象       |                说明                 |
| :-----------------: | :---------------------------------: |
|     StringField     |              文本字段               |
|    TextAreaField    |            多行文本字段             |
|    PasswordField    |            密码文本字段             |
|     HiddenField     |            隐藏文本字段             |
|      DateField      |   文本字段，值为datetime.date格式   |
|    DateTimeField    | 文本字段，值为datetime.datetime格式 |
|    IntegerField     |         文本字段，值为整数          |
|    DecimalField     |    文本字段，值为decimal.Decimal    |
|     FloatField      |        文本字段，值为浮点数         |
|    BooleanField     |       复选框，值为True和False       |
|     RadioField      |             一组单选框              |
|     SelectField     |              下拉列表               |
| SelectMultipleField |       下拉列表，可选择多个值        |
|      FileField      |            文本上传字段             |
|     SubmitField     |            表单提交按钮             |
|      FormField      |    把表单作为字段嵌入另一个表单     |
|      FieldList      |         一组指定类型的字段          |

**WTForms常用验证函数**

|   验证函数   |                   说明                   |
| :----------: | :--------------------------------------: |
| DataRequired |             确保字段中有数据             |
|   EqualTo    | 比较两个字段的值，常用于比较两次密码输入 |
|    Length    |           验证输入的字符串长度           |
| NumberRange  |         验证输入的值在数字范围内         |
|     URL      |                 验证URL                  |
|    AnyOf     |          验证输入值在可选列表中          |
|    NoneOf    |         验证输入值不在可选列表中         |

使用Flask-WTF需要配置参数SECRET_KEY。

CSRF_ENABLED是为了CSRF（跨站请求伪造）保护。 SECRET_KEY用来生成加密令牌，当CSRF激活的时候，该设置会根据设置的密匙生成加密令牌。

#### 2.8.4 使用Flask-WTF实现表单。

配置参数：

```base
 app.config['SECRET_KEY'] = 'silents is gold'
```

模板页面：

```html
 <form method="post">
        #设置csrf_token
        {{ form.csrf_token() }}
        {{ form.us.label }}
        <p>{{ form.us }}</p>
        {{ form.ps.label }}
        <p>{{ form.ps }}</p>
        {{ form.ps2.label }}
        <p>{{ form.ps2 }}</p>
        <p>{{ form.submit() }}</p>
        {% for x in get_flashed_messages() %}
            {{ x }}
        {% endfor %}
 </form>
```

视图函数：

```python
#coding=utf-8
from flask import Flask,render_template,redirect,url_for,session,request,flash

#导入wtf扩展的表单类
from flask_wtf import FlaskForm
#导入自定义表单需要的字段
from wtforms import SubmitField,StringField,PasswordField
#导入wtf扩展提供的表单验证器
from wtforms.validators import DataRequired,EqualTo
app = Flask(__name__)
app.config['SECRET_KEY']='1'

#自定义表单类，文本字段、密码字段、提交按钮
class Login(Form):
    us = StringField(label=u'用户：',validators=[DataRequired()])
    ps = PasswordField(label=u'密码',validators=[DataRequired(),EqualTo('ps2','err')])
    ps2 = PasswordField(label=u'确认密码',validators=[DataRequired()])
    submit = SubmitField(u'提交')

@app.route('/login')
def login():
    return render_template('login.html')

#定义根路由视图函数，生成表单对象，获取表单数据，进行表单数据验证
@app.route('/',methods=['GET','POST'])
def index():
    form = Login()
    if form.validate_on_submit():
        name = form.us.data
        pswd = form.ps.data
        pswd2 = form.ps2.data
        print name,pswd,pswd2
        return redirect(url_for('login'))
    else:
        if request.method=='POST':
            flash(u'信息有误，请重新输入！')
        print form.validate_on_submit()

    return render_template('index.html',form=form)
if __name__ == '__main__':
    app.run(debug=True)
```

#### 2.8.5 宏

类似于python中的函数，宏的作用就是在模板中重复利用代码，避免代码冗余。

Jinja2支持宏，还可以导入宏，需要在多处重复使用的模板代码片段可以写入单独的文件，再包含在所有模板中，以避免重复。

1. 定义宏

   ```python
   {% macro input() %}
     <input type="text"
            name="username"
            value=""
            size="30"/>
   {% endmacro %}
   
   # 带参数的宏
   {% macro input(name,value='',type='text',size=20) %}
       <input type="{{ type }}"
              name="{{ name }}"
              value="{{ value }}"
              size="{{ size }}"/>
   {% endmacro %}
   ```

2. 调用宏

   ```python
   {{ input() }}
   ```

   调用宏，并传递参数

   ```python
   {{ input(value='name',type='password',size=40)}}
   ```

3. 把宏单独抽取出来，封装成html文件，其它模板中导入使用

   ```HTML
   {% macro function() %}
       <input type="text" name="username" placeholde="Username">
       <input type="password" name="password" placeholde="Password">
       <input type="submit">
   {% endmacro %}
   ```

   在其它模板文件中先导入，再调用

   ```
   {% import 'macro.html' as func %}
   {% func.function() %}
   ```

#### 2.8.6 模板继承

模板继承是为了重用模板中的公共内容。一般Web开发中，继承主要使用在网站的顶部菜单、底部。这些内容可以定义在父模板中，子模板直接继承，而不需要重复书写。

`{% block top %}``{% endblock %}`标签定义的内容，相当于在父模板中挖个坑，当子模板继承父模板时，可以进行填充。

子模板使用extends指令声明这个模板继承自哪？父模板中定义的块在子模板中被重新定义，在子模板中调用父模板的内容可以使用super()。

1. 父模板：base.html

   ```
     {% block top %}
       顶部菜单
     {% endblock top %}
   
     {% block content %}
     {% endblock content %}
   
     {% block bottom %}
       底部
     {% endblock bottom %}
   ```

2. 子模板：

   ```
     {% extends 'base.html' %}
     {% block content %}
      需要填充的内容
     {% endblock content %}
   ```

模板继承使用时注意点：

1. 不支持多继承。
2. 为了便于阅读，在子模板中使用extends时，尽量写在模板的第一行。
3. 不能在一个模板文件中定义多个相同名字的block标签。
4. 当在页面中使用多个block标签时，建议给结束标签起个名字，当多个block嵌套时，阅读性更好。

#### 2.8.7 包含(Include)

Jinja2模板中，除了宏和继承，还支持一种代码重用的功能，叫包含(Include)。它的功能是将另一个模板整个加载到当前模板中，并直接渲染。

{\% include 'hello.html' %}

包含在使用时，如果包含的模板文件不存在时，程序会抛出TemplateNotFound异常，可以加上ignore missing关键字。如果包含的模板文件不存在，会忽略这条include语句。

{\% include 'hello.html' ignore missing %}

#### 2.8.8 宏、继承、包含

- 宏(Macro)、继承(Block)、包含(include)均能实现代码的复用。
- 继承(Block)的本质是代码替换，一般用来实现多个页面中重复不变的区域。
- 宏(Macro)的功能类似函数，可以传入参数，需要定义、调用。
- 包含(include)是直接将目标模板文件整个渲染出来。

### 2.9 反向路由

Flask提供了url_for()辅助函数，可以使用程序URL映射中保存的信息生成URL；url_for()接收视图函数名作为参数，返回对应的URL；

### 2.10 Flash

`get_flashed_messages`：返回之前在Flask中通过 flash() 传入的信息列表。把字符串对象表示的消息加入到一个消息队列中，然后通过调用 get_flashed_messages() 方法取出。

```python
{% for message in get_flashed_messages() %}
    {{ message }}
{% endfor %}
```



