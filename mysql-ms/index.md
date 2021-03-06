# MySQL 复制简介(Replication)

主从复制（也称 AB 复制）允许将来自一个MySQL数据库服务器（主服务器）的数据复制到一个或多个MySQL数据库服务器（从服务器）。

> 复制是异步的 从站不需要永久连接以接收来自主站的更新。

根据配置，您可以复制数据库中的所有数据库，所选数据库甚至选定的表。

## MySQL中复制的优点包括：

- 横向扩展解决方案 - 在多个从站之间分配负载以提高性能。在此环境中，所有写入和更新都必须在主服务器上进行。但是，读取可以在一个或多个从设备上进行。该模型可以提高写入性能（因为主设备专用于更新），同时显着提高了越来越多的从设备的读取速度。
- 数据安全性 - 因为数据被复制到从站，并且从站可以暂停复制过程，所以可以在从站上运行备份服务而不会破坏相应的主数据。
- 分析 - 可以在主服务器上创建实时数据，而信息分析可以在从服务器上进行，而不会影响主服务器的性能。
- 远程数据分发 - 您可以使用复制为远程站点创建数据的本地副本，而无需永久访问主服务器。

> **前提是作为主服务器角色的数据库服务器必须开启二进制日志**


1. 主服务器上面的任何修改都会通过自己的 I/O tread(I/O 线程)保存在二进制日志 `Binary log` 里面。
2. 从服务器上面也启动一个 I/O thread，通过配置好的用户名和密码, 连接到主服务器上面请求读取二进制日志，然后把读取到的二进制日志写到本地的一个`Realy log`（中继日志）里面。
3. 从服务器上面同时开启一个 SQL thread 定时检查 `Realy log`(这个文件也是二进制的)，如果发现有更新立即把更新的内容在本机的数据库上面执行一遍。

每个从服务器都会收到主服务器二进制日志的全部内容的副本。

从服务器设备负责决定应该执行二进制日志中的哪些语句。

除非另行指定，否则主从二进制日志中的所有事件都在从站上执行。

如果需要，您可以将从服务器配置为仅处理一些特定数据库或表的事件。

**重要: 您无法将主服务器配置为仅记录特定事件。**

每个从站(从服务器)都会记录二进制日志坐标：

- 文件名
- 文件中它已经从主站读取和处理的位置。

由于每个从服务器都分别记录了自己当前处理二进制日志中的位置，因此可以断开从服务器的连接，重新连接然后恢复继续处理。

## 关于二进制日志

**mysqld**将数字扩展名附加到二进制日志基本名称以生成二进制日志文件名。每次服务器创建新日志文件时，该数字都会增加，从而创建一系列有序的文件。每次启动或刷新日志时，服务器都会在系列中创建一个新文件。服务器还会在当前日志大小达到[`max_binlog_size`]参数设置的大小后自动创建新的二进制日志文件 。二进制日志文件可能会比[`max_binlog_size`](使用大型事务时更大， 因为事务是以一个部分写入文件，而不是在文件之间分割。

为了跟踪已使用的二进制日志文件， [**mysqld**]还创建了一个二进制日志索引文件，其中包含所有使用的二进制日志文件的名称。默认情况下，它具有与二进制日志文件相同的基本名称，并带有扩展名`'.index'`。在[**mysqld**]运行时，您不应手动编辑此文件。

术语`二进制日志文件`通常表示包含数据库事件的单个编号文件。

术语 `二进制日志`  表示含编号的二进制日志文件集加上索引文件。

`SUPER` 权限的用户可以使用`SET sql_log_bin=0`语句禁用其当前环境下自己的语句的二进制日志记录