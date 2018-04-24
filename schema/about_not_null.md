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


