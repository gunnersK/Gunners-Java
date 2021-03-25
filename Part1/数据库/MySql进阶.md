- 定位事务和锁，可以用下面几个表

  - INFORMATION_SCHEMA.INNODB_TRX

    ```
    TRX_ID 	自增id
    TRX_WEIGHT 	事务权重，反映(但不一定是准确的计数)事务更改的行数和锁定的行数。为了解决死锁，InnoDB选择权重最小的事务作为要回滚的“受害者”
    TRX_STATE 	事务执行状态。允许的值包括运行(RUNNING)、锁等待(LOCK WAIT)、回滚(ROLLING BACK)和提交(COMMITTING)。
    TRX_STARTED 	事务开始时间
    TRX_REQUESTED_LOCK_ID 	事务当前等待的锁的ID，如果TRX_STATE为LOCK WAIT;否则无效。要获取关于锁的详细信息，请将此列与INNODB_LOCKS表的LOCK_ID列关联
    TRX_WAIT_STARTED 	事务开始等待锁的时间，如果TRX_STATE为锁等待(LOCK WAIT);否则无效。
    TRX_MYSQL_THREAD_ID 	MySql事务线程id，要获取关于线程的详细信息，与INFORMATION_SCHEMA PROCESSLIST表的ID列关联
    TRX_QUERY 	事务正在执行的SQL语句
    TRX_OPERATION_STATE 	事务当前操作
    TRX_TABLES_IN_USE 	处理此事务的当前SQL语句使用的InnoDB表的数量
    TRX_TABLES_LOCKED 	当前SQL语句具有行锁(row locks)的InnoDB表的数量(因为这些是行锁(row locks)，而不是表锁(table locks)，所以表通常仍然可以由多个事务读写，尽管有些行被锁定了)
    TRX_LOCK_STRUCTS 	事务保留的锁的数量
    TRX_LOCK_MEMORY_BYTES 	此事务在内存中的锁结构占用的总大小
    TRX_ROWS_LOCKED 	此事务锁定的近似数目或行。该值可能包括物理上存在但对事务不可见的删除标记行
    TRX_ROWS_MODIFIED 	此事务中修改和插入的行数量
    TRX_CONCURRENCY_TICKETS 	指示当前事务在换出之前可以做多少工作的值，由innodb_concurrency_tickets系统变量指定
    TRX_ISOLATION_LEVEL 	事务隔离级别
    TRX_UNIQUE_CHECKS 	是否为当前事务打开或关闭唯一性检查
    TRX_FOREIGN_KEY_CHECKS 	是否为当前事务打开或关闭外键检查
    TRX_ADAPTIVE_HASH_LATCHED 	自适应哈希索引是否被当前事务锁定
    TRX_ADAPTIVE_HASH_TIMEOUT 	是否立即放弃自适应哈希索引的搜索锁存器，还是在来自MySQL的调用之间保留它
    TRX_IS_READ_ONLY 	值为1表示只读事务
    TRX_AUTOCOMMIT_NON_LOCKING 	值1表示事务是一个SELECT语句，它不使用FOR UPDATE或LOCK IN SHARED MODE子句，并且在执行时启用了autocommit，因此事务将只包含这一条语句。当这个列和TRX_IS_READ_ONLY都为1时，InnoDB优化事务，以减少与更改表数据的事务相关的开销。
    ```

  - INFORMATION_SCHEMA.INNODB_LOCKS
  
    ```
    LOCK_ID 	自增id
    LOCK_TRX_ID 	持有锁的事务的ID。要获取关于事务的详细信息，请将此列与INNODB_TRX表的TRX_ID列关联。
    LOCK_MODE 	描述如何获取锁. 可以是 S, X, IS, IX, GAP, AUTO_INC, and UNKNOWN.
    LOCK_TYPE 	锁的类型。 RECORD for a row-level lock, TABLE for a table-level lock
    LOCK_TABLE 	已锁定或包含锁定记录的表的名称
    LOCK_INDEX 	索引名称，仅当LOCK_TYPE为RECORD时；否则无效
    LOCK_SPACE 	如果LOCK_TYPE为RECORD，则锁定记录的表空间ID；否则无效
    LOCK_PAGE 	如果LOCK_TYPE为RECORD，则锁定记录的页码；否则无效
    LOCK_REC 	如果LOCK_TYPE为RECORD，则页内锁定的记录的堆号；否则无效
    ```
  
  - INFORMATION_SCHEMA.INNODB_LOCK_WAITS
  
    ```
    REQUESTING_TRX_ID 	被阻塞的正在请求事务id
    REQUESTED_LOCK_ID 	事务正在等待的锁的ID。要获取有关锁的详细信息，请将此列与INNODB_LOCKS表的LOCK_ID列关联
    BLOCKING_TRX_ID 	阻塞事务的ID
    BLOCKING_LOCK_ID 	事务持有的阻止另一个事务继续进行的锁的ID。要获取关于锁的详细信息，请将此列与INNODB_LOCKS表的LOCK_ID列关联。
    ```
  
  - INFORMATION_SCHEMA.PROCESSLIST
  
    ```
    ID 	标识ID。这与在SHOW PROCESSLIST语句的Id列、Performance Schema threads表的PROCESSLIST_ID列中显示的值类型相同，并由CONNECTION_ID()函数返回
    USER 	发出该语句的mysql用户
    HOST 	发出该语句的客户机的主机名(系统用户除外，没有主机)。
    DB 	默认数据库
    COMMAND 	线程正在执行的命令的类型
    TIME 	线程处于当前状态的时间(以秒为单位)
    STATE 	指示线程正在执行的操作、事件或状态
    INFO 	线程正在执行的语句，如果没有执行任何语句，则为NULL
    ```
  
  - 参考文章：https://mingshan.fun/2019/09/01/transaction-running/

