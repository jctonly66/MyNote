### 一、Flask组成

Flask集成了Werkzeug以及Jinja，其中Werkzeug实现了WGSI。

### 二、Pipenv使用

Pipenv：用来更好地管理Python虚拟环境以及依赖包。

#### 1. 常用命令

```python
pipenv insall  # 为当前项目创建虚拟环境，并且安装基本的包以及Pipfile文件中列出的依赖包
pipenv shell  # 显式地激活虚拟环境
pipenv run python hello.py  # （非虚拟环境）执行命令
pipenv graph  # 查看当前环境下的依赖情况
pip list  # 查看依赖列表
pipenv update flask  # 更新依赖包
pipenv --venv  # 查看项目对应的虚拟环境路径
```

#### 2. 要点

Pipenv的管理依赖两个文件：Pipfile和Pipfile.lock文件。前者用来记录项目依赖包列表，后者记录了固定版本的详细依赖包列表。

在创建虚拟环境时，如果项目根目录下没有Pipfile文件，pipenv install命令会在项目文件夹根目录下创建.Pipfile和Pipfile.lock文件，当使用Pipenv安装/删除/更新依赖包时，Pipfile以及Pipfile.lock会自动更新。

开发依赖在使用pipenv install命令时需要添加额外的 --dev选项才会安装dev-packages部分定义的开发依赖包。

### 三、Flask使用

#### 1. 创建Flask实例

```python
from flask import Flask
app = Flask(__name__)  # __name__代表本模块
```

#### 2. Flask路由

1. 一个视图函数可以绑定多个URL

   ```python
   @app.route('/hi')
   @app.route('/hello')
   def say_hello():
       return '<h1>Hello, Flask!</h1>'
   ```

2. 动态URL

   ```python
   @app.route('/greet', defaults={'name': 'Programmer'})  # 变量添加默认值
   @app.route('/greet/<name>')  # <name>就是变量名的形式
   def greet(name):
       return '<h1>Hello, %s!</h1>' % name
   ```

### 四、命令行交互界面 CLI

CLI：Command Line Interface，命令行交互界面。

Flask通过依赖包Click内置CLI，安装Flask后，环境会自动添加一个flask命令脚本，我们可以通过flask命令执行内置命令、扩展提供的命令或是我们自己定义的命令。

#### 1. 常用flask命令

```python
flask run  # 启动内置的开发服务器
flask --help  # 查看所有可用的命令
flask run --host=0.0.0.0  # 使服务器对外可见
flask run --port=80000  # 使服务器对外可见
```

### 五、Flask环境变量

Flask依靠环境变量对程序的功能、数据进行配置。

#### 1. 常用环境变量

Flask内置的命令，都可以通过配置环境变量定义默认选项值，格式为：`FLASK_<COMMAND>_<OPTION>`

- FLASK_ENV：用来配置服务器是生产环境还是测试环境。

  ```python
  FLASK_ENV=development  # 配置为开发环境，此时调试模式（Debug Mode）将被开启，程序会自动激活Werkzeug内置的调试器（debugger）和重载器（reloader），它们会为开发带来很大的帮助。
  FLASK_ENV=production  # 配置为生产环境
  ```

  调试器：用来调试代码，错误追踪信息等。

  重载器：监测文件变动，然后重新启动开发服务器。默认会使用Werkzeug内置的stat重载器，但是它的缺点是耗电较严重，而且准确性一般。我们一般安装另一个用于检测文件变动的Python库Watchdog，安装后Werkzeug会自动使用它来监测文件变动。

- FLASK_RUN_HOST：配置服务器host

- FLASK_RUN_PORT：配置服务器短裤

**以上配置需要结合Python-dotenv使用。**

#### 2. Python-dotenv使用

Python-dotenv：进行环境变量以及程序数据的的管理。它会生成两个文件：.flaskenv以及.env。.flaskenv用来存储和flask相关的公开环境变量，而.env用来存储包含敏感信息的环境变量

一般来说，在执行flask run命令运行程序前，我们需要提供程序实例所在模块的位置。我们可以通过Flask run直接运行程序，是因为Flask会自动探测程序实例，自动探测存在下面的规则：

1. 从当前目录寻找app.py和wsgi.py模块，并从中寻找名为app或application的程序实例；
2. 从环境变量FLASK_APP对应的值寻找名为app或application的程序实例；
3. 如果安装了python-dotenv，那么在使用flask run或其他命令时会使用它自动从.flaskenv文件和.env文件中加载环境变量。

当安装了python-dotenv时，Flask在加载环境变量的优先级是：手动设置的环境变量 > .env中设置的环境变量 > .flaskenv设置的环境变量

#### 3. 环境变量的配置

3.1 临时配置

在Flask环境下通过命令配置：

```
set FLASK_APP=xxxx  # Windows下设置
export FLASK_APP=xxxx  # Linux下设置
```

3.2 持久配置，通过python-dotenv在.flaskenv文件中以健值对的形式配置。

```
VAR = 1
```

### 六、Git的gitignore提交参考

Python项目的.gitgnore模板可以参考 https://github.com/github/gitignore/blob/master/Python.gitignore。

使用PyCharm编写程序时会产生一些配置文件，这些配置文件保存在 .idea 目录下，关于这些文件的忽略设置可以参考 https://www.gitignore.io/api/pycharm。

