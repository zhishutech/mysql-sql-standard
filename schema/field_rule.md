# 字段规范

## 写在前面
* InnoDB表是索引聚集组织表（IOT）， 所有的行数据（row data）都是以主键（严格意义讲，是聚集索引）逻辑顺序存储，而二级索引（或称辅助索引，secondary index）的value则同时包含主键。
* InnoDB的最小I/O单位是data page（默认一个data page大小是16KB），在buffer pool中的最小单位是data page（而不是每行数据哦）。因此也可以这么理解，一个data page里的热点数据越多，其在buffer pool的命中率就会越高。
* MySQL复制环境中，如果binlog format是row的，则从库上的数据更新时是以主键为依据进行apply的，如果没有主键则将可能会有灾难性的后果。
* 此外，强烈建议每张表三个必加字段：aid（int/bigint unsigned类型，自增长列，并且作为主键），create_time（timestamp或int unsigned）、update_time（和create_time相同）用于记录行创建时间以及最后更新时间，在业务上以及日常维护上会有很多便利；

## 字段设计参考
1. 每个表建议不超过30-50个字段
2. 优先选择utf8mb4字符集，它的兼容性最好，而且还支持emoji字符。如果对存储容量比较敏感的，可以改成latin1字符集
3. 严禁在数据库中明文存储用户密码、身份证、信用卡号（信用卡PIN码）等核心机密数据，务必先行加密
4. 存储整型数据时，默认加上UNSIGNED，扩大存储范围
5. 建议用INT UNSIGNED存储IPV4地址，查询时再利用INET_ATON()、INET_NTOA()函数转换
6. 如果遇到BLOB、TEXT字段，则尽量拆出去，再用主键做关联
7. 在够用的前提下，选择尽可能小的字段，用于节省磁盘和内存空间
8. 涉及精确金额相关用途时，建议扩大N倍后，全部转成整型存储（例如把分扩大百倍），避免浮点数加减出现不准确问题

## 常用数据类型参考
1. 字符类型建议采用varchar数据类型（InnoDB建议用varchar替代char）
2. 金额货币科学计数建议采用decimal数据类型，如果运算在数据库中完成可以考虑使用bigint存储，单位：分
3. 自增长标识建议采用int或bigint数据类型，如果该表有大量的删除及再写入就使用bigint,反之int就够用
4. 时间类型建议采用为datetime/timestamp数据类型
5. 禁止使用text、longtext等的数据类型
6. 字段值如果为非负数，就加上unsigned定语，提升可用范围
