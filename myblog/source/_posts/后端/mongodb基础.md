---
cover: /img/node.jpeg
---
##mongodb 数据库学习笔记

1. mac 下用 brew 安装。网上例子很多，照着走即可。
2. 默认配置文件地址为/usr/local/etc mongod.conf 访达 shift+win+G 直达
3. 命令台 mongo 即可连接数据库

#### mongodb 数据库创建删除、表（集合）创建删除、数据增删改查

#### mongodb 海量大数据查询优化、 MongoDB 索引、复合索引、唯一索引、 explain 分析查询速度

#### mongodb 开启权限验证、 mongodb 超级管理员 、mongodb 用户权限管理

#### 关系型数据库表（集合）与表（集合）之间的几种关系

`另一篇文章有写`

#### MongoDB 的高级查询 、MongoDB 多表关联查询、aggregate 聚合管道 $project 、$match 、$group、$sort、$limit、$skip、\$lookup 表关联

1. aggregate 聚合管道

> 使用聚合管道可以对集合中的文档进行变换和组合

实际应用：表关联查询、数据的统计

管道常用操作符：

```
$project   增加、删除、重命名字段 (只查出需要的字段)
$match  条件匹配。满足条件的文档才能进入下一阶段
$limit   限制结果数量
$skip   跳过文档数量
$sort   条件排序
$group   条件组合结果
$lookup   用来引入其他集合的数据 (表关联查询)
```

步骤：

1. 从 order 订单表关联 order_item 订单详情表
2. 主表\从表的关联字段为 order_id
3. 从 order_item 表查出的数据存放在 items 中
4. order 查出的数据过滤掉多余字段，只显示 trade_no 和 all_price 和 items 字段
5. 过滤出 all_price 大于 90 的数据
6. 将数据按 all_price 字段排序（从大到小）

示例：

```
db.order.aggregate([
{
      $lookup:
        {
          from: "order_item",
          localField: "order_id",
          foreignField: "order_id",
          as: "items"
        }
   },
{
	$project:{ trade_no:1, all_price:1,items:1 }
},
{
	$match:{"all_price":{$gte:90}}
},
{
	$sort:{"all_price":-1}
},
])

```
