
基本文件结构

关键结构体

分配回滚段

使用回滚段

如何写入undo日志

事务Prepare阶段

事务Commit

    入口函数：trx_commit_low-->trx_write_serialisation_history

事务回滚
    
    入口函数为：row_undo_step --> row_undo

多版本控制

基本概念

    提供回滚和多个行版本控制(MVCC)。

    undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，
    undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录
    当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚
    
undo log的存储方式
    
    innodb存储引擎对undo的管理采用段的方式。rollback segment称为回滚段，
    每个回滚段中有1024个undo log segment。

