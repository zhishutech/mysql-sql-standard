# 类型溢出

以MySQL5版本，int类型为例：
# 建表
root@localhost(test2)14:46>create table test2 (a int(10) UNSIGNED);
Query OK, 0 rows affected (0.12 sec)
# 插入数据
root@localhost(test2)14:56>insert test2 values (10);
Query OK, 1 row affected (0.00 sec)
# 模拟更新溢出
root@localhost(test2)14:56>update test2 set a=a-11;
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1
# 查看warnings
root@localhost(test2)14:57>show warnings;
+---------+------+--------------------------------------------+
| Level   | Code | Message                                    |
+---------+------+--------------------------------------------+
| Warning | 1264 | Out of range value for column 'a' at row 1 |
+---------+------+--------------------------------------------+
1 row in set (0.00 sec)
# 确定实际得到的值已经溢出
root@localhost(test2)14:57>select * from test2;
+------------+
| a          |
+------------+
| 4294967295 |
+------------+
1 row in set (0.00 sec)
# 清理数据
root@localhost(test2)14:59>delete from test2;
Query OK, 1 row affected (0.00 sec)
# 模拟插入溢出
root@localhost(test2)14:59>insert test2 values (-1);
Query OK, 1 row affected, 1 warning (0.00 sec)
# 查看warnings
root@localhost(test2)14:59>show warnings;
+---------+------+--------------------------------------------+
| Level   | Code | Message                                    |
+---------+------+--------------------------------------------+
| Warning | 1264 | Out of range value for column 'a' at row 1 |
+---------+------+--------------------------------------------+
1 row in set (0.00 sec)
# 确定实际得到的值已经溢出
root@localhost(test2)14:59>select * from test2;
+------+
| a    |
+------+
|    0 |
+------+
1 row in set (0.00 sec)
# 原因
int占用4个字节，而int又分为无符号型和有符号性。对于无符号型的范围是0 到 4294967295；有符号型的范围是-2147483648 到 2147483647。
举一反三，其他类型都可能有类似问题，均需要考量。
# 控制方法
可以通过sql_mode参数控制，但一般建议程序控制，比如：对表单项的值进行校验。


