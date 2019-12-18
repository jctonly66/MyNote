## 一、虚拟环境

在开发过程中，当需要使用python的包时可以联网安装

```
sudo pip3 install 包名称
```

使用如上命令，会将包安装到 /usr/local/lib/python3.5/dist-packages 下。

**产生的问题**

如果在一台机器上，想开发多个不同的项目，需要用到同一个包的不同版本，如果还使用上面的命令，在同一个目录下安装或者更新，其它的项目必须就无法运行了，怎么办呢？

> 解决方案：**虚拟环境**。

### 1. 虚拟环境

那么什么是虚拟环境呢？

虚拟环境其实就是对真实pyhton环境的复制，这样我们在复制的python环境中安装包就不会影响到真实的python环境。通过建立多个虚拟环境，在不同的虚拟环境中开发项目就实现了项目之间的隔离。

#### 1.1 虚拟环境的安装及配置

以下是MAC电脑的虚拟环境安装：

```bash
sudo pip3 install virtualenv #安装虚拟环境
sudo pip3 install virtualenvwrapper  # 安装虚拟环境扩展包。为了用更加简单的命令来管理虚拟环境

which virtualenvwrapper.sh  # 获取该文件路径 编号1
```

修改用户家目录下的配置文件 ~/.bash_profile ，添加如下内容：

```bash
VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3  
export WORKON_HOME=$HOME/.virtualenvs	# 配置家目录
source 编号1
```

使用 source ~/.bash_profile 命令使配置文件生效。

#### 1.2 虚拟环境使用

```
mkvirtualenv -p python3 虚拟环境名称  # 创建虚拟环境
workon  # 查看虚拟环境
workon env  # 进入虚拟环境
rmvirtualenv env  # 删除虚拟环境
(virenv) dectivate	# 退出虚拟环境
(virenv) pip install package_name  # 注意，在虚拟环境中不可使用sudo来安装Python包，这样安装的包实际上安装在了真实的主机环境上。
(virenv) pip list  # 查看当前已经安装的包
(virenv) pip freeze  # 查看当前已经安装的包

```