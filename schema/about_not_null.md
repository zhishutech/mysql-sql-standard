# 关于NOTNULL

建议每个字段都设置上NOT NULL属性，可减小存储开销及避免索引失效的问题。
同时，为了避免程序写入失败，还可以增加默认值。如：

```
a1 varchar(32) not null default ‘’
```

或

```
c1 int(10) not null default ‘0’
```

** 特注意 **

* count(*)  会统计NULL的行， count(列名） 不会统计此列为NULL值的行
* count(distinct col) 计算该列除 NULL 之外的不重复行数，注意 count(distinct col1, col2) 如果其中一列全为 NULL，那么即使另一列有不同的值，也返回为 0。

**  旧表新加字段，需要允许为NULL（避免全表数据更新 ，长期持锁导致阻塞）  **

使用pt-osc类工具操作。
