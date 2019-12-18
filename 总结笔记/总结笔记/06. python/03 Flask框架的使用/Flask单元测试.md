### 一、单元测试概念

程序开发过程中，写代码是为了实现需求。当我们的代码通过了编译，只是说明它的语法正确，功能能否实现则不能保证。 因此，当我们的某些功能代码完成后，为了检验其是否满足程序的需求。可以通过编写测试代码，模拟程序运行的过程，检验功能代码是否符合预期。

单元测试就是开发者编写一小段代码，检验目标代码的功能是否符合预期。通常情况下，单元测试主要面向一些功能单一的模块进行。

在Web开发过程中，单元测试实际上就是一些“断言”（assert）代码。

断言就是判断一个函数或对象的一个方法所产生的结果是否符合你期望的那个结果。 python中assert断言是声明布尔值为真的判定，如果表达式为假会发生异常。单元测试中，一般使用assert来断言结果。

### 二、Flask中单元测试

#### 1. 常用的断言方法

```bash
assertEqual     如果两个值相等，则pass
assertNotEqual  如果两个值不相等，则pass
assertTrue      判断bool值为True，则pass
assertFalse     判断bool值为False，则pass
assertIsNone    不存在，则pass
assertIsNotNone 存在，则pass
```

#### 2. 单元测试的基本写法

1. 首先，定义一个类，继承自unittest.TestCase

   ```python
   import unittest
   class TestClass(unitest.TestCase):
       pass
   ```

2. 其次，在测试类中，定义两个测试方法

   ```python
   import unittest
   class TestClass(unittest.TestCase):
   
       #该方法会首先执行，方法名为固定写法
       def setUp(self):
           pass
   
       #该方法会在测试代码执行完后执行，方法名为固定写法
       def tearDown(self):
           pass
   ```

3. 最后，在测试类中，编写测试代码

   ```python
    # 测试代码
    def test_app_exists(self):
    	pass
   ```

#### 3. 单元测试实例

##### 3.1 发送邮件测试

```python
#coding=utf-8
import unittest
from Flask_day04 import app
class TestCase(unittest.TestCase):
    # 创建测试环境，在测试代码执行前执行
    def setUp(self):
        self.app = app
        # 激活测试标志，此表示会在激活后打印所有源代码的异常
        app.config['TESTING'] = True
        # 测试模拟客户端
        self.client = self.app.test_client()

    # 在测试代码执行完成后执行
    def tearDown(self):
        pass

    # 测试代码
    def test_email(self):
        resp = self.client.get('/')
        print resp.data
        self.assertEqual(resp.data,'Sent　Succeed')
```

##### 3.2 数据库测试

```python
#coding=utf-8
import unittest
from author_book import *

#自定义测试类，setUp方法和tearDown方法会分别在测试前后执行。以test_开头的函数就是具体的测试代码。

class DatabaseTest(unittest.TestCase):
    def setUp(self):
        app.config['TESTING'] = True
        app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://root:mysql@localhost/test0'
        self.app = app
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()

    #测试代码
    def test_append_data(self):
        au = Author(name='itcast')
        bk = Book(info='python')
        db.session.add_all([au,bk])
        db.session.commit()
        author = Author.query.filter_by(name='itcast').first()
        book = Book.query.filter_by(info='python').first()
        #断言数据存在
        self.assertIsNotNone(author)
        self.assertIsNotNone(book)
```