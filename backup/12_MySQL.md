# [MySQL](https://github.com/liu-cj25/blog/issues/12)

# mysql常见问题
## Mysql中有哪几种锁？
1. 行级锁：开销大，加锁慢；会出现死锁，锁粒度最小；
2. 表级锁：开销小，加锁快；不会出现死锁，锁粒度最大
## 简述在MySQL数据库中MyISAM和InnoDB的区别
### MyISAM：
- 不支持事务，每次查询都是原子的
- 支持表级锁，每次操作都对整个表加锁
- 不支持外键
### InnoDb：
- 支持事务，支持事务的四种隔离级别
- 支持行级锁及外键；因此支持写并发
- 主键使用聚簇索引（索引的数据存储数据本身），辅索引的数据存储主键的值。因此从辅索引查找数据，先要通过辅索引找到主键值，再访问主键索引；最好使用自增主键，防止插入数据时，为维持b+树结构，文件的大小调整
  
### 区别
1. innodb支持事务，myisam不支持事务
2. innodb支持行级锁，myisam支持表级锁
3. innodb支持mvcc，myisam不支持mvcc
4. innodb支持外键，myisam不支持外键
5. innodb不支持全文索引，myisam支持全文索引
## Mysql中InnoDB支持的四种事务隔离级别名称，以及逐级之间的区别？
- 读未提交
- 读已提交
- 可重复读
- 串行化
## MySQL中varchar与char的区别以及varchar(50)中的50代表的涵义
- varchar是可变长度，char是固定长度
- varchar(50)的含义为最多存放50个字符，varchar(50)和(200)存储hello所占空间一样，但后者在排序时会消耗更多内存
## 怎样才能找出最后一次插入时分配了哪个自动增量？
- LAST_INSERT_ID将返回由Auto_increment分配的最后一个值

## SQL的执行
1. 用户发送请求、执行sql
2. Mysql数据库服务器：数据库连接池,线程获取sql，sql接口处理接收到的sql，sql查询解析器解析sql，sql优化器选择最优的查询路径，调用存储引擎执行真正的sql
3. update语句在mysql中如何执行：
    - 从磁盘中加载数据到缓冲池
    - 将数据的旧值写入undolog
    - 更新内存数据
    - 写redolog日志
    - 把redolog写入磁盘
4. innodb_flush_log_at_trx_commit：
    - 这个参数为0 的话，提交事务的时候不会把redolog的数据刷入磁盘（宕机会丢失）
    - 这个参数为1 的话，必须把redolog刷入磁盘文件中（宕机不丢失）
    - 这个参数为2的话，提交事务的时候，把redolog日志写入磁盘文件对应的os cache内存缓存中，而不是直接进入磁盘文件，可能1秒后才会把os cache里的数据写入到磁盘文件中（宕机会丢失）

## undo、redo、binlog
- undolog：修改前的数据（可能要回滚的数据）
- redolog：记录下来你对数据修改了什么（宕机要恢复的数据日志，innodb特有）
- binlog，归档日志里面记录的是偏向于逻辑性的日志，类似于"对user表中的id=10的数据进行了更改，更改的值为xx"

## 基于binlog和redolog完成事务的提交

当我们把binlog写入磁盘文件后，接着就会完成最终的事务提交，此时会把本次更新对应的binlog文件和这次更新的binlog日志在文件里的位置，都写入到redolog日志文件中，同时在redolog日志文件里写入一个commit标记。

## 参考资料
- https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247502168&idx=1&sn=ff63afcea1e8835fca3fe7a97e6922b4&scene=21#wechat_redirect
