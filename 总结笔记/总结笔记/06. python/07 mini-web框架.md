## 一、mini-web框架

### 1. 服务器动态资源请求

服务器中，可以根据请求来读取静态页面或者代码生成的动态数据。

但是，正常情况下，Web服务器应该与代码进行分离。 Web服务器如何如代码进行分离然而又互相关联呢？ 肯定不能再Web服务器中导入业务代码吧。如果Web服务器与业务代码遵循相同的协议，就可以了。

![Snip20161117_8](07 mini-web框架.assets/Snip20161117_8.png)

### 2. WSGI

怎么在你刚建立的Web服务器上运行一个`Django应用`和`Flask应用`，如何不做任何改变而适应不同的web架构呢？

在以前，选择 `Python web 架构`会受制于可用的`web服务器`，反之亦然。如果架构和服务器可以协同工作，那就好了：

但有可能面对（或者曾有过）下面的问题，当要把一个服务器和一个架构结合起来时，却发现他们不是被设计成协同工作的：

那么，怎么可以不修改服务器和架构代码而确保可以在多个架构下运行web服务器呢？答案就是 Python Web Server Gateway Interface (或简称 WSGI，读作“wizgy”)。

WSGI允许开发者将选择web框架和web服务器分开。可以混合匹配web服务器和web框架，选择一个适合的配对。比如,可以在Gunicorn 或者 Nginx/uWSGI 或者 Waitress上运行 Django, Flask, 或 Pyramid。真正的混合匹配，得益于WSGI同时支持服务器和架构：

web服务器必须具备WSGI接口，所有的现代Python Web框架都已具备WSGI接口，它让你不对代码作修改就能使服务器和特点的web框架协同工作。

WSGI由web服务器支持，而web框架允许你选择适合自己的配对，但它同样对于服务器和框架开发者提供便利使他们可以专注于自己偏爱的领域和专长而不至于相互牵制。其他语言也有类似接口：java有Servlet API，Ruby 有 Rack。

#### 2.1 定义WSGI接口

WSGI接口定义非常简单，它只要求Web开发者实现一个函数，就可以响应HTTP请求。我们来看一个最简单的Web版本的“Hello World!”：

```python
def application(environ, start_response):
    # 返回响应头
    start_response('200 OK', [('Content-Type', 'text/html')])
    # 返回响应Body
    return 'Hello World!'
```

上面的`application()`函数就是符合WSGI标准的一个HTTP处理函数，它接收两个参数：

- environ：一个包含所有HTTP请求信息的dict对象；
- start_response：一个发送HTTP响应的函数。

整个`application()`函数本身没有涉及到任何解析HTTP的部分，也就是说，把底层web服务器解析部分和应用程序逻辑部分进行了分离，这样开发者就可以专心做一个领域了

不过，等等，这个`application()`函数怎么调用？如果我们自己调用，两个参数environ和start_response我们没法提供，返回的str也没法发给浏览器。

所以`application()`函数必须由WSGI服务器来调用。有很多符合WSGI规范的服务器。而我们此时的web服务器项目的目的就是做一个既能解析静态网页还可以解析动态网页的服务器

#### 2.2 WSGI传递的字典

web服务器-----WSGI协议---->web框架 

```python
{
    'HTTP_ACCEPT_LANGUAGE': 'zh-cn',
    'wsgi.file_wrapper': <built-infunctionuwsgi_sendfile>,
    'HTTP_UPGRADE_INSECURE_REQUESTS': '1',
    'uwsgi.version': b'2.0.15',
    'REMOTE_ADDR': '172.16.7.1',
    'wsgi.errors': <_io.TextIOWrappername=2mode='w'encoding='UTF-8'>,
    'wsgi.version': (1,0),
    'REMOTE_PORT': '40432',
    'REQUEST_URI': '/',
    'SERVER_PORT': '8000',
    'wsgi.multithread': False,
    'HTTP_ACCEPT': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'HTTP_HOST': '172.16.7.152: 8000',
    'wsgi.run_once': False,
    'wsgi.input': <uwsgi._Inputobjectat0x7f7faecdc9c0>,
    'SERVER_PROTOCOL': 'HTTP/1.1',
    'REQUEST_METHOD': 'GET',
    'HTTP_ACCEPT_ENCODING': 'gzip,deflate',
    'HTTP_CONNECTION': 'keep-alive',
    'uwsgi.node': b'ubuntu',
    'HTTP_DNT': '1',
    'UWSGI_ROUTER': 'http',
    'SCRIPT_NAME': '',
    'wsgi.multiprocess': False,
    'QUERY_STRING': '',
    'PATH_INFO': '/index.html',
    'wsgi.url_scheme': 'http',
    'HTTP_USER_AGENT': 'Mozilla/5.0(Macintosh;IntelMacOSX10_12_5)AppleWebKit/603.2.4(KHTML,likeGecko)Version/10.1.1Safari/603.2.4',
    'SERVER_NAME': 'ubuntu'
}
```

### 3. 自定义服务器框架

```python
import time
import os

template_root = "./templates"


def index(file_name):
    """返回index.py需要的页面内容"""
    # return "hahha" + os.getcwd()  # for test 路径问题
    try:
        file_name = file_name.replace(".py", ".html")
        f = open(template_root + file_name)
    except Exception as ret:
        return "%s" % ret
    else:
        content = f.read()
        f.close()
        return content


def center(file_name):
    """返回center.py需要的页面内容"""
    # return "hahha" + os.getcwd()  # for test 路径问题
    try:
        file_name = file_name.replace(".py", ".html")
        f = open(template_root + file_name)
    except Exception as ret:
        return "%s" % ret
    else:
        content = f.read()
        f.close()
        return content

def application(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-Type', 'text/html')]
    start_response(status, response_headers)

    file_name = environ['PATH_INFO']
    if file_name == "/index.py":
        return index(file_name)
    elif file_name == "/center.py":
        return center(file_name)
    else:
        return str(environ) + '==Hello world from a simple WSGI application!--->%s\n' % time.ctime()
```

web_server.py 

```python
import select
import time
import socket
import sys
import re
import multiprocessing


class WSGIServer(object):
    """定义一个WSGI服务器的类"""

    def __init__(self, port, documents_root, app):

        # 1. 创建套接字
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # 2. 绑定本地信息
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind(("", port))
        # 3. 变为监听套接字
        self.server_socket.listen(128)

        # 设定资源文件的路径
        self.documents_root = documents_root

        # 设定web框架可以调用的函数(对象)
        self.app = app

    def run_forever(self):
        """运行服务器"""

        # 等待对方链接
        while True:
            new_socket, new_addr = self.server_socket.accept()
            # 创建一个新的进程来完成这个客户端的请求任务
            new_socket.settimeout(3)  # 3s
            new_process = multiprocessing.Process(target=self.deal_with_request, args=(new_socket,))
            new_process.start()
            new_socket.close()

    def deal_with_request(self, client_socket):
        """以长链接的方式，为这个浏览器服务器"""

        while True:
            try:
                request = client_socket.recv(1024).decode("utf-8")
            except Exception as ret:
                print("========>", ret)
                client_socket.close()
                return

            # 判断浏览器是否关闭
            if not request:
                client_socket.close()
                return

            request_lines = request.splitlines()
            for i, line in enumerate(request_lines):
                print(i, line)

            # 提取请求的文件(index.html)
            # GET /a/b/c/d/e/index.html HTTP/1.1
            ret = re.match(r"([^/]*)([^ ]+)", request_lines[0])
            if ret:
                print("正则提取数据:", ret.group(1))
                print("正则提取数据:", ret.group(2))
                file_name = ret.group(2)
                if file_name == "/":
                    file_name = "/index.html"

            # 如果不是以py结尾的文件，认为是普通的文件
            if not file_name.endswith(".py"):

                # 读取文件数据
                try:
                    print(self.documents_root+file_name)
                    f = open(self.documents_root+file_name, "rb")
                except:
                    response_body = "file not found, 请输入正确的url"

                    response_header = "HTTP/1.1 404 not found\r\n"
                    response_header += "Content-Type: text/html; charset=utf-8\r\n"
                    response_header += "Content-Length: %d\r\n" % (len(response_body))
                    response_header += "\r\n"

                    response = response_header + response_body

                    # 将header返回给浏览器
                    client_socket.send(response.encode('utf-8'))

                else:
                    content = f.read()
                    f.close()

                    response_body = content

                    response_header = "HTTP/1.1 200 OK\r\n"
                    response_header += "Content-Length: %d\r\n" % (len(response_body))
                    response_header += "\r\n"

                    # 将header返回给浏览器
                    client_socket.send(response_header.encode('utf-8') + response_body)

            # 以.py结尾的文件，就认为是浏览需要动态的页面
            else:
                # 准备一个字典，里面存放需要传递给web框架的数据
                env = dict()
                # ----------更新---------
                env['PATH_INFO'] = file_name  # 例如 index.py

                # 存web返回的数据
                response_body = self.app(env, self.set_response_headers)

                # 合并header和body
                response_header = "HTTP/1.1 {status}\r\n".format(status=self.headers[0])
                response_header += "Content-Type: text/html; charset=utf-8\r\n"
                response_header += "Content-Length: %d\r\n" % len(response_body.encode("utf-8"))
                for temp_head in self.headers[1]:
                    response_header += "{0}:{1}\r\n".format(*temp_head)

                response = response_header + "\r\n"
                response += response_body

                client_socket.send(response.encode('utf-8'))


    def set_response_headers(self, status, headers):
        """这个方法，会在 web框架中被默认调用"""
        response_header_default = [
            ("Data", time.time()),
            ("Server", "ItCast-python mini web server")
        ]

        # 将状态码/相应头信息存储起来
        # [字符串, [xxxxx, xxx2]]
        self.headers = [status, response_header_default + headers]


# 设置静态资源访问的路径
g_static_document_root = "./static"
# 设置动态资源访问的路径
g_dynamic_document_root = "./dynamic"

def main():
    """控制web服务器整体"""
    # python3 xxxx.py 7890
    if len(sys.argv) == 3:
        # 获取web服务器的port
        port = sys.argv[1]
        if port.isdigit():
            port = int(port)
        # 获取web服务器需要动态资源时，访问的web框架名字
        web_frame_module_app_name = sys.argv[2]
    else:
        print("运行方式如: python3 xxx.py 7890 my_web_frame_name:app")
        return

    print("http服务器使用的port:%s" % port)

    # 将动态路径即存放py文件的路径，添加到path中，这样python就能够找到这个路径了
    sys.path.append(g_dynamic_document_root)

    ret = re.match(r"([^:]*):(.*)", web_frame_module_app_name)
    if ret:
        # 获取模块名
        web_frame_module_name = ret.group(1)
        # 获取可以调用web框架的应用名称
        app_name = ret.group(2)

    # 导入web框架的主模块
    web_frame_module = __import__(web_frame_module_name)
    # 获取那个可以直接调用的函数(对象)
    app = getattr(web_frame_module, app_name) 

    # print(app)  # for test

    # 启动http服务器
    http_server = WSGIServer(port, g_static_document_root, app)
    # 运行http服务器
    http_server.run_forever()


if __name__ == "__main__":
    main()
```

思路：

1. 完成可执行的服务器
2. 将服务器与Python代码分离，遵循WSGI协议

### 4. URL

目前开发的网站其实真正意义上都是动态网站，只是URL上有些区别，一般URL分为静态URL、动态URL、伪静态URL，他们的区别是什么?

#### 4.1 静态URL

类似域名`/news/2012-5-18/110.html` 我们一般称为`真静态URL`，每个网页有真实的物理路径，也就是真实存在服务器里的。

优点：网站打开速度快，因为它不用进行运算；另外网址结构比较友好，利于记忆。

缺点：最大的缺点是如果是中大型网站，则产生的页面特别多，不好管理。

静态网站对SEO的影响：静态URL对SEO肯定有加分的影响，因为打开速度快，这个是本质。

#### 4.2 动态URL

类似域名`/NewsMore.asp?id=5` 或者 域名`/DaiKuan.php?id=17`，带有？号的URL，我们一般称为动态网址，每个URL只是一个逻辑地址，并不是真实物理存在服务器硬盘里的。

优点：适合中大型网站，修改页面很方便，因为是逻辑地址，所以占用硬盘空间要比纯静态网站小。

缺点：因为要进行运算，所以打开速度稍慢，不过这个可有忽略不计，目前有服务器缓存技术可以解决速度问题。最大的缺点是URL结构稍稍复杂，不利于记忆。

动态URL对SEO的影响：目前百度SE已经能够很好的理解动态URL，所以对SEO没有什么减分的影响（特别复杂的URL结构除外）。所以你无论选择动态还是静态其实都无所谓，看你选择的程序和需求了。

#### 4.3 伪静态UL

类似域名`/course/74.html` 这个URL和真静态URL类似。他是通过伪静态规则把动态URL伪装成静态网址。也是逻辑地址，不存在物理地址。

优点是：URL比较友好，利于记忆。非常适合大中型网站，是个折中方案。

缺点是：设置麻烦，服务器要支持重写规则，小企业网站或者玩不好的就不要折腾了。另外进行了伪静态网站访问速度并没有变快，因为实质上它会额外的进行运算解释，反正增加了服务器负担，速度反而变慢，不过现在的服务器都很强大，这种影响也可以忽略不计。还有可能会造成动态URL和静态URL都被搜索引擎收录，不过可以用robots禁止掉动态地址。

对SEO的影响：和动态URL一样，对SEO没有什么减分影响。

### 5. 路由

在web框架中，如何根据请求的参数自动决定返回的页面数据，就是路由。代码如下：

```python
g_url_route = dict()
def route(url):
    def func1(func):
        # 添加键值对，key是需要访问的url，value是当这个url需要访问的时候，需要调用的函数引用
        g_url_route[url]=func
        def func2(file_name):
            return func(file_name)
        return func2
    return func1

@route("/index.html")
def index(file_name, url=None):
    """返回index.py需要的页面内容"""
    return "hahha"

def app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-Type', 'text/html')]
    start_response(status, response_headers)

    file_name = environ['PATH_INFO']
    try:
        return g_url_route[file_name](file_name)
    except Exception as ret:
        return "%s" % ret
    else:
        return str(environ) + '-----404--->%s\n'
```

#### 5.1 给路由添加正则表达式

在实际开发中，url中往往会带有很多参数，例如 /add/000007.html 中000007就是一个参数。如果没有正则的话，就需要编写N此@route来进行添加url对应的函数到字典中，此时字典中的键值对有N个，浪费空间。而采用了正则的话，只需要编写1此@route就可以完成多个url。

```python
g_url_route = dict()
def route(url):
    def func1(func):
        # 添加键值对，key是需要访问的url，value是当这个url需要访问的时候，需要调用的函数引用
        g_url_route[url]=func
        def func2(file_name):
            return func(file_name)
        return func2
    return func1

@route(r"/index.html")
def index(file_name, url=None):
    """返回index.py需要的页面内容"""
    return "hahha"

def app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-Type', 'text/html')]
    start_response(status, response_headers)

    file_name = environ['PATH_INFO']
    try:
         for url, call_func in g_url_route.items():
            print(url)
            ret = re.match(url, file_name)
            if ret:
                return call_func(file_name, url)
                break

        else:
            return "没有访问的页面--->%s" % file_name
    except Exception as ret:
        return "%s" % ret
    else:
        return str(environ) + '-----404--->%s\n'
```

### 6. URL编码

```python
import urllib.parse
# Python3 url编码
print(urllib.parse.quote("天安门"))
# Python3 url解码
print(urllib.parse.unquote("%E5%A4%A9%E5%AE%89%E9%97%A8"))
```

### 7. Log日志

#### 7.1 日志级别

日志一共分成5个等级，从低到高分别是：

1. DEBUG：详细的信息,通常只出现在诊断问题上
2. INFO：确认一切按预期运行
3. WARNING：一个迹象表明,一些意想不到的事情发生了,或表明一些问题在不久的将来(例如。磁盘空间低”)。这个软件还能按预期工作。
4. ERROR：更严重的问题,软件没能执行一些功能
5. CRITICAL：一个严重的错误,这表明程序本身可能无法继续运行

这5个等级，也分别对应5种打日志的方法： debug 、info 、warning 、error 、critical。默认的是WARNING，当在WARNING或之上时才被跟踪。

#### 7.2 日志输出

有两种方式记录跟踪，一种输出控制台，另一种是记录到文件中，如日志文件。

#### 7.3 将日志输出到控制台

比如，`log1.py` 如下：

```python
import logging  

logging.basicConfig(level=logging.WARNING,  # 输出级别
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')  

# 开始使用log功能
logging.info('这是 loggging info message')  
logging.debug('这是 loggging debug message')  
logging.warning('这是 loggging a warning message')  
logging.error('这是 an loggging error message')  
logging.critical('这是 loggging critical message')
```

运行结果

```python
2017-11-06 23:07:35,725 - log1.py[line:9] - WARNING: 这是 loggging a warning message
2017-11-06 23:07:35,725 - log1.py[line:10] - ERROR: 这是 an loggging error message
2017-11-06 23:07:35,725 - log1.py[line:11] - CRITICAL: 这是 loggging critical message
```

通过`logging.basicConfig`函数对日志的输出格式及方式做相关配置，上面代码设置日志的输出等级是WARNING级别，意思是WARNING级别以上的日志才会输出。另外还制定了日志输出的格式。

注意，只要用过一次log功能再次设置格式时将失效，实际开发中格式肯定不会经常变化，所以刚开始时需要设定好格式。

#### 7.4 将日志输出到文件

我们还可以将日志输出到文件，只需要在`logging.basicConfig`函数中设置好输出文件的文件名和写文件的模式。

log2.py 如下：

```python
import logging  

logging.basicConfig(level=logging.WARNING,  
                    filename='./log.txt',  
                    filemode='w',  
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')  
# use logging  
logging.info('这是 loggging info message')  
logging.debug('这是 loggging debug message')  
logging.warning('这是 loggging a warning message')  
logging.error('这是 an loggging error message')  
logging.critical('这是 loggging critical message')
```

运行效果

```python
python@ubuntu: cat log.txt 
2017-11-06 23:10:44,549 - log2.py[line:10] - WARNING: 这是 loggging a warning message
2017-11-06 23:10:44,549 - log2.py[line:11] - ERROR: 这是 an loggging error message
2017-11-06 23:10:44,549 - log2.py[line:12] - CRITICAL: 这是 loggging critical message
```

#### 7.5 既要把日志输出到控制台， 还要写入日志文件

这就需要一个叫作Logger 的对象来帮忙，下面将对他进行详细介绍，现在这里先学习怎么实现把日志既要输出到控制台又要输出到文件的功能。

```python
import logging  

# 第一步，创建一个logger  
logger = logging.getLogger()  
logger.setLevel(logging.INFO)  # Log等级总开关  

# 第二步，创建一个handler，用于写入日志文件  
logfile = './log.txt'  
fh = logging.FileHandler(logfile, mode='a')  # open的打开模式这里可以进行参考
fh.setLevel(logging.DEBUG)  # 输出到file的log等级的开关  

# 第三步，再创建一个handler，用于输出到控制台  
ch = logging.StreamHandler()  
ch.setLevel(logging.WARNING)   # 输出到console的log等级的开关  

# 第四步，定义handler的输出格式  
formatter = logging.Formatter("%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s")  
fh.setFormatter(formatter)  
ch.setFormatter(formatter)  

# 第五步，将logger添加到handler里面  
logger.addHandler(fh)  
logger.addHandler(ch)  

# 日志  
logger.debug('这是 logger debug message')  
logger.info('这是 logger info message')  
logger.warning('这是 logger warning message')  
logger.error('这是 logger error message')  
logger.critical('这是 logger critical message')
```

运行时终端的输出结果:

```python
2017-11-06 23:14:04,731 - log3.py[line:28] - WARNING: 这是 logger warning message
2017-11-06 23:14:04,731 - log3.py[line:29] - ERROR: 这是 logger error message
2017-11-06 23:14:04,731 - log3.py[line:30] - CRITICAL: 这是 logger critical message
```

在log.txt中，有如下数据：

```python
2017-11-06 23:14:04,731 - log3.py[line:27] - INFO: 这是 logger info message
2017-11-06 23:14:04,731 - log3.py[line:28] - WARNING: 这是 logger warning message
2017-11-06 23:14:04,731 - log3.py[line:29] - ERROR: 这是 logger error message
2017-11-06 23:14:04,731 - log3.py[line:30] - CRITICAL: 这是 logger critical message
```

#### 7.6 日志格式说明

`logging.basicConfig`函数中，可以指定日志的输出格式format，这个参数可以输出很多有用的信息，如下:

- `%(levelno)s`: 打印日志级别的数值
- `%(levelname)s`: 打印日志级别名称
- `%(pathname)s`: 打印当前执行程序的路径，其实就是`sys.argv[0]`
- `%(filename)s`: 打印当前执行程序名
- `%(funcName)s`: 打印日志的当前函数
- `%(lineno)d`: 打印日志的当前行号
- `%(asctime)s`: 打印日志的时间
- `%(thread)d`: 打印线程ID
- `%(threadName)s`: 打印线程名称
- `%(process)d`: 打印进程ID
- `%(message)s`: 打印日志信息

在工作中给的常用格式如下:

```python
format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s'
```

这个格式可以输出日志的打印时间，是哪个模块输出的，输出的日志级别是什么，以及输入的日志内容。

### 8. 一切皆对象

`Python globals()` 函数 会以字典类型返回当前位置的全部全局变量。

当定义一个函数、类、全局变量时，其实就是创建一个“对象”，然后在globals获取的这个字典中添加一个名字，让这个名字指向刚刚创建的对象空间而已。