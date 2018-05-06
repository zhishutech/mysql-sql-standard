# 分页技巧

## 传统分页

select * from table limit 10000,10;
* LIMIT的坑
偏移量越大则越慢
例如 limit 10000,10，则至少需要先扫描前面的10000条记录，然后再往后位移10条记录，效率很差。

## 下面我们演示了几种分页SQL改进方案。


* 分页SQL改进方案一： 利用主键条件，跳过前面N条记录扫描

```    
    select * from table WHERE id>=23423 limit 11; #10+1 (每页10条)
    select * from table WHERE id>=23434 limit 11;
```
    
* 改进方案二：利用子查询优化
   
    ``` 
    Select * from table WHERE id >= ( select id from table limit 10000,1 ) limit 10;
    ```
* 改进方案三：利用JOIN优化

    ```
    SELECT * FROM table INNER JOIN (SELECT id FROM table LIMIT 10000,10) USING (id) ;
    ```
*	改进方案四：直接利用主键ID进行取值（其实和方案二、三类似）

    ```
    程序中先取的主键ID值：select id from table limit 10000,10;
    再把这些值传递到IN子句中：select * from table WHERE id in (123,456…) ;
    ```
## LIMIT分页举例

```
MySQL> select sql_no_cache * from post limit 10,10; 
10 row in set (0.01 sec) 

MySQL> select sql_no_cache * from post limit 20000,10; 
10 row in set (0.13 sec) 

MySQL> select sql_no_cache * from post limit 80000,10; 
10 rows in set (0.58 sec) 
```
**随着LIMIT中OFFSET值的增加，SQL耗时也明显增加**

```
MySQL> select sql_no_cache * from post WHERE id>=323423 limit 10; 
10 rows in set (0.01 sec) 
```

**利用主键ID先进行条件判断，效率提升很高**


```
MySQL> select * from post WHERE id >= ( select sql_no_cache id from post limit 80000,1 ) limit 10 ; 
10 rows in set (0.02 sec)
```

**改成子查询方案，效率也不错**

分页的业务逻辑改进

## Limit分页与缓存结合
类似sina微博的首页，只推荐 最新的500条微博记录，最多10页
 ![](/img/page-1.png)
 
分页样式类似如下：
 
  ![](/img/page-2.png)
  
并且在逻辑中限死，这样就不会出现分页逻辑被爬虫爬死的问题了。

