# NoSQL选择

** 推荐： Redis **

对于Redis高可用选择方面建议：
*  对于单个业务需求空间不大的情况下，推荐： Redis主从+Sentinel结构，该结构更加灵活，运维简单。
*  对于单个业务需求空间比较大，业务层不方面控制Redis分片时，可以考虑Redis Cluster。

目前Redis使用基本是互联网里标配缓存。




