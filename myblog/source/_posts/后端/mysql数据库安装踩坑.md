#### mac os Mysql 安装

### 1. 首先已安装 homebrew

### 2. brew install mysql

### 3. 配置 mysql 初始信息

```
//执行
mysql_secure_installation

//如果报如下错误：
//ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

//执行
sudo chmod 777 /var/run/mysql

//再次执行
mysql_secure_installation
```

```
//开始对mysql进行安全设置
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?
//是否设置验证密码组件：y
Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file
//密码验证策略有三个级别：0=低，1=中，2=强：0
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
Please set the password for root here.
//输入密码：admin123
New password:
//重复密码：admin123
Re-enter new password:

Estimated strength of the password: 50
//密码强度50，是否使用这个密码：y
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.
//删除匿名用户（测试用用户）：y
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.

Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.
//不允许远程登录根目录：n   （此处根据个人选择，建议输入y）
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

//是否删除测试数据库：y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

//是否刷新权限：y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

//完成
All done!
```

### 5. mysql 的启动、停止、重启

```
mysql.server stop
mysql.server start
mysql.server restart
```

### 6. 开始使用 mysql

```
mysql -uroot -p
//回车输入密码即可
```

### 7. mysql 常用命令

```
//创建数据库：
    CREATE DATABASE db_name;
//查看编码格式：
    SHOW CREATE DATABASE db_name;
//查看当前服务器下的数据库列表：
    SHOW DATABASES;
//修改编码格式：
    ALTER DATABASE db_name CHARACTER SET utf8;
//删除数据库：
    DROP DATABASE db_name;
//选择数据库：
    USE db_name;
//显示当前数据库：
    SELECT DATABASE();
//-----------------------------------------------------------------------------
//创建数据表(创建字段名)：
    CREATE TABLE table_name (
        column_name data_type,
        ......
    )
//显示数据表列表：
    SHOW TABLES [FROM db_name];
//显示数据表的结构：
    SHOW COLUMNS FROM tb_name;
//修改数据表：
    //添加一列：
        ALTER TABLE tb_name ADD column_name data_type;
    //添加多列：
        ALTER TABLE tb_name ADD (column_name data_type,…);
    //删除列：
        ALTER TABLE tb_name DROP column_name,DROP column_name,……
//-----------------------------------------------------------------------------
//插入记录(创建字段下的数据)：
    INSERT tb_name (col_name,...) VALUES(val,...);
//更新记录UPDATE：
    UPDATE tb_name SET age = age + 5, sex = 2 WHERE username='TOM';
//删除记录DELETE：
    DELETE FROM tb_name WHERE id=2;
//查找记录：
    SELECT expr,... FROM tb_name;

//排序
    SELECT * FROM stu ORDER BY 笔试 LIMIT 0,8
```

### 8. Navicat12 连接 mysql

> 我本地已破解 Navicat12 分享（MAC 版）:https://pan.baidu.com/s/1LaYeKffXurubNnfn2l-USA 密码:19u6

### 坑来了！！！

连接报错
2059 - Authentication plugin 'caching_sha2_password' cannot be loaded: dlope

#### 出现这个原因是 mysql8 之前的版本中加密规则是 mysql_native_password,而在 mysql8 之后,加密规则是 caching_sha2_password, 解决问题方法有两种,一种是升级 navicat 驱动,一种是把 mysql 用户登录密码加密规则还原成 mysql_native_password.

这里用第二种方式 ，解决方法如下

1. 登入 Mysql
```
mysql -u root -p
password: //登入
<!-- 登入mysql后 -->
use mysql;    //切换到mysql数据库
select user,host,plugin,authentication_string from user; //查询user表。我们来看一下信息
```
执行后我们可以看到。user表中root用户的加密规则为caching_sha2_password

2. 修改账户密码加密规则并更新用户密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;   #修改加密规则

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';   #更新一下用户的密码
```
3. 刷新权限并重置密码
FLUSH PRIVILEGES;   #刷新权限 

4. 再次打开Navicat Premium 12连接MySQL问题数据库就会发现可以连接成功了