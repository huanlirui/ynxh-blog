# 安装oracle18

> 使用airbyte工具，必须目标源数据库oracle版本大于18 或者12+ 设置字符串长为120

拉镜像

docker pull registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c

docker run -d -p 21521:1521 -p 6500:5500 \
  --name oracle18c \
  -e ORACLE_SID=ORCLCDB \
  -e ORACLE_PDB=ORCLPDB1 \
  -e ORACLE_PWD=PassWord123 \
  -v /data/oracle18/oracle_data:/opt/oracle/oradata \
  registry.cn-hangzhou.aliyuncs.com/zhengqing/oracle18c

怎么建表呢?

新建tablespace
create bigfile tablespace XXX
datafile '/opt/oracle/oradata/bigfiledrg01.dbf' size 100G;

新建用户
create user XXX identified by 123456 default tablespace XXX;

报错说用户名非法

alter session set "_ORACLE_SCRIPT"=true;  

create user XXX identified by 123456;

成功了
