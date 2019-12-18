### Apache

Apache本身没有功能，它的功能是加载的模块提供的。

#####Apache的启动过程

1. 解析配置文件，解析主配置文件Http.conf中的配置信息。将LoadModule、AddType等指令加载至内存。
2. 加载静态/动态模块，依据AddModule、LoadModule等指令加载Apache模块，映射到Apache的地址空间。
3. 系统资源初始化，日志文件、共享内存段、数据库连接等进行初始化。

Apache常用的命令：

- `httpd`：Apache http服务器程序；
  - -v：查看apache的版本；	-t：检测apache配置；
  - -m：查看加载的模块；

#####Apache配置：

1. 更改虚拟目录（httpd.conf）

   ```
   DocumentRoot "/home/www"
   ```

2. 更改虚拟目录权限（httpd.conf）

   ```
   Options Indexes FollowSymLinks	# 列出文件夹中的目录结构
   
   Order allow,deny   # 权限的生效顺序
   Allow from all # 全部允许
   Deny from all  # 全部拒绝
   Allow from 192.168.101.50  # 允许特定的IP地址
   Allow from 192.168         # 允许192.168开头的
   ```

3. 设置默认首页（httpd.conf）

   ```
   <IfModule dir_module>
       DirectoryIndex index.html
   </IfModule>
   ```

4. 使用虚拟主机

   ```
   1.开启虚拟主机
   Include /private/etc/apache2/extra/httpd-vhosts.conf   
   2.配置虚拟主机，最好注释掉主主机（httpd.conf）的虚拟主机，否则输入的域名会被解析到主配置中的虚拟目录。
   <VirtualHost *:80>
       ServerAdmin webmaster@dummy-host2.example.com     # 绑定管理员
       DocumentRoot "/usr/docs/dummy-host2.example.com"  # 虚拟目录路径
       ServerName dummy-host2.example.com                # 绑定域名
       DirectoryIndex index.php                          # 默认首页
       <Directory "/fff/ff">
   	    AllowOverride All
   	    Require all granted
       </Directory>
       ErrorLog "/private/var/log/apache2/dummy-host2.example.com-error_log"
       CustomLog "/private/var/log/apache2/dummy-host2.example.com-access_log" common
   </VirtualHost>
   ```

   > 虚拟目录：站点+权限。
   >
   > 虚拟主机：虚拟目录和域名绑定在一起。

#####分布式部署

一个Apache支撑多个虚拟主机，如果httpd.conf和php.ini配置发生了变化，所有的虚拟主机的配置都发生变化。分布式部署可以实现不同的虚拟主机有不同的配置。

.htaccess 文件又称为分布式部署文件，这个文件可以覆盖httpd.conf文件中的配置。一个网站下可以有多个分布式部署文件。每个 .htaccess 文件只能作用于当前目录和子目录。

1. Apache配置文件由主配置文件和分布式配置文件组成；

2. 主配置文件修改后需要重启服务器，分布式配置修改后不需要重启服务器；

3. 创建分布式部署文件必须借助于编辑器；

4. 分布式部署会降低Apache的性能，不是必须使用就不要用；

5. 必须在虚拟主机中允许分布式部署文件覆盖。

   ```
   <virtualHost *:80>
       DocumentRoot "/sss"
       ServerName www.baifu.com
       DirectoryIndex index.php
       <Directory "/sss">
       	AllowOverride all
       </Directory>
   </VirtualHost>
   ```

**通过分布式部署文件还可以更改PHP的配置**。 

分部署部署文件中可以通过php_value和php_flag来更改php配置的值（注意，这两个指令属于apache的指令）

- php_flag用来更改开关性质的配置；
- php_value用来更改值性质的配置；

```
php_flag session.auto_start 1
php_value session.gc_maxlifetime 10

<?php
echo ini_get('session.auto_start');      # 可以使用ini_get()来获取PHP配置的值
echo ini_get('session.gc_maxlifetime');
```

##### 