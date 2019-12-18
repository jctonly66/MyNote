##   Mysql

### 基本概念

- 冗余：存储两倍数据，冗余降低了性能，但提高了数据的安全性。
- 主键：主键是唯一的。一个数据表只能包含一个主键。可以使用主键来查询数据。
- 外键：外键用于关联两个表。
- 复合键：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。
- 索引：使用索引可快速访问数据表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。
- 参照完整性：参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。
- 事务：在mysql中只有使用了Innodb数据库引擎的数据库或表才支持事务。
- 临时表：临时表只在当前连接可见，关闭连接时，mysql会自动删除表并释放所有空间。创建临时表时再table前添加temporary关键字。

Mysql目前属于Oracle公司。

### 数据类型

#### 整数类型的存储和范围

| 类型        | 大小          | 范围（有符号）            | 范围（无符号）  | 用途            |
| ----------- | ------------- | :------------------------ | --------------- | --------------- |
| TINYINT     | 1字节         | (-128，127)               | （0，255）      | 小整数值        |
| SMALLINT    | 2字节         | (-32 768，32767)          | （0，65535）    | 大整数值        |
| MEDIUMINT   | 3字节         | (-8 388 608，8388 607)    | （0，16777215） | 大整数值        |
| INT/INTEGER | 4字节         |                           |                 | 大整数值        |
| BIGINT      | 8 字节        |                           |                 | 极大整数值      |
| FLOAT       | 4 字节        | float(M,D) 总长度，小数位 |                 | 单精度 浮点数值 |
| DOUBLE      | 8 字节        |                           |                 | 双精度 浮点数值 |
| DECIMAL     | m + 2 / d + 2 | 依赖于M和D的值            | 依赖于M和D的值  | 小数值          |

#### 日期和时间类型

| 类型      | 大小 (字节) | 格式                | 用途                     |      |
| --------- | ----------- | ------------------- | ------------------------ | ---- |
| DATE      | 3           | YYYY-MM-DD          | 日期值                   |      |
| TIME      | 3           | HH:MM:SS            | 时间值或持续时间         |      |
| YEAR      | 1           | YYYY                | 年份值                   |      |
| DATETIME  | 8           | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |      |
| TIMESTAMP | 4           | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |      |

#### 字符串类型

| 类型       | 大小                | 用途                            |
| ---------- | ------------------- | ------------------------------- |
| CHAR       | 0-255字节           | 定长字符串                      |
| VARCHAR    | 0-65535 字节        | 变长字符串                      |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255字节           | 短文本字符串                    |
| BLOB       | 0-65 535字节        | 二进制形式的长文本数据          |
| TEXT       | 0-65 535字节        | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                    |

CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。

有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。

### Mysql 命令

- systemctl start mysqld	// 启动mysql
	 systemctl status mysqld	// 查看mysql运行状态
- ps -ef | grep mysqld            // 检查Mysql服务器是否启动，如果启动，将输出mysql进程列表。

#### 添加Mysql用户

如果需要添加 MySQL 用户，你只需要在 mysql 数据库中的 user 表添加新用户即可。

```mysql
mysql> use mysql;
Database changed
mysql> INSERT INTO user 
          (host, user, authentication_string, 
           select_priv, insert_priv, update_priv) 
           VALUES ('localhost', 'guest', 
           PASSWORD('guest123'), 'Y', 'Y', 'Y');	# PASSWORD()函数可以对密码进行加密
mysql> FLUSH PRIVILEGES;	#刷新权限，如果不使用，新用户就无法连接，除非重启mysql

# 同时可以在创建用户时，为用户指定权限，在对应的权限列中，在插入语句中设置为‘Y’即可，用户权限列表如下：
#Select_priv, Insert_priv, Update_priv, Delete_priv, Create_priv, Drop_priv, Reload_priv, Shutdown_priv, Process_priv, File_priv, Grant_priv, References_priv, Index_priv,Alter_priv
```

也可以通过SQL的GRANT命令给指定数据库（TUTORIALS）添加用户 zara，密码为 zara123。

```mysql
mysql> use mysql;
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
    -> ON TUTORIALS.*
    -> TO 'zara'@'localhost'
    -> IDENTIFIED BY 'zara123';
```

#### 显示数据库/表信息

```mysql
use db_name; # 选择要操作的数据库，使用该命令后所有 mysql 命令都只针对该数据库
show tables;	   # 列出数据库的所有表
show columns from tb_name; # 显示数据表的属性，属性类型，主键信息，是否为NULL，默认值等其他信息。
show index from tb_name; # 显示数据表的详细索引信息，包括主键。
show tables status like [from db_name][like 'pattern']\G; # 输出Mysql DBMS的性能及统计信息。
									 # \G 表示查询结果按列打印。
```

##### 数据库操作

```mysql
# 创建数据库，[编码集为utf8][校对集（数据比较的规则，依赖字符集）]
create database db_name [charset utf8][collate utf8_general_ci];
# 查看数据库 [模糊查询]
show databases [like 'pattern'];
# 查看数据库的创建语句
show create database db_name;
# 更新数据库，数据库的名字不可以修改，数据库的修改仅限库选项。
alter database db_name charset [=] gbk;
# 删除数据库
drop database db_name;
```

##### 数据表操作

```mysql
# 创建数据表
use db_name;
create table [if not exists] tb_name (
    column_name column_type
)[表选项];
eg:
CREATE TABLE IF NOT EXISTS tb_name(	      
   `column1` INT UNSIGNED AUTO_INCREMENT,   //自动增长
   `column2` VARCHAR(100) NOT NULL,	        //非空
   `column3` VARCHAR(40) NOT NULL,
   `colum4` DATE,	
   PRIMARY KEY ( `runoob_id` )			    //设置主键
)[ENGINE=InnoDB][ DEFAULT CHARSET=utf8];	 //ENGINE 设置存储引擎，CHARSET设置编码
# 查看所有表
show tables [like 'pattern'];
# 查看表创建语句
show create table tb_name;
# 查看表结构
desc tb_name | describe tb_name | show columns from tb_name;
# 修改数据表
rename table old_name to new_name;     //修改表名
alter table tb_name charset [=] utf8;  //修改表选项
# 修改表字段
alter table tb_name add [column] 字段名 数据类型 [列属性] [位置];//添加列，位置：first、after 字段名。
alter table tb_name modify 字段名 数据类型 [属性] [位置];  //修改列
alter table tb_name change 旧字段 新字段名 数据类型 [属性] [位置];  //修改列名
alter table tb_name drop 字段名;  //删除列
# 删除数据表
drop table tb_name,tb_name2,tb_name3...;  # 删除表
```

##### 数据操作

```mysql
# 插入数据
// 给全表字段插入数据，如果不指定字段列表，值的顺序必须与表中字段顺序一致
// 凡是非数值类型，都需要使用引号（建议是单引号）。
insert into tb_name [(field1, field2, ...fieldN)] values (value1, value2, ...valueN) [,(value1, value2, ...valueN), (value1, value2, ...valueN)];
# 查询数据
select */column_name from tb_name [where Clause [BINARY]] [limit n][offset m];
// 可以使用一个或多个表，表之间用，分割
// 可以使用OFFSET来指定select语句开始查询的数据偏移量，默认情况下偏移量为0
// BINARY：区分大小写	
# 更新数据
update tb_name set 字段 = 值[, 字段=值] [where Clause];
# 删除数据
delete from tb_name [where clause];

# 模糊匹配 LIKE子句中使用%来表示任意字符，如果没有使用，与等号=效果一样。
select field1, field2, ...fieldN from tb_name where field1 LIKE '%fff'

# UNION操作符：用于连接两个以上的select语句的结果组合到一个结果集合中，多个select语句会删除重复的数据。
select expression1, expression2, ...expression_n 	# 要检索的列。	
from tables [where conditions]						# 要检索的数据表，检索条件
union [all | distinct]								# all：返回重复数据，distinct：删除重复数据
select expression1, expression2, ...expression_n	
from tables [where conditions]

# order by排序
select field1, field2, ...fieldN 
from tb_name1, tb_name2... 
order by field1, [field2...] [ASC | DESC]	# 默认升序，DESC：降序

# group by分组，在分组的列上我们可以使用COUNT，SUM，AVG等函数
select column_name, function(column_name)
from tb_name
where column_name operator value
group by column_name;

# 使用WITH ROLLUP可以实现在分组统计数据基础上再进行相同的统计（SUM，AVG，COUNT...）。会新增行，并命名为NULL。可以使用coalesce来设置一个渠道NULL的名称。
coalesce(name, '总数') # 若name存在，则使用name；若为NULL则使用‘总数’

# 内连接：获取两个表中字段匹配关系的记录；交集
select a.name, b.country from tb_name1 a inner join tb_name2 b on a.id = b.id
# 左连接：获取左表所有记录，即使右表没有对应匹配的记录；把b表数据加入a表再查询
select a.name, b.country from tb_name1 a left join tb_name2 b on a.id = b.id
# 右连接：获取右表所有记录，即使左表没有对应匹配的记录；把a表数据加入b表再查询

# 当提供的查询条件字段为NULL时，该命令可能就无法工作（不能使用 = !=）；为此，提供了三大运算符：
# is NULL：当列的值是NULL，返回true
# is NOT NULL：当列的值不为NULL，返回true
# <=>：比较操作符，当比较的两个值为NULL时返回true。

# mysql同样支持正则表达式的匹配，使用regexp操作符。
# '^'：匹配输入字符串的开始位置；
# '$'：匹配输入字符串的结束位置；
# '.'：匹配除'\n'之外的任何单个字符；
# '[...]'：字符集合，匹配所包含的任意一个字符；
# '[^...]'：负值字符集合。匹配未包含的任意字符；
# 'p1|p2|p3'：匹配p1或p2或p3。
# '*'：匹配前面的子表达式零次或多次。例如，'zo*' 能匹配'z'以及'zoo'；
# '+'：匹配前面的字表达式一次或多次。例如，'zo+' 能匹配'zo'以及'zoo'，但不能匹配'z'；
# '{n}'：n是一个非负整数。匹配确定的n次。例如：'food'；
# '{n, m}'：m和n均为非负整数，其中n <= m。最少匹配n次且最多匹配m次。
select name from tb_name where name regexp '正则表达式'

# 开始事务
begin
# 提交事务
commit
# 回滚事务
rollback

# 修改数据表名或修改数据表字段，mysql alter
# 使用alter命令及drop子句来删除表字段
alter table tb_name drop i	# 如果只剩下一个字段则无法使用DROP来删除
# 修改字段类型及名称
alter table tb_name modify 字段 字段类型
alter table tb_name change 原字段 新字段 字段类型
# 删除字段默认值
alter table tb_name alter i drop default
# 修改数据表名
alter table tb_name rename to new_name
# 修改存储引擎
alter table tb_name engine = myisam
# 删除外键约束
alter table tb_name drop foreign key keyName

# 索引会大大提高查询速度，但会降低更新表的速度。更新表时，还要保存一下索引文件。建立索引会占用磁盘空间的索引文件。
# 创建索引
create index index_name on tb_name(username(length))
# 添加索引
alter table tb_name add index index_name(column_name)
# 删除索引
drop index [indexName] on tb_name
# 创建唯一索引（需要保证索引列的值必须唯一，但允许有空值。如果是组合索引，列值的组合必须唯一）
create unique index index_name on tb_name(username(length))
# 添加唯一索引
alter table tb_name add unique [index_name] (username(length))
```

###问题集锦

#### Mysql密码重置

如果不小心忘记Mysql密码，可以通过 my.cnf 文件添加 skip-grant-tables 来重置密码。步骤如下：

1. 打开 my.cnf 配置文件，找到 [mysqld] ，在该行下面添加参数 `skip-grant-tables`；

2. 重启Mysql服务，`service mysql restart`；

3. 登录mysql，此时不需要输入密码，直接回车，`mysql -u root -p`；

4. 更改root密码为123456

   ```mysql
   mysql> use mysql;
   mysql>  update user set authentication_string=password("123456") where user='root';
   mysql> flush privileges;  # 刷新权限
   ```

5. 修改成功后，注释掉 `skip-grant-tables`参数，重启 Mysql 服务。

##### 注释：

-- ：单行注释。

##### mysql端插入/读取中文时报错

1. 在会话级别修改：
   1. `show variables like 'character_set%';`   // 查看服务器默认的对外处理的字符集
   2. 查看客户端的字符集，假设为GBK
   3. `set character_set_client = gbk;`  // 修改服务器处理来自客户端请求的字符集
   4. `set character_set_results = gbk;` //修改服务器给定数据的字符集为GBK

   or

   1. 也可以快速设置字符集`set names gbk`;

2. 

#### 模糊查询

1. `_`：匹配单个字符。
2. `%`：匹配多个字符。 

#### 校对集

校对集：数据比较的方式。

校对集有三种格式：

1. _bin:binary，二进制比较，取出二进制位，一位一位的比较，区分大小写。 
2. _cs:case sensitive，大小写敏感，区分大小写。
3. _ci:case insensitive，大小写不敏感，不区分大小写。

```
# 查看所有校对集 
show collation; 
```

校对集应用：只有当数据产生比较的时候，校对集才会生效。（比如排序）

校对集必须在没有数据之前声明好，如果有了数据，那么再进行校对集修改，那么修改无效。 















