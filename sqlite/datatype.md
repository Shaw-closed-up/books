# SQLite 数据类型(datatype)

SQLite数据类型用于指定任何对象的数据类型。 SQLite中的每列，变量和表达式都有相关的数据类型。 这些数据类型在创建表时使用。 SQLite使用更通用的动态类型系统。 在SQLite中，值的数据类型与值本身相关联，而不是与其容器相关联。

## SQLite数据类型的类型

**SQLite存储类**

SQLite数据库中存储的值是以下存储类之一：

| 存储类  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| NULL    | 表示值为空(`null`)值。                                       |
| INTEGER | 表示值是一个有符号整数，根据值的大小存储在`1`,`2`,`3`,`4`,`6`或`8`个字节中。 |
| REAL    | 表示值是一个浮点值，存储为`8`位IEEE浮点数。                  |
| text    | 表示值是一个文本字符串，使用数据库编码(`utf-8`，`utf-16be`或utf-16le)存储 |
| BLOB    | 表示值是一个数据块，与输入的数据完全相同。                   |

> **注意**：SQLite存储类比数据类型更通用一些。 例如：`INTEGER`存储类包括不同长度的`6`种不同的整数数据类型。

## SQLite的近似类型

SQLite支持列的类型近似性。列可以存储任何类型的数据，但是列的首选存储类称为它的近似性类型。

在SQLite3数据库中有以下类型近似可用于分配。

| 存储类  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| TEXT    | 此列可使用存储类为`NULL`，`TEXT`或`BLOB`来存储所有数据。     |
| NUMERIC | 此列可包含使用所有五个存储类的值。                           |
| INTEGER | 它的行为与带有转换表达式异常的具有数字近似的列相同。         |
| REAL    | 它的行为类似于具有数字近似的列(除了它将整数值强制以浮点表示) |
| NONE    | 具有近似性`NONE`的列不会将一个存储类转为另一个存储类型       |

## SQLite近似和类型名称

以下是可以在创建SQLite表时使用的各种数据类型名称的列表。

| 数据类型                                                     | 相应的近似类型 |
| ------------------------------------------------------------ | -------------- |
| INT INTEGER TINYINT SMALLINT MEDIUMINT BIGINT UNSIGNED BIG INT INT2 INT8 | INTEGER        |
| CHARACTER(20) VARCHAR(255) VARYING CHARACTER(255) NCHAR(55) NATIVE CHARACTER(70) NVARCHAR(100) TEXT CLOB | TEXT           |
| BLOB - 未指定数据类型                                        | NONE           |
| REAL DOUBLE DOUBLE PRECISION FLOAT                           | REAL           |
| NUMERIC DECIMAL(10,5) BOOLEAN DATE DATETIME                  | NUMERIC        |

## 日期和时间数据类型

在SQLite中，没有单独的类型来存储日期和时间。 但是可以将日期和时间存储为TEXT，REAL或INTEGER值。

| 存储类  | 日期格式                                                 |
| ------- | -------------------------------------------------------- |
| TEXT    | 它以“`yyyy-mm-dd hh:mm:ss.sss`” 格式指定日期             |
| REAL    | 它规定了从公元前4714年11月24日在格林威治中午以后的天数。 |
| INTEGER | 它指定从`1970-01-01 00:00:00` utc开始的秒数。            |

## 布尔数据类型

在SQLite中，没有一个单独的布尔存储类。一个代替办法是将布尔值存储为整数`0`(假)和`1`(真)。