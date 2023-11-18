查看所有的 dblink

```sql
select * from ALL_DB_LINKS;
```

查看用户权限

```sql
SELECT * FROM USER_SYS_PRIVS;
```

赋予权限

```sql
GRANT CREATE PUBLIC DATABASE LINK TO SYSTEM;
```

创建 dblink

```sql
CREATE DATABASE LINK oracle143
CONNECT TO system IDENTIFIED BY oracle
USING '(DESCRIPTION=
                (ADDRESS=(PROTOCOL=TCP)(HOST=1.0.0.143)(PORT=11521))
                (CONNECT_DATA=(SERVICE_NAME=xe))
            )';
```

-- DROP DATABASE LINK oracle143; 删除

操作

```sql
CREATE TABLE SETL_D AS  SELECT * FROM XXX.SETL_D@oracle143

```

踩坑：

当执行远程 dblink 的查询语句时，出现报错

ORA-22992: cannot use LOB locators selected from remote tables

这是因为这个表有的列不支持 lob

我们来到远程数据库，执行一下这个命令。

```sql
SELECT table_name, column_name, data_type
FROM all_tab_columns
WHERE data_type LIKE '%LOB%' AND table_name ='FEE_LIST_D';
```

可以看到相关的列。在 select 语句不要使用\* ，而是具体指定你需要的字段即可
