在MySQL中MVCC是如何实现的，以及如何判断记录的可见性


回滚段

InnoDB也是采用回滚段的方式构建old version记录，这跟Oracle方式类似。

todo 找到更新链条

记录的DB_ROLL_PTR指向最近一次更新所创建的回滚段；每条undo log也会指向更早版本的undo log，
从而形成一条


分配rollback segment




MVCC概念

    多版本控制（Multiversion Concurrency Control）: 指的是一种提高并发的技术

    MVCC只在 Read Committed 和 Repeatable Read两个隔离级别下工作。其他两个隔离级别和MVCC不兼容

MVCC 的实现依赖：隐藏字段、Read View、Undo log。

    隐藏字段
        
        DB_TRX_ID（6字节）最近一次对本记录行做修改（insert | update）的事务ID
        
        DB_ROLL_PTR（7字节）回滚指针，指向当前记录行的undo log
        
        DB_ROW_ID（6字节）随着新增记录单调自增的行ID 这个跟MVCC没多大关系
    
    Read View （跟视图，Snapshot 一个概念）
    
        作用：主要用来做可见性判断
        
        Read View 结构
        
        low_limit_id：目前出现过的最大ID+1， 下一个将被分配的事务ID
        
        up_limit_id：活跃事务列表trx_ids中最小的事务ID    
        
        trx_ids：Read View创建时其他未提交的活跃事务ID列表    
        
        creator_trx_id：当前创建事务的ID
        
    Undo log
    
        Purge线程：为了实现InnoDB的MVCC机制，更新或者删除操作都只是设置一下旧记录的deleted_bit，并不真正将旧记录删除

RR和RC的Read View产生区别

    在innodb中的Repeatable Read级别, 只有事务在begin之后，执行第一条select（读操作）时, 才会创建一个快照(read view)，将当前系统中活跃的其他事务记录起来；并且事务之后都是使用的这个快照，不会重新创建，直到事务结束。
    
    在innodb中的Read Committed级别, 事务在begin之后，执行每条select（读操作）语句时，快照会被重置，即会重新创建一个快照(read view)。
    
    