# master_slave

[TOC]

## binlog日志介绍
1、binlog是mysql的二进制日志、  将对mysql的所有操作（增删改查）记录在日志文件中

2、binlog会在每次执行 flush_log 命令时  重新生成一个新的binlog文件 如mysql_binlog_00001;

3、mysql会生成一个binlog_index 文件 来管理当前的binlog日志文件  binlog_index文件记录了有多少个binlog

4、每一次执行数据库的curd语句  binlog会记录

```sql
start position  
curd语句  
end position    
```


5、数据库宕机后  使用mysql binlog日志恢复数据的原理  就是利用mysqlbin 命令读取 mysql的binlog日志   指定start_position  end_position  恢复数据


6、binlog 是记录的二进制文件 不能直接打开 需要使用mysql 提供的mysqlbin 命令打开







## 1、工作流程

master_mysql 开始binlog日志； 启动一个i/0线程 将binlog日志增量跟新 推送到slave_mysql;  slave_mysql会开启两个线程、 一个i/o线程将master_mysql 推送的数据同步到中继日志、  另外一个sql线程将 中继日志中的sql语句同步跟新到数据表  完成整个同步流程；


## 2、关键点
1、master_mysql  slave_mysql  server_id  必须唯一

2、由于采用的是master_mysql 推送的方式  因此在多slave的时候 会加重master_mysql压力  导致同步缓慢

3、master_mysql 推送成功后 slave_mysql 没有同步跟新  可能是slave_mysql sql线程挂掉






