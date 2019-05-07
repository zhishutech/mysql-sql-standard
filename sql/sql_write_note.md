# SQL编写规范36条
以后可能还会增多，也可以考虑做一下减法。

1. 去掉不必要的括号

    ```
    如：      ((a AND b) AND c OR (((a AND b) AND (c AND d)))) 
    修改成    (a AND b AND c) OR (a AND b AND c AND d)
    ```

2. 去掉重叠常量

    ```
    如：      (a<b AND b=c) AND a=5
    修改成    b>5 AND b=c AND a=5
    ```
    
3.	去除常量条件(由于常量重叠需要)

    ```
    如：      (B>=5 AND B=5) OR (B=6 AND 5=5) OR (B=7 AND 5=6)
    修改成    B=5 OR B=6
    
    **郑松华**注解 ：但是必须注意 or 两边必须用括号括起来，不然会导致结果不一致
    
    ```
4. 去掉无意义的连接用条件

    ```
    如：1=1，2>1，1<2等   直接从
    
    子句中去掉。
    **郑松华**注解 ：但是对于1=1 有可能是开发为了动态添加sql 标准化而写 
    如 select * from emp where 1=1 and emp_no=10001 and ….  这样后面都是and 开始 
    ```
    
5.	开发过程中不使用拼字符串的方式来完成where子句

6.	多使用等值操作，少使用非等值操作

    ```
    WHERE条件中的非等值条件（IN、BETWEEN、<、<=、>、>=）会导致后面的条件使用不了索引，因为不能同时用到两个范围条件。
    郑松华注解 ：对这条有疑问 因为一个sql 的一个表来说只分access 和 fiter 只要使用了相应的选择率好的索引过滤了大部分数据对后面
    作为fiter 条件来说没有大问题 
    ```
7.	常数表优先，字典表或小表其次，大表最后

    ```
    常数表指：空表或只有1行的表。与在一个PRIMARY KEY或UNIQUE索引的WHERE子句一起使用的表。如：
    SELECT * FROM t WHERE primary_key=1;
    SELECT * FROM t1,t2 WHERE t1.primary_key=1 AND t2.primary_key=t1.id;
    字典表指：小数量的行。如：自定义的自增字段表，而不使用MySQL的AUTO_INCREMENT。
    郑松华注解 ：SQL的执行计划是基于cost的 除非强制使用hint 所以无法控制
    上面的可以改为 表的连接条件最好是查询结果集最少的为驱动表 后续表要有良好的索引
    ```
8.	减少或避免临时表

    ```
    如果有一个ORDER BY子句和不同的GROUP BY子句，或如果ORDER BY或GROUP BY包含联接队列中的第一个表之外的其它表的列，则创建一个临时表。
    郑松华注解 ：这里说的临时表应该是执行计划中extra部分的 using temporary 和 using filesort 关键字 
    ```
9.	where子句中的数据扫描别跨越表的30%

    ```
    比如：where primary_key <> 1或者primary_key not in(…)，这样跨表的数据肯定超过30%了。
          where status=1，其中1值非常少，主要是0值，比如一个表的记录删除用了一个状态位，而删除的记录又比较少。
    郑松华注解 ：这个应该讲的是索引的选择率问题 ，如果是删除数据的话 大量删除考虑创建一个临时表 改表名的方法或者
    分段删除的方法 最好不要一次性 进行大量删除
    ```
10.	where子句中同一个表的不同的字段组合建议小于等于5组，否则考虑业务逻辑或分表

    ```
    郑松华注解 ：只要有一个进行好的选择率的索引条件 有几个都无所谓
    ```
11. 不使用is null或is not null，字段设计时建议not null，若麻烦可折中考虑给一默认值

12. 使用like时，%不要放在首字符位置。

    ```
    如果%必须放在首字符位置
    郑松华注解 ：这是基于有索引而言，记住只是字符类型 如果是别的数据类型即使把%放在后面一样不能使用索引
    ```
13. 值域比较多的表字段放在前面
    
    ```
    比如：id，date字段放在前面，而status这样的字段放在后面，具体的可以通过执行计划来把握。
    郑松华注解 ：不知道作者想表达意思 ？ 如果是SQL中的select 列中的话 上面的没有关系 因为只跟是否回表有关系
    @松华，主要是想表达，把过滤性好的条件放在联合索引的前面，这样过滤效果会更好些。
    8.0的 skip index scan 可以反过来用，但是它的局限性很大，需要不能回表等等
    ```
14. 表字段组合中出现比较多的表字段放在前面

    ```
    方便综合评估索引，缓解因为索引过多导致的增删改的一些性能问题。
    ```
15. 表字段不能有表达式或是函数

    ```
    如：where abs(列)>3或where 列*10>100
    郑松华注解 ：这是基于该列有索引 如果使用上述方法会导致不能使用索引
    5.6的时候 是没有办法的
    5.7的时候 可以使用虚拟列
    8.0开始 支持函数索引 
    ```
16. 注意表字段的类型，避免表字段的隐示转换

    ```
    比如：c1列为varchar，如果where c1=1，建议后面整数类型的值加上引号，例如：where c1='1'。
    郑松华注解：如上面的情况之后 额外注意 表或者数据库的字符集，例如列的数据类型是一样的 但是表或者数据库的字符集不一致的话 也会出现
    隐士转换
    ```

17. 考虑使用union all，少使用union，注意考虑去重

    ```
    union all不去重，而少了排序操作，速度相对比union要快，如果没有去重的需求，优先使用union all。
    郑松华注解 ：MySQL 5.6和5.7对union all 的执行计划是不一样的注意把握 5.6的union all 跟union 一样 在执行计划中会产生 <union reslut> 
    会在内存中使用临时表 ，而extra部分不出现using temp相关的提示，使用内存临时表就跟
	数据库的tmp table size 和 heap size 有关。 
    ```
    
18. 不同字段的值or或in大于等于3次，考虑用union all替换；同一字段的值or用in替换

    ```
    Select * from opp WHERE phone=‘12347856' or phone=‘42242233';
    考虑用
    Select * from opp WHERE phone in ('12347856' , '42242233');
    
    Select * from opp WHERE phone='010-88886666' or cellPhone='13800138000'; 
    考虑用
    Select * from opp WHERE phone='010-88886666' 
    union all
    Select * from opp WHERE cellPhone='13800138000';
    郑松华注解 ：MySQL 5.6 和5.7当中对in 的处理机制有点不一样 注意把握
    尤其是 不同的列的or 条件的时候 有可能一个索引满足不了 ，这时候 需要union all 分开来满足不同的索引 
    ```

19. 用Where子句替换HAVING子句

    ```
    select id,count(*) from table group by id having age>=30 order by null;
    考虑用
    select id,count(*) from table where age>=30 group by id order by null;
    郑松华注解：上面的第一个sql 本身就不严谨 有可能出现执行计划不同结果不同
    ```
20. 对同一表的order by和group by操作分别小于3组，否则考虑业务逻辑或分表
21. 尽量使用主键进行update和delete
22. 小心text/blobs等大字段，如果确实不需要这样的大字段，则不用放入sql语句中，避免产生过多额外I/O读。
    
    ```
    郑松华注解：有可能产生行链接问题，还有排序的时候产生大量消耗
    ```
23. 使用INSERT ... ON DUPLICATE KEY update (INSERT IGNORE)来避免不必要的查询
24. limit N以及limit M,N场景中，不管是M还是N的值都不宜过大（一般不超过一万）。当M（起始值）较大时，建议用延迟关联的方式优化，例如：
```
SELECT * FROM (SELECT * FROM `t1` WHERE id > ( SELECT id FROM `t1` ORDER BY id DESC LIMIT 935510, 1) LIMIT 10) t ORDER BY id DESC;

或
SELECT * FROM `t1` INNER JOIN ( SELECT id FROM `t1`ORDER BY id DESC LIMIT 935500,10) t2 USING (id);
```
25. 减少或避免排序，如：group by语句中如果不需要排序，可以增加order by null
    ```
    郑松华注解：8.0 之前有group by 是有排序的 但是8.0开始 group by 没有排序
    但是如果group by 的列是索引的前导列 那么还是有排序的，但是extra部分是没有using file sort 
    ```
26. 增删改语句中不使用不确定值函数和随机函数，如：RAND()和SYSDATE()等。
    ```
    郑松华注解：如果非得 使用， 需要提前进行一次子查询 查询使它固定，然后以调用的形式使用
    ```
27. INSERT语句使用batch提交（INSERT INTO table VALUES(),(),()„„），values的个数不超过500。
28. 避免使用存储过程、触发器、函数、UDF、events等，容易将业务逻辑和DB耦合在一起，并且MySQL的存储过程、触发器、函数、UDF、events中存在一定的bug。
29. 避免使用超过3次的JOIN查询。
30. 使用合理的SQL语句减少与数据库的交互次数。

    ```
    INSERT ... ON DUPLICATE KEY UPDATE
    REPLACE INTO、INSERT IGNORE 、INSERT INTO VALUES(),(),()
    UPDATE … WHERE ID IN(10,20,50,…)
    ```
31. 减少使用视图，避免复杂的语句。
    ```
    郑松华注解：视图有可能导致 外部参数进不去的情况 这样会导致sql 运行缓慢
    mysql 视图创建的时候 有两个选项，大家查阅不同之处 然后根据实际情况使用 
    ```
32. SQL语句中IN包含的值尽量不超过200个。
    ```
    郑松华注解： in 在5.6 和5.7 有不同之处 ，5.7对 隐士转换有一定的缓解
    但是还是有问题 ，强调 where a in ( 1) 这种案例中 a 和 in () 里面的数据类型必须保持一致性，
    且in 里面的所有值的数据类型也应保持一致。
    ```

33. UPDATE、DELETE语句不使用LIMIT(binlog格式是statement或是mixed格式时，容易造成主从不一致,binlog_format=row时，请忽略)。有主键id的表WHERE条件应结合主键。
34. 使用prepared statement，可以提供性能并且避免SQL注入。
35. InnoDB表避免使用COUNT(*)操作，计数统计实时要求较强可以使用memcache或者redis，非实时统计可以使用单独统计表，定时更新。
36. 禁止在Update语句，将“,”写成“and”，非常危险。

    ```
    正确示例：update Table set uid=uid+1000,gid=gid+1000 where id <=2 ;
    错误示例：update Table set uid=uid+1000 and gid=gid+1000 where id <=2 ;
    此时“uid=uid+1000 and gid=gid+1000”将作为值赋给uid，并且无Warning！！！
    ```
37 update 语句禁止使用 in ,exists等 尽量使用 join 
