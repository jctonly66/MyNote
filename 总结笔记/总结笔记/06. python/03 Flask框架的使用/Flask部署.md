### 一、Flask的部署

当我们执行下面的hello.py时，使用的flask自带的服务器，完成了web服务的启动。在生产环境中，flask自带的服务器，无法满足性能要求，我们这里采用Gunicorn做wsgi容器，来部署flask程序。Gunicorn（绿色独角兽）是一个Python WSGI的HTTP服务器。从Ruby的独角兽（Unicorn ）项目移植。该Gunicorn服务器与各种Web框架兼容，实现非常简单，轻量级的资源消耗。Gunicorn直接用命令启动，不需要编写配置文件，相对uWSGI要容易很多。

区分几个概念：

**WSGI：**全称是Web Server Gateway Interface（web服务器网关接口），它是一种规范，它是web服务器和web应用程序之间的接口。它的作用就像是桥梁，连接在web服务器和web应用框架之间。

**uwsgi：**是一种传输协议，用于定义传输信息的类型。

**uWSGI：**是实现了uwsgi协议WSGI的web服务器。

这里使用的部署方式：nginx + gunicom + flask

#### 1. 使用Gunicorn

安装gunicorn成功后，通过命令行的方式可以查看gunicorn的使用信息。

```
$gunicorn -h
```

```bash
#直接运行，默认启动的127.0.0.1::8000
gunicorn 运行文件名称:Flask程序实例名
```

```bash
# 指定进程和端口号:-w: 表示进程（worker）。 -b：表示绑定ip地址和端口号（bind）。
$gunicorn -w 4 -b 127.0.0.1:5001 运行文件名称:Flask程序实例名
```

#### 2. 使用Nginx

web开发中，部署方式大致类似。简单来说，前端代理使用Nginx主要是为了实现分流、转发、负载均衡，以及分担服务器的压力。Nginx部署简单，内存消耗少，成本低。Nginx既可以做正向代理，也可以做反向代理。

正向代理：请求经过代理服务器从局域网发出，然后到达互联网上的服务器；服务端并不知道真正的客户端是谁。

反向代理：请求从互联网发出，先进入代理服务器，再转发给局域网内的服务器；客户端并不知道真正的服务端是谁。

区别：正向代理的对象是客户端。反向代理的对象是服务端。

**Nginx配置：**

默认安装到/usr/local/nginx/目录，进入目录。

Nginx操作：

```
#启动
sudo sbin/nginx
#查看
ps aux | grep nginx
#停止
sudo sbin/nginx -s stop
```

打开/usr/local/nginx/conf/nginx.conf文件

```
server {
    # 监听80端口
    listen 80;
    # 本机
    server_name localhost; 
    # 默认请求的url
    location / {
        #请求转发到gunicorn服务器
        proxy_pass http://127.0.0.1:5001; 
        #设置请求头，并将头信息传递给服务器端 
        proxy_set_header Host $host; 
    }
}
```