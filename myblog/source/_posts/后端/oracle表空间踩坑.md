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

#### 一些查询命令

1、查询数据库中所有的表空间以及表空间所占空间的大小，直接执行语句就可以了：

```sql
select tablespace_name, sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;
```

2、查看表空间物理文件的名称及大小

```sql
select tablespace_name, file_id, file_name,
round(bytes/(1024*1024),0) total_space
from dba_data_files
order by tablespace_name;
```

3、查询所有表空间以及每个表空间的大小，已用空间，剩余空间，使用率和空闲率，直接执行语句就可以了：

```sql
select a.tablespace_name, total, free, total-free as used, substr(free/total * 100, 1, 5) as "FREE%", substr((total - free)/total * 100, 1, 5) as "USED%" from
(select tablespace_name, sum(bytes)/1024/1024 as total from dba_data_files group by tablespace_name) a,
(select tablespace_name, sum(bytes)/1024/1024 as free from dba_free_space group by tablespace_name) b
where a.tablespace_name = b.tablespace_name
order by a.tablespace_name;
```

4、查询某个具体的表所占空间的大小，把“TABLE_NAME”换成具体要查询的表的名称就可以了：

```sql
select t.segment_name, t.segment_type, sum(t.bytes / 1024 / 1024) "占用空间(M)"
from dba_segments t
where t.segment_type='TABLE'
and t.segment_name='TABLE_NAME'
group by OWNER, t.segment_name, t.segment_type;

```

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

在Oracle数据库中，用户和表空间是密切相关的概念。下面是它们之间的关系：

用户（User）：用户是数据库中的一个标识，用于识别和管理数据库中的对象和资源。每个用户都有自己的登录名和密码，并可以具有特定的角色和权限。用户用于连接到数据库，并在其拥有的表空间中创建和管理对象。

表空间（Tablespace）：表空间是逻辑存储结构，用于组织和管理数据库中的数据。它是由一个或多个数据文件组成的。每个表空间可以包含多个数据库对象，如表、索引、视图等。表空间定义了数据在物理存储介质上的布局和组织方式。

用户和表空间之间的关系是：

每个用户可以被分配到一个或多个表空间。用户在其分配的表空间中创建和管理自己的数据库对象。
表空间可以被多个用户共享，这意味着多个用户可以在同一个表空间中创建和管理对象。
用户可以访问和操作属于同一表空间的其他用户创建的对象，前提是他们具有适当的权限。
通过将用户和表空间结合使用，可以实现对数据库资源和对象的分离、管理和控制。每个用户在自己的表空间中拥有独立的存储空间，并且可以根据需要分配不同的权限和角色。

总结来说，用户是数据库中的标识和访问凭据，表空间是用于组织和管理数据的逻辑存储结构。用户在其分配的表空间中创建和管理对象。
