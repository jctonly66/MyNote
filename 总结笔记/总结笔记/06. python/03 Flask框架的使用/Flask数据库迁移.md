### 一、使用数据库迁移工具的必要性

在开发过程中，需要修改数据库模型，而且还要在修改之后更新数据库。最直接的方式就是删除旧表，但这样会丢失数据。

更好的解决办法是使用数据库迁移框架，它可以追踪数据库模式的变化，然后把变动应用到数据库中。

在Flask中可以使用Flask-Migrate扩展，来实现数据迁移。并且集成到Flask-Script中，所有操作通过命令就能完成。

### 二、Flask-Migrate的使用

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
# 数据库迁移工具
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand


app = Flask(__name__)


class Config(object):
	# 使用pymysql驱动
    SQLALCHEMY_DATABASE_URI = "mysql+pymysql://root:ff@#0510@10.81.1.122:3306/FlaskTest"
    SQLALCHEMY_TRACK_MODIFICATIONS = True  # 跟踪打印mysql语句


app.config.from_object(Config)
db = SQLAlchemy(app)

# 创建Flask脚本管理工具对象
manage = Manager(app)
# 创建真正的迁移管理工具
Migrate(app, db)
# 为了导出数据库迁移命令，Flask-Migrate提供了一个MigrateCommand类，可以附加到flask-script的manager对象上。
manage.add_command("db", MigrateCommand)


class Author(db.Model):
    __tablename__ = 'tbl_authors'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(32), unique=True)
    email = db.Column(db.String(64))
    books = db.relationship("Book", backref="author")


class Book(db.Model):
    __tablename__ = 'tbl_books'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    author_id = db.Column(db.Integer, db.ForeignKey("tbl_authors.id"))


if __name__ == '__main__':
    manage.run()
```

迁移文件写好后就需要使用命令来完成迁移操作

#### 1. 创建迁移仓库

```
# 这个命令会创建migrations文件夹，所有迁移文件都放在里面。
python database.py db init
```

#### 2. 创建迁移脚本

```
#创建自动迁移脚本，m后代表描述符
python database.py db migrate -m 'initial migration'
```

自动创建迁移脚本有两个函数，upgrade()函数把迁移中的改动应用到数据库中。downgrade()函数则将改动删除。自动创建的迁移脚本会根据模型定义和数据库当前状态的差异，生成upgrade()和downgrade()函数的内容。对比不一定完全正确，有可能会遗漏一些细节，需要进行检查。

#### 3. 更新数据库

```
python database.py db upgrade
```

#### 4. 回退数据库

```
python database.py db downgrade 版本号
```

回退数据库时，需要指定回退版本号，由于版本号是随机字符串，为避免出错，建议先使用`python database.py db history`命令查看历史版本的具体版本号，然后复制具体版本号执行回退。

