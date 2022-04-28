## **7 有用的系统函数**

```
查看当前日志文件lsn位置： 
select pg_current_xlog_location(); 
select pg_current_wal_lsn(); 


当前xlog buffet 中的insert位置，注意和上面pg_current_xlog_location()的区别： 
select pg_current_xlog_insert_location(); 


查看某个lsn对应的日志名： 
select pg_xlogfile_name(lsn)
select pg_walfile_name(lsn)


查看某个lsn在日志中的偏移量： 
select pg_xlogfile_name_offset('lsn'); 
select pg_walfile_name_offset('lsn'); 


查看两个lsn位置的差距： 
select pg_xlog_location_diff('lsn','lsn');
select pg_wal_lsn_diff('lsn','lsn');


查看备库接收到的lsn位置： 
select pg_last_xlog_receive_location(); 
select pg_last_wal_receive_lsn(); 


查看备库放回lsn位置： 
select pg_last_xlog_relay_location(); 
select pg_last_xact_replay_timestamp(); 


创建还原点： 
select pg_create_restore_point('20201111'); 

查看表的数据文件路径，filenode: 
select pg_relation_filepath('test'::regclass); 
select pg_relation_filenode('test'); 


查看表的oid: 
select 'test'::reglass::oid; 

查看当前会话pid: 
select pg_backenk_pid(); 

生成序列： 
select gernate_series(1,8,2); 

生成uuid(pg13新特性） 
select gen_randonm_uuid(); 

重载配置文件信息 
select pg_reload_conf(); 

查看数据库启动时间： 
select pg_postmaster_start_time(); 

查看用户表、列等权限信息： 
select has_any_column_privilege(user, table, privilege); 
select has_any_column_privilege(table, privilege); 

查看当前快照信 
select txid_current_snapshot() 

切换一个运行日志： 
select pg_rotate_logfile(); 

暂停、恢复回访进程： 
select pg_xlog_replay_pause(); 
select pg_xlog_replay_resume(); 

导出一个快照： 
select pg_export_snapshot(); 

查看对象的大小信息： 
select pg_relation_size(); 
select pg_table_size(); 
select pg_total_relation_size(); 

物理、逻辑复制槽： 
pg_create/drop_physical_replication_slot(slotname); 
pg_create_logical_replication_slot(slotname, decodingname); 
pg_logical_slot_get_changes(); 
```

