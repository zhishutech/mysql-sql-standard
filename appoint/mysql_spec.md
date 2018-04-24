# MySQL特点


* mysql是单进程多线程，不像Oracle那样是多进程的
* 每个mysql内部线程同时只能用到一个逻辑cpu线程
* 每个SQL同时只能用到一个逻辑CPU线程
* 无执行计划缓存（无类似ORACLE的library cache），不过MySQL的执行计划解析比较轻量级，效率还不错，这方面不会是瓶颈
* query cache的更新需要持有 全局mutex，数据有任何更新都需要等待该mutex，效率低，且整个表的query cache也会失效，因此，强烈建议关闭query cache
* 没有thread pool时，如果有瞬间大量连接请求，性能会急剧下降


