
查看表空间大小（需要 sys 用户权限）：

```sql
SELECT tablespace_name, SUM(bytes) AS total_bytes
FROM dba_data_files
GROUP BY tablespace_name;
```

修改表空间大小

```sql
ALTER TABLESPACE xxx RESIZE 500G
```

报错

```bash

ALTER TABLESPACE xxx RESIZE 500G
> ORA-01237: cannot extend datafile 13
ORA-01110: data file 13: '/opt/oracle/oradata/bigfiledrg01.dbf'
ORA-19502: write error on file "/opt/oracle/oradata/bigfiledrg01.dbf", block number 43734272 (block size=8192)
ORA-27072: File I/O error
Additional information: 4
Additional information: 43734272
Additional information: 806912

> Time: 1260.106s
```

应该是其他表空间占用过高，磁盘空间不够了。
尝试清理表空间。再执行

### 表空间概念

ORACLE 数据库被划分成称作为表空间的逻辑区域——形成 ORACLE 数据库的逻辑结构。

一个 ORACLE 数据库能够有一个或多个表空间，而一个表空间则对应着一个或多个物理的数据库文件，但一个数据库文件只能与一个表空间相联系。表空间是 ORACLE 数据库恢复的最小单位，容纳着许多数据库实体，如表、视图、索引、聚簇、回退段和临时段等。

每个 ORACLE 数据库均有 SYSTEM 表空间，这是数据库创建时自动创建的，用于存储系统的数据字典表、程序单元、过程、函数、包和触发器等。SYSTEM 表空间必须总要保持联机，因为其包含着数据库运行所要求的基本信息(关于整个数据库的数据字典、联机求助机制、所有回退段、临时段和自举段、所有的用户数据库实体、其它 ORACLE 软件产品要求的表)。

一个小型应用的 ORACLE 数据库通常仅包括 SYSTEM 表空间,然而一个稍大型应用的 ORACLE 数据库采用多个表空间会对数据库的使用带来更大的方便。

表空间类型

永久性表空间：一般保存表、视图、过程和索引等的数据。

临时性表空间：只用于保存系统中短期活动的数据。

撤销表空间：用来帮助回退未提交的事务数据。

表空间作用

表空间的作用能帮助 DBA 用户完成以下工作：

1. 决定数据库实体的空间分配

2. 设置数据库用户的空间份额

3. 控制数据库部分数据的可用性

4. 分布数据于不同的设备之间以改善性能

5. 备份和恢复数据。

用户创建其数据库实体时，必须给予表空间中具有相应的权力，所以对一个用户来说,其要操纵一个 ORACLE 数据库中的数据，应该：

1. 被授予关于一个或多个表空间中的 RESOURCE 特权

2. 被指定缺省表空间

3. 被分配指定表空间的存储空间使用份额

4. 被指定缺省临时段表空间,建立不同的表空间，设置最大的存储容量。
