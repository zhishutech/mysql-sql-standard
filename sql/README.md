# SQL规范

## SQL规范的目的
通过约定，统一项目中的SQL语句设计编写，从而达到：

1. 提高数据库系统的处理效率
2. 避免数据库不必要的死锁及资源浪费
3. 提高应用系统的数据库升级方便 

## SQL语句约定
1. 在MySQL中SQL语句一般不区分大小写，全部小写
2. sql语句在使用join, 子查询一定先要进行explain确定执行计划
3. 为每个业务收集sql list.

## SQL安全
- SQL注入常见套路，参考：[https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF)

## 参考备注

在MySQL中，数据库对应于操作系统数据目录中的目录。 数据库中的每个表对应于数据库目录中的至少一个文件（并且可能更多，取决于您使用的存储引擎）。 触发器也对应于文件。 因此，如果底层操作系统区分大小写，那么在数据库中，表和触发器名称也区分大小写。 也就是说这些名称在 Windows 中不区分大小写，但在大多数 Unix 中都区分大小写。 一个明显的例外是 macOS，它基于 Unix，但使用不区分大小写的默认文件系统类型（HFS +）。 但是，macOS 也支持 UFS 卷，与任何 Unix 一样区分大小写。建议在配置文件中设置 lower_case_table_names=1，并且在创建数据库表时使用小写。编写 SQL 语句时建议全部使用小写。

