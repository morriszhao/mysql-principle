# 事务

[TOC]

事务是啥就不解释了  百度一搜一大把、暂时只有Innodb 引擎支持；



## ACID

- ### Atomicity 	 原子

  - 一个事务中的多条sql语句  要么都成功 要么都失败 不是1就是0  只有这两种情况 绝不可能存在第三种

- ### Consistency      一致

  - 事务的操作必须满足现实世界的约束、 比如红绿灯只有三种颜色、 房价不可能为负、 如果我们执行的事务破坏了上述情况、就说破坏了事务的一致性

- ### Isolation           隔离

  - 牵涉到四种隔离级别、 太麻烦

- ### Durability         持久

  - 本次事务对数据所做的修改应该进行落盘、不论之后发生什么事情，本次的操作的数据都不应该被影响

    



## 具体操作

​	开启

```
begin
```

​	提交

```
commit
```

​	回滚

```
rollback
```

​	需要注意的是   mysql中的事务默认是自动提交的  

```sql
mysql> SHOW VARIABLES LIKE 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)
```

可以看到它的默认值为`ON`，也就是说默认情况下，如果我们不显式的使用`START TRANSACTION`或者`BEGIN`语句开启一个事务，那么每一条语句都算是一个独立的事务，这种特性称之为事务的`自动提交`

## 保存点

​	事务对应的数据库语句中打几个点，我们在调用`ROLLBACK`语句时可以指定回滚到哪个点，只撤销部分改动； 而不是回到最初的原点

​	lavaral框架的嵌套事务 就是使用的save point 实现；	



## 隐式提交

​	当我们使用begin 手动开启一个显示事务、或者将系统参数autocommit设置为off时、 事务就不会自动提交、需要我们手动的输入commit 命令、事务才会提交； 但是 如果我们输入咯某些语句后、事务就会悄悄的提交、 就像我们输入咯commit一样

​	那有哪些操作会让事务隐式提交呢？ 

### 	DDL

​		创建新表或者修改表结构(create table  ...  or alter table )

### 	嵌套事务

​		一个事务还没提交的时候  开启了另一个事务、 那么上一个事务会自动提交；如：

```sql
BEGIN;

SELECT ... # 事务中的一条语句
UPDATE ... # 事务中的一条语句
... # 事务中的其它语句

BEGIN; # 此语句会隐式的提交前边语句所属于的事务
```

​	



# redo

​	 我们开启事务 执行以下sql语句

```
begin;
update table set field = 'zhangsan';
commit;
```

​	需要注意的是  为咯提高效率； mysql是使用了buffer pool的； 也就是说 mysql将pagesize大小的数据从磁盘加载到内存； 经过sql命令修改过后、 修改的只是内存中的值、 这个时候数据还未刷新到磁盘，  也就是说这个时候磁盘上的值 还是原来未改动过的值； mysql会在后续将内存中修改的值 刷新到磁盘 保证数据的持久性；



但是、  如果在我们执行咯commit操作   内存已经修改了数据  但还未同步到磁盘时、  遇到系统断电 或者别的不可抗的故障时、  那我们的sql所作的修改不就丢失了、   如何保证事务的持久性呢？

​	

## 	原理

​	 redo日志会把事务在执行过程中对数据库所做的所有修改都记录下来，在之后系统奔溃重启后可以把事务所做的任何修改都恢复出来、 依次来保证事务的持久性；

## 	实现

​	mysql在启动的时候  会根据系统参数innodb_log_buffer_size 来申请一个固定大小的内存块；然后在执行事务的同时、 将sql语句记录到此内存块当中； 然后mysql会开一个线程一直从此内存块将数据刷新到磁盘 ；

### 	redo日志落盘时机

1. `log buffer`空间不足时

2. 事务提交时

   1. 事务提交时  会将当前事务的sql语句同步到磁盘

3. 后台线程一直刷

   1. 后台有一个线程 固定时间(1s)刷新到磁盘

   

     



​	为什么 不直接将sql语句修改的内容刷新到磁盘、 而使用刷新内存加记录日志的方式？

1. mysql以pasesize进行数据管理  pagesize一般为16kb  我们修改的一条数据可能1kb都不到、 会做很多的无用功

2. 一次事务中 执行的sql语句   可能发生在不一样的数据页、 操作系统会执行很多的随机IO  随机IO的速度很慢

3. 使用redo日志的优点是  redo日志是顺序IO  顺序IO的速度 基本上可以比肩内存IO的速度 提高速度（kafka也是使用顺序IO 来提高速度）

   



# undo

 我们开启事务 执行以下sql语句

```
select name from table where id =1 ;
#name='wangwu'

#执行事务修改
begin;
update table set field='lisi' where id = 1;
select name from table where id =1;
#name='lisi';
rollback;

select name from table where id = 1;
#name='wangwu'

```



​	我们先查看id=1的数据  name值为wangwu  然后我们开启事务 执行咯两次修改 、在rollback  然后查看name 发现还是wangwu   mysql是怎样实现的？

​	 mysql在执行一条sql语句的时候  将数据在内存中进行修改、  也就是说在我们执行过

```
update table set filed='lisi' where id =1;
```

​	此sql后  内存中的值已经是变化后的值咯；  那rollack的时候  还怎么恢复到原先的值呢？





## 	原理

​		mysql会在执行sql执行 将原始数据进行保存  存在undo日志中  以便在rollback的时候 从undo日志中恢复数据





## 	实现

​	undo日志示意图：

![image_1d8po6kgkejilj2g4t3t81evm20.png-81.7kB](https://user-gold-cdn.xitu.io/2019/4/19/16a33e277a98dbec?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

事务中的每一次修改  mysql都会记录到undo日志中  形成这样一个版本链   在rollback的时候  回滚到具体版本;



## mvcc

   MVCC（多版本并发控制）的实现原理   也是根据undo日志实现