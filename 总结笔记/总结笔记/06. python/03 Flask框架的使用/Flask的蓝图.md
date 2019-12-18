### 一、蓝图引入目的

为了实现代码的高内聚、低耦合，我们必须进行模块的划分，但是使用Flask时，单纯的划分模块是无法正确的注册路由的，需要我们调用单独的装饰函数进行加载，类似：`app.route("/route")(fun_name)`，需要对每个路由都注册一次，代码繁琐。

这时，Blueprint横空出世。

蓝图：用于实现单个应用的视图、模板、静态文件的集合。蓝图就是模块化处理的类。

简单来说，蓝图就是一个存储操作路由映射方法的容器，主要用来实现客户端请求和URL相互关联的功能。 在Flask中，使用蓝图可以帮助我们实现模块化应用的功能。

#### 1. 蓝图的运行机制：

蓝图是保存了一组将来可以在应用对象上执行的操作。

注册路由就是一种操作,当在程序实例上调用route装饰器注册路由时，这个操作将修改对象的url_map路由映射列表；当我们在蓝图对象上调用route装饰器注册路由时，它只是在内部的一个延迟操作记录列表**defered_functions**中添加了一个项。当执行应用对象的 **register_blueprint()** 方法时，应用对象从蓝图对象的 **defered_functions** 列表中取出每一项，即调用应用对象的 add_url_rule() 方法，这将会修改程序实例的路由映射列表。

#### 2. 蓝图的使用

1. 创建蓝图对象：

   ```python
   # 此时，也可以指定该模块的模板文件等。template_folder
   admin = Blueprint('admin', __name__)
   ```

2. 注册蓝图路由

   ```python
   @admin.route('/')
   def admin_index():
       return 'admin_index'
   ```

3. 在程序实例中注册该蓝图

   ```python
   # url_prefix参数会在路由前添加 /admin
   app.register_blueprint(admin,url_prefix='/admin')
   ```

   