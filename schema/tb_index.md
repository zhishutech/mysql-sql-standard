# 索引设计

1. 非唯一索引按照“i_字段名称_字段名称[_字段名]”进行命名。 
2. 唯一索引按照“u_字段名称_字段名称[_字段名]”进行命名。 
3. 索引名称使用小写。 
4. 索引中的字段数不超过5个。 
5. 唯一键由3个以下字段组成，并且字段都是整形时，使用唯一键作为主键。 
6. 没有唯一键或者唯一键不符合5中的条件时，使用自增（或者通过发号器获取）id作为主键。 
7. 唯一键不和主键重复。 
8. 索引字段的顺序需要考虑字段值去重之后的个数，个数多的放在前面。 
9. ORDER BY，GROUP BY，DISTINCT的字段需要添加在索引的后面。 
10. 单张表的索引数量控制在5个以内，若单张表多个字段在查询需求上都要单独用到索引，需要经过DBA评估。查询性能问题无法解决的，应从产品设计上进行重构。
11. 使用EXPLAIN判断SQL语句是否合理使用索引，尽量避免extra列出现：Using File Sort，Using Temporary。
12. UPDATE、DELETE语句需要根据WHERE条件添加索引。 
13. 对长度大于50的VARCHAR字段建立索引时，按需求恰当的使用前缀索引，或使用其他方法。
14. 下面的表增加一列url_crc32，然后对url_crc32建立索引，减少索引字段的长度，提高效率。 

    ```
    CREATE TABLE all_url(ID INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    url VARCHAR(255) NOT NULL DEFAULT 0,      
    url_crc32 INT UNSIGNED NOT NULL DEFAULT 0,
    index idx_url(url_crc32));
    ```
    
15. 合理创建联合索引（避免冗余），(a,b,c) 相当于 (a) 、(a,b) 、(a,b,c)。 
16. 合理利用覆盖索引。

