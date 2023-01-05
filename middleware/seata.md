## seata 学习
1. seata首先需要一个seata server 进行协调， 
 - seata server 底层用的netty进行的实现rpc，进行seata的分布式事务的协调
 - 无论 seata 使用的模型是 tcc / xa / saga /ta 模型，都是无法逃脱二阶段的过程，所以核心的设计逻辑为seata的整个协调执行过程
2. seata client 集成于spring framework ，故可以与支持jdbc的OLTP的DATABASE进行分布式事务的管理，只需要globalTransaction注解进行注入即可，
   底层也时用的比较著名的编程模型AOP进行的注入，无论是和dubbo还是grpc 还是springcloud等微服务框架，均可继承，官方的seata的模块中已经基于
   市面常用的微服务框架进行了支持
> Seata 底层设计逻辑是基于分布式事务的session和transactionId以及undolog的状态进行的开发 , undolog的存储支持三种 file database 以及redis, seata server的集群支持常见的注册中心 consul/etcd3/eureka/naocs/redis/sofa/zk

* 一个globalSession中包含了branchsession 即的子操作 源代码 io.seata.server.session.GlobalSession  io.seata.server.session.BranchSession 包含的状态分别对应为 io.seata.core.model.GlobalStatus io.seata.core.model.BranchStatus 分别对应的 事务模型 进行状态的转换变更操作