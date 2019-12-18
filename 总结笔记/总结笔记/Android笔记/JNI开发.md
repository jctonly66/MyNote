#JNI开发
JNI（java native interface native）：java本地接口

###C语言：
####c的基本数据类型

| 基本数据类型 | 所占字节个数 |
| :-: | :-: |
| char | 1 |
| short | 2 |
| int | 4 |
| long | 8 |
| float | 4 |
| double | 8 |

修饰符：signed（有符号）、unsigned（无符号）

Java与C区别：

- java（char 2字节）     c （char 1字节）
- signed、unsigned只能用来修饰整型变量
- c没有boolean byte， c用0、非0代表false、true

>编译系统不同，占的字节数不同。

####c的输出函数

| 基本数据类型 | 占位符 | 基本数据类型 | 占位符 |
| :-: | :-: | :-: | :-: |
| char | %c | short | %hd |
| int | %d | long int | %ld |
| long long | %lld | unsigned | %u |
| float | %f | double | %lf |
| 0x输出 | %x | 八进制输出 | %o |
| 字符串 | %s |

####c的输入函数




