# 安装 oracle18

> 使用 airbyte 工具，必须目标源数据库 oracle 版本大于 18 或者 12+ 设置字符串长为 120

拉镜像

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c

docker run -d -p 1521:1521 -p 6500:5500 \
  --name oracle18c \
  -e ORACLE_SID=ORCLCDB \
  -e ORACLE_PDB=ORCLPDB1 \
  -e ORACLE_PWD=PassWord123 \
  -v /data/oracle18/oracle_data:/opt/oracle/oradata \
  registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c
```

怎么建表空间呢?

新建 tablespace

```sql
create bigfile tablespace XXX
datafile '/opt/oracle/oradata/bigfiledrg01.dbf' size 100G;
```

新建用户

```sql
create user XXX identified by 123456 default tablespace XXX;
```

报错说用户名非法

```sql
alter session set "_ORACLE_SCRIPT"=true;

create user XXX identified by 123456;
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
