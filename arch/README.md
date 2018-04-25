# 高可用架构
高可用话题是数据库中比较关心的一个话题。

目前生产中推荐的高可用架构：

## 基于复制的高可用

*  keepalived+VIP 最简单，现有业务不用改造，可以实现DB故障自动切换。
*  MHA +VIP 或是服务发现， 在GTID出现以前，对切换一致性要求高的环境，基本都是MHA为主
*  在GTID出现后，就有点落后了，特别是MySQL 5.7的增强半同步+GTID，基本不需要MHA。现在推荐的高可用：
  * [replcation-manager](https://github.com/signal18/replication-manager)  创建独立公司在运作这个软件。  
  * [orchestrator ](https://github.com/github/orchestrator)  现在归到Github支持及开源
  这两个软件现在可以和ProxySQL，Consul这类工具结合，实现平台RDS方式的高可用。
  
## 强一致性高可用

* Percona XtraDB Cluster  （同步复制）
* MySQL Group Replicaton  

在这种结构特别注意： **不能进行多节点同时进行update操作**。
  
  

   


