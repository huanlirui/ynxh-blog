# 安装 oracle18

> 使用 airbyte 工具，必须目标源数据库 oracle 版本大于 18 或者 12+ 设置字符串长为 120

## 拉镜像

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c

docker run -d -p 1521:1521 -p 6500:5500 \
  --name oracle18c \
  -e ORACLE_SID=ORCLCDB \
  -e ORACLE_PDB=ORCLPDB1 \
  -e ORACLE_PWD=heo9zWvJ475r \
  -v /data/oracle18/oracle_data:/opt/oracle/oradata \
  registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c
```

## 修改密码

```bash
docker exec -it <容器名称或ID> bash

sqlplus / as sysdba

ALTER USER SYSTEM IDENTIFIED BY XXXX;
```

## 新建 tablespace

```sql

CREATE TABLESPACE XXX
DATAFILE '/opt/oracle/oradata/drg01.dbf' SIZE 150G
AUTOEXTEND ON
NEXT 50M
MAXSIZE UNLIMITED;

-- 大文件空间
create bigfile tablespace XXX
datafile '/opt/oracle/oradata/bigfiledrg01.dbf' size 100G;
```

SIZE 100M：设置数据文件的初始大小为100M。这意味着在创建表空间时，会为数据文件分配100M的空间。

AUTOEXTEND ON：开启数据文件的自动扩展功能。这样，当数据文件满了之后，它会自动增长以容纳更多的数据。

NEXT 10M：设置数据文件的增长大小为10M。当数据文件需要扩展时，它会按照10M的大小逐步增长。

MAXSIZE UNLIMITED：设置数据文件的最大大小为无限制。这样，数据文件可以无限制地增长，只受到磁盘空间的限制。

> 大型文件（Bigfile）表空间是Oracle数据库中的一种特殊类型的表空间，用于存储大型数据文件。相比于传统的小型文件表空间，大型文件表空间具有以下特点和用途：

支持更大的数据文件：大型文件表空间可以容纳单个数据文件的大小远远超过传统表空间的限制。在Oracle 10g之前，每个数据文件的最大大小为32GB，而大型文件表空间可以支持达到4TB的单个数据文件大小。从Oracle 10g开始，这个限制进一步提高到了128TB。

简化管理和提高性能：使用大型文件表空间可以简化数据库的管理和维护工作。由于一个大型文件表空间只需要一个数据文件，所以减少了管理多个小型数据文件的复杂性。此外，大型文件表空间还可以提供更好的I/O性能，因为它们可以使用更大的数据块大小。

提高数据库的可伸缩性：大型文件表空间提供了更大的存储容量，可以支持处理大量数据和应用的需求。它们适用于存储大型表、分区表、LOB（Large Object）数据等需要更大存储空间的场景。

##### 如果有这个报错

```
CREATE TABLESPACE XXX
DATAFILE '/opt/oracle/oradata/drg01.dbf' SIZE 150G
AUTOEXTEND ON
NEXT 50M
MAXSIZE UNLIMITED

> ORA-01144: File size (19660800 blocks) exceeds maximum of 4194303 blocks
```

根据你提供的错误信息，看起来你正在尝试创建一个超过4194303个块的文件，但是这超过了Oracle数据库的最大块数限制。

默认情况下，Oracle数据库的块大小为8KB，因此最大块数的计算公式为：

最大块数 = 最大文件大小 / 块大小

根据你提供的错误信息，最大块数为4194303，这意味着你的数据库块大小可能为8KB。根据这个计算，最大文件大小约为32GB。

如果你想创建更大的表空间，你需要考虑以下选项：

使用大型文件表空间（Bigfile）：大型文件表空间允许单个数据文件的最大大小超过传统表空间的限制。你可以考虑创建一个大型文件表空间来容纳更大的数据文件。

分割数据文件：如果你需要更大的存储空间，你可以将数据文件分割为多个较小的文件，并将它们放置在不同的表空间中。这样，你可以通过多个数据文件来扩展存储空间。

## 新建用户

```sql
create user XXX identified by 123456 default tablespace XXX;
```

### 报错说用户名非法

```sql
alter session set "_ORACLE_SCRIPT"=true;

create user XXX identified by 123456;
```

### 登录报错

```bash
ORA-01045: user XXX lacks CREATE SESSION privilege; logon denied
```

ORA-01045错误表示用户XXX缺少CREATE SESSION权限，因此无法登录。CREATE SESSION权限是允许用户登录到数据库的基本权限之一。

要解决此问题，您需要确保用户XXX具有CREATE SESSION权限。您可以使用具有足够权限的其他用户登录到数据库，并为用户XXX授予CREATE SESSION权限。

下面是授予用户XXX CREATE SESSION权限的示例：

```sql
GRANT CREATE SESSION TO XXX;
```

成功了

你可能会出现这个状况：

```
ERROR at line 1:
ORA-01109: database not open


SQL> Disconnected from Oracle Database 18c Enterprise Edition Release 18.0.0.0.0 - Production
Version 18.3.0.0.0
mkdir: cannot create directory '/opt/oracle/oradata/dbconfig': Permission denied
mv: cannot stat '/opt/oracle/product/18c/dbhome_1/dbs/spfileORCLCDB.ora': No such file or directory
mv: cannot move '/opt/oracle/product/18c/dbhome_1/dbs/orapwORCLCDB' to '/opt/oracle/oradata/dbconfig/ORCLCDB/': No such file or directory
mv: cannot move '/opt/oracle/product/18c/dbhome_1/network/admin/sqlnet.ora' to '/opt/oracle/oradata/dbconfig/ORCLCDB/': No such file or directory
mv: cannot move '/opt/oracle/product/18c/dbhome_1/network/admin/listener.ora' to '/opt/oracle/oradata/dbconfig/ORCLCDB/': No such file or directory
mv: cannot move '/opt/oracle/product/18c/dbhome_1/network/admin/tnsnames.ora' to '/opt/oracle/oradata/dbconfig/ORCLCDB/': No such file or directory
cp: cannot create regular file '/opt/oracle/oradata/dbconfig/ORCLCDB/': No such file or directory
ln: failed to create symbolic link '/opt/oracle/product/18c/dbhome_1/dbs/orapwORCLCDB': File exists
ln: failed to create symbolic link '/opt/oracle/product/18c/dbhome_1/network/admin/sqlnet.ora': File exists
ln: failed to create symbolic link '/opt/oracle/product/18c/dbhome_1/network/admin/listener.ora': File exists
ln: failed to create symbolic link '/opt/oracle/product/18c/dbhome_1/network/admin/tnsnames.ora': File exists
cp: cannot stat '/opt/oracle/oradata/dbconfig/ORCLCDB/oratab': No such file or directory
The Oracle base remains unchanged with value /opt/oracle
#####################################
########### E R R O R ###############
DATABASE SETUP WAS NOT SUCCESSFUL!
Please check output for further info!
########### E R R O R ###############
#####################################
The following output is now a tail of the alert.log:
```

大概意思是容器内的用户没有权限对宿主主机的目录进行操作。

同样的问题：
<https://github.com/oracle/docker-images/issues/227>

我的解决方案：

1. 把容器停了，彻底删了。

   ```
   docker stop id
   docker rm id
   ```

2. 找到宿主主机的目录
 /data/oracle18/oracle_data

 进到目录执行 ：

 ```bash
 sudo chmod 777 oracle_data/

 sudo chown 54321:54322 oracle_data/

 ```

> 在容器中，用户 oracle 是 UID 54321，组 dba 是 GID 54322。

 3. 然后重新执行容器运行命令即可
