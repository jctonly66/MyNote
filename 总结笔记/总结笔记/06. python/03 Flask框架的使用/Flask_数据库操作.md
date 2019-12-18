### 一、Flask数据库概论

SQLALchemy实际上是对数据库的抽象，让开发者不用直接和SQL语句打交道，而是通过Python对象来操作数据库，在舍弃一些性能开销的同时，换来的是开发效率的较大提升。

SQLAlchemy是一个关系型数据库框架，它提供了高层的ORM和底层的原生数据库的操作。flask-sqlalchemy是一个简化了SQLAlchemy操作的flask扩展。

安装：

```
pip install flask-sqlalchemy
```

**常用的SQLAlchemy字段类型**

| 类型名       | python中类型      | 说明                                                |
| ------------ | ----------------- | --------------------------------------------------- |
| Integer      | int               | 普通整数，一般是32位                                |
| SmallInteger | int               | 取值范围小的整数，一般是16位                        |
| BigInteger   | int或long         | 不限制精度的整数                                    |
| Float        | float             | 浮点数                                              |
| Numeric      | decimal.Decimal   | 普通整数，一般是32位                                |
| String       | str               | 变长字符串                                          |
| Text         | str               | 变长字符串，对较长或不限长度的字符串做了优化        |
| Unicode      | unicode           | 变长Unicode字符串                                   |
| UnicodeText  | unicode           | 变长Unicode字符串，对较长或不限长度的字符串做了优化 |
| Boolean      | bool              | 布尔值                                              |
| Date         | datetime.date     | 时间                                                |
| Time         | datetime.datetime | 日期和时间                                          |
| LargeBinary  | str               | 二进制文件                                          |

**常用的SQLAlchemy列选项**

| 选项名      | 说明                                              |
| ----------- | ------------------------------------------------- |
| primary_key | 如果为True，代表表的主键                          |
| unique      | 如果为True，代表这列不允许出现重复的值            |
| index       | 如果为True，为这列创建索引，提高查询效率          |
| nullable    | 如果为True，允许有空值，如果为False，不允许有空值 |
| default     | 为这列定义默认值                                  |

**常用的SQLAlchemy关系选项**

| 选项名         | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| backref        | 在关系的另一模型中添加反向引用                               |
| primary join   | 明确指定两个模型之间使用的联结条件                           |
| uselist        | 如果为False，不使用列表，而使用标量值                        |
| order_by       | 指定关系中记录的排序方式                                     |
| secondary      | 指定多对多中记录的排序方式                                   |
| secondary join | 在SQLAlchemy中无法自行决定时，指定多对多关系中的二级联结条件 |

在Flask-SQLAlchemy中，插入、修改、删除操作，均由数据库会话管理。会话用db.session表示。在准备把数据写入数据库前，要先将数据添加到会话中然后调用commit()方法提交会话。数据库会话也可以回滚，通过db.session.rollback()方法，实现会话提交数据前的状态。

在Flask-SQLAlchemy中，查询操作是通过query对象操作数据。最基本的查询是返回表中所有数据，可以通过过滤器进行更精确的数据库查询。

**将数据添加到会话中示例：**

```base
user = User(name='python')
db.session.add(user)
db.session.commit()
```

1. 在视图函数中定义模型类

   ```python
   from flask import Flask
   from flask_sqlalchemy import SQLAlchemy
   
   
   app = Flask(__name__)
   
   #设置连接数据库的URL
   app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://root:mysql@127.0.0.1:3306/Flask_test'
   # 追踪修改
   app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
   #查询时会显示原始SQL语句
   app.config['SQLALCHEMY_ECHO'] = True
   db = SQLAlchemy(app)
   
   class Role(db.Model):
       # 定义表名
       __tablename__ = 'roles'
       # 定义列对象
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(64), unique=True)
       us = db.relationship('User', backref='role')
   
       #repr()方法显示一个可读字符串
       def __repr__(self):
           return 'Role:%s'% self.name
   
   class User(db.Model):
       __tablename__ = 'users'
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(64), unique=True, index=True)
       email = db.Column(db.String(64),unique=True)
       pswd = db.Column(db.String(64))
       role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
   
       def __repr__(self):
           return 'User:%s'%self.name
   if __name__ == '__main__':
       # 删除表
       db.drop_all()
       # 创建表
       db.create_all()
       ro1 = Role(name='admin')
       ro2 = Role(name='user')
       # 插入数据
       db.session.add_all([ro1,ro2])
       db.session.commit()
       us1 = User(name='wang',email='wang@163.com',pswd='123456',role_id=ro1.id)
       us2 = User(name='zhang',email='zhang@189.com',pswd='201512',role_id=ro2.id)
       us3 = User(name='chen',email='chen@126.com',pswd='987654',role_id=ro2.id)
       us4 = User(name='zhou',email='zhou@163.com',pswd='456789',role_id=ro1.id)
       db.session.add_all([us1,us2,us3,us4])
       db.session.commit()
       app.run(debug=True)
   ```

查询

常用的SQLAlchemy查询过滤器

|   过滤器    |                       说明                       |
| :---------: | :----------------------------------------------: |
|  filter()   |      把过滤器添加到原查询上，返回一个新查询      |
| filter_by() |    把等值过滤器添加到原查询上，返回一个新查询    |
|    limit    |         使用指定的值限定原查询返回的结果         |
|  offset()   |       偏移原查询返回的结果，返回一个新查询       |
| order_by()  | 根据指定条件对原查询结果进行排序，返回一个新查询 |
| group_by()  | 根据指定条件对原查询结果进行分组，返回一个新查询 |

常用的SQLAlchemy查询执行器

|      方法      |                     说明                     |
| :------------: | :------------------------------------------: |
|     all()      |         以列表形式返回查询的所有结果         |
|    first()     |  返回查询的第一个结果，如果未查到，返回None  |
| first_or_404() |  返回查询的第一个结果，如果未查到，返回404   |
|     get()      |   返回指定主键对应的行，如不存在，返回None   |
|  get_or_404()  |   返回指定主键对应的行，如不存在，返回404    |
|    count()     |              返回查询结果的数量              |
|   paginate()   | 返回一个Paginate对象，它包含指定范围内的结果 |

查询:filter_by精确查询

返回名字等于wang的所有人

```python
User.query.filter_by(name='wang').all()
```

first()返回查询到的第一个对象

```python
User.query.first()
```

all()返回查询到的所有对象

```python
User.query.all()
```

filter模糊查询，返回名字结尾字符为g的所有数据。

```python
User.query.filter(User.name.endswith('g')).all()
```

get()，参数为主键，如果主键不存在没有返回内容

```python
User.query.get()
```

逻辑非，返回名字不等于wang的所有数据。

```python
User.query.filter(User.name!='wang').all()
```

逻辑与，需要导入and*，返回and*()条件满足的所有数据。

```python
from sqlalchemy import and_
User.query.filter(and_(User.name!='wang',User.email.endswith('163.com'))).all()
```

逻辑或，需要导入or_

```python
from sqlalchemy import or_
User.query.filter(or_(User.name!='wang',User.email.endswith('163.com'))).all()
```

not_ 相当于取反

```python
from sqlalchemy import not_
User.query.filter(not_(User.name=='chen')).all()
```

更新数据

```python
user = User.query.first()
user.name = 'dong'
db.session.commit()
User.query.first()
```

使用update

```python
User.query.filter_by(name='zhang').update({'name':'li'})
```

关联查询示例：角色和用户的关系是一对多的关系，一个角色可以有多个用户，一个用户只能属于一个角色。

查询角色的所有用户：

```python
#查询roles表id为1的角色
ro1 = Role.query.get(1)
#查询该角色的所有用户
ro1.us
```

查询用户所属角色：

```python
#查询users表id为3的用户
us1 = User.query.get(3)
#查询用户属于什么角色
us1.role
```

