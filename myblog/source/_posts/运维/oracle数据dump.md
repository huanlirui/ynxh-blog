前情提要：

我的 oracle 部署在 docker 容器中。

首先进入到 oracle 的容器内

```bash
docker exec -it <container_id> /bin/bash
```

链接到 oracle

```bash
sqlplus system/passowrd
```

创建导出目录

```sql
CREATE OR REPLACE DIRECTORY dump_dir AS '/home/oracle/dump/';
```

赋予 drg 用户权限

```sql
GRANT  read,  write  ON  DIRECTORY dump_dir  TO  drg;
```

导出命令

```sql
EXPDP drg/123456@ORCLCDB DIRECTORY=dump_dir DUMPFILE=data_pump.dmp SCHEMAS=drg
```

报错了

```
unknown command beginning "EXPDP drg/..." - rest of line ignored
```

这个指令不能再数据库中执行，退出到命令窗口执行

依然报错

```
EXPDP: command not found
```

应该是这个命令没有配置在环境变量中。

换成这个命令

```bash

$ORACLE_HOME/bin/expdp drg/123456@ORCLCDB DIRECTORY=dump_dir DUMPFILE=data_pump.dmp SCHEMAS=drg
```

接下来等候执行完成。

在目录/home/oracle/dump 下就有了 data_pump.dmp 文件

首先，将 dmp 文件从 Docker 容器中复制到本地机器上。你可以使用以下命令将文件从 Docker 容器复制到本地：

```bash
docker cp d00b8cca3e01:/home/oracle/dump/data_pump.dmp /home/drg/oracle_dmp

```

然后 gzip 压缩。传输到另一台服务器。

```bash
gzip <文件名>
```

在另外一台服务器解压缩。然后传到 docker 容器中。

```bash
gzip -d example.txt.gz

docker cp /home/drg/oracle_dmp d00b8cca3e01:/home/oracle/dump/
```

进入 docker 容器

```bash
docker exec -it -u root 容器名称 bash
```

连接到数据库

```
sqlplus system/passowrd
```

建立目录

```sql
CREATE DIRECTORY dump_dir AS '/home/oracle';
```

授予权限

```sql
GRANT READ, WRITE ON DIRECTORY dump_dir TO DRG;
```

开始执行

```bash
$ORACLE_HOME/bin/impdp drg/heo9zWvJ475r@ORCLCDB  DIRECTORY=dump_dir DUMPFILE=data_pump.dmp LOGFILE=import.log
```

报错了：

```bash
ORA-39001: invalid argument value
ORA-39000: bad dump file specification
ORA-31640: unable to open dump file "/home/oracle/data_pump.dmp" for read
ORA-27041: unable to open file
Linux-x86_64 Error: 13: Permission denied

```

因为容器中是 oracle 用户。没有权限对 root 用户的 dmp 文件操作。

退出容器

重新进入容器

```bash
docker exec -it -u root 容器名称 bash
```

修改文件的用户

```bash
chown -R oracle data_pump.dmp
```

重新用 oracle 用户登录进去

执行导入操作。又报错了

大概意思是，某些表和存储过程已存在。。没有覆盖式导入。

```bash
$ORACLE_HOME/bin/impdp drg/heo9zWvJ475r@ORCLCDB table_exists_action=replace  DIRECTORY=dump_dir DUMPFILE=data_pump.dmp LOGFILE=import.log
```

table_exists_action=replace 就是覆盖式导入表

### 一些个小 tip

查询某个 schema 下的所有 functions ，PROCEDURE 等

```sql
SELECT \* FROM all_procedures WHERE OBJECT_TYPE IN ('FUNCTION','PROCEDURE','PACKAGE')
and owner = 'DRG' order by object_name;
```

批量删除所有

```sql
BEGIN
FOR obj IN (SELECT object_name, object_type FROM all_procedures WHERE OBJECT_TYPE IN ('FUNCTION','PROCEDURE','PACKAGE') AND owner = 'DRG')
LOOP
IF obj.object_type = 'FUNCTION' THEN
EXECUTE IMMEDIATE 'DROP FUNCTION DRG.' || obj.object_name;
ELSIF obj.object_type = 'PROCEDURE' THEN
EXECUTE IMMEDIATE 'DROP PROCEDURE DRG.' || obj.object_name;
ELSIF obj.object_type = 'PACKAGE' THEN
EXECUTE IMMEDIATE 'DROP PACKAGE DRG.' || obj.object_name;
END IF;
END LOOP;
END;
```

还有个坑，做数据迁移，记得数据库的时区问题
