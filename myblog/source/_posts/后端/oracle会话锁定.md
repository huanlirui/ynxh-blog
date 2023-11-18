用 navicat 执行一个耗时很长的函数。

结果 navicat 闪退了。导致这个会话没了。

执行这个语句查看：

```sql
select b.username,b.sid,b.serial#,logon_time
from v$locked_object a,v$session b

where a.session_id = b.sid order by b.logon_time;

```

这个SQL查询语句是用来查找当前被锁定的对象及其相关信息。它联结了v$locked_object和v$session视图，通过session_id与sid进行匹配来获取锁定对象的会话信息。

这个查询语句会返回被锁定对象的用户名、会话ID（SID）、序列号（SERIAL#）和登录时间（LOGON_TIME），并按照登录时间进行排序。

请注意，v$locked_object和v$session是Oracle数据库中的系统视图，用于提供关于锁定对象和会话的信息。

```sql
-- alter system kill session 'sid,serial#;

alter system kill session '1352,13371';
```

杀死会话ID（SID）为1352，序列号（SERIAL#）为13371的会话。执行这个语句将会终止该会话，并释放相关的资源。
