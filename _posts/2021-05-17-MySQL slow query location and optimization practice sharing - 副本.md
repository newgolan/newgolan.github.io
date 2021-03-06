---
layout:     post
title:      "Mysql慢查询定位和优化实践分享"
subtitle:   "MySQL slow query location and optimization practice sharing"
date:       2021-05-17 00:00:00
author:     "Axming"
catalog: true
header-style: text
tags:
  - MySql
---

调优目标：提高io的利用率，减少无谓的io能力浪费。

1、打开慢查询日志定位慢sql：

my.cnf:

slow_query_log

slow_query_log_file=mysql.slow

long_query_time=2(超过2s的sql会记录下来)

cacti监控工具

sed -n '/ 2017-04-08 20:50:37 /,/end/p' mysql.slow>slow.log

mysqldumpslow -s t -t slow.log

2、sql调优方法：

（1）not in子查询优化

      尽量避免子查询
    
      select * from a where id not in（select id from b）;
    
      select * from a where not exists（select id from b WHERE id ='100'）
    
      建议使用表连接：
    
      select * from a left join b on a.id=b.id WHERE b.id='100'
    
      测试如下：第三条语句效率最高
    
    	<1>吞吐量197
    
    	<2>吞吐量211
    
    	<3>吞吐量305

（2）模式匹配like'AA%'优化

    避免使用like'%100%'不能用到索引。建议使用like'100%'可以用到索引。

SELECT * FROM a查询不使用索引。

建议修改为（id已经建立索引）：

SELECT * FROM a  INNER JOIN (SELECT id FROM b ORDER BY created DESC LIMIT 10000,       10) AS  page USING(id)

   <1>吞吐量73

   <2>吞吐量906

（3）limit分页优化

      SELECT id FROM a ORDER BY id DESC LIMIT 1000, 10
    
      建议修改为(上面是全表扫描，修改为范围扫描)：
    
      SELECT id FROM a  where id>1000 ORDER BY id LIMIT 10。
    
      或者先使用连接取出十条数据：
    
     SELECT * FROM article  INNER JOIN (SELECT id FROM article ORDER BY created DESC LIMIT 1000, 1) AS page USING(id) LIMIT 10
    
    <1>tps828
    
    <2>优化后876
    
    <3>优化后862

（4）count (*)优化

      select count(DISTINCT id) from a;
    
    建议优化为（先使用索引排重再统计）：

   SELECT COUNT(1) FROM (SELECT COUNT(DISTINCT id) FROM a) AS page

<1>TPS为430

<2>TPS为440

（5）or条件优化

      select * from a where id=1 or name='2'
    
    建议优化为(使用到索引):
    
      select * from a where id=1 union all select * from a where name='2'

<1>TPS为105

<2>TPS为364

（6）使用ON DUPLICATE KEY UPDATE子句(主键冲突就updateid+1，不冲突就insert)

      insert into a(id,name) values('1','2') ON DUPLICATE KEY UPDATE id=id+1

（7）不必要的排序

      SELECT COUNT(1) FROM (SELECT COUNT(DISTINCT id) FROM a  ORDER BY id DESC) AS page
    
      修改为：
    
      SELECT COUNT(1) FROM (SELECT COUNT(DISTINCT id) FROM a)

<1>TPS为434

 <2>TPS为444

（8）不必要的镶嵌select查询

      SELECT * FROM (SELECT a.id,a.name FROM a,b  WHERE a.id=b.id) AS page
    
      修改为：
    
      select  a.id,b.id,a.name from a join b on a.id=b.id

<1>TPS为633

<2>TPS为668

（9）不必要的自连接

      select id1 from a  JOIN (select id2 from a) ON id1=id2

（10）where子句替换having子句

      having子句用于一些集合函数的比较，如count().
    
      select * from a group by id having id>1 and id<10 limit 3;
    
      修改为:
    
      select * from a where id>1 and id<10 group by id limit 3;

以上调试可以使用执行计划：

      explain select * from innodb_index_stats a where id like '111%';  select_type显示为简单查询；
      
       type显示使用了index。


[张明远]:      https://zmy1123347389.github.io/