---
cover: /img/mysql.jpeg
---

错误信息：
1055 - Expression #3 of SELECT list is not in GROUP BY clause and contains nonaggregated column ‘fbjs.mscc.ContactTime’ which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by, Time: 0.000000s

报错原因：
在 mysql5.7 以上的版本中，对于 group by 的这种聚合操作，如果在 select 中的列，没有在 group by 中出现，那么这个 SQL 是不合法的，因为列不在 group by 的从句中，所以对于设置了这个 mode 的数据库，在使用 group by 的时候，就要用 MAX()，SUM()，ANT_VALUE()的这种聚合函数，才能完成 GROUP BY 的聚合操作。

修改数据库配置文件
修改/etc/my.cnf，将 sql_mode=中的 only_full_group_by 给删掉
如果没有 my.cnf 文件则创建一个，mysql 的配置文件路径查找优先级为/etc/my.cnf,/etc/mysql/my.cnf,/usr/local/etc/my.cnf，通过 Homebrew 安装的 my.cnf 放在/usr/local/etc/中。

配置文件加入：
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

重启即可mysql.server restart