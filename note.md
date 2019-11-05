# note

[TOC]



## 1、优化器

在执行mysql具体语句之前  mysql server层会进行sql语句的优化   mysql内部的优化器会分析sql语句 然后执行它认为的最优sql

比如说我们有一张表student   字段name age  为age建立的索引



我们执行 

select * from student where age = 1 and name = ‘zhangsan’

select * from student where name = ‘zhangsan’ and age = 1



在经过mysql的优化器 优化后 使用explain 分析  可以看到 effect rows 都是1  说明都用到了索引







## 2、最左原则

针对多个字段建立的联合索引  比如 nama,age  建立联合索引 idx_union

在查询时候   

select * from student where name = ‘zhangsan’      命中索引

select * from student where name = ‘zhangsan’ and age =1     命中

select * from student where age = 1                         不能命中

这既是mysql的最左原则； 





## 3、null 与 空值的区别

空值不占用空间  null需要占用额外的空间

null 只能使用is_null  is_not_null  空值可使用name=‘'





## 4、where子句使用咯索引字段 却不能正确命中的情况

 

假设 name 建立咯索引  nickname  未建立索引

select * from student  where name=‘zhangsan’ or nickname=‘lisi’   

​     一个使用到索引字段的where条件 和 未使用到索引的where条件 or 链接时 是无法使用到索引的



## 5、set names utf8   

​    告诉mysql  client 和 server 交互的数据流使用utf8编码    而不是制定mysql的存储引擎的编码方式  mysql 的存储引擎的编码方式在建表的时候指定 





## 6、pdo预处理

$pdo->prepare() ;webserver 提交sql语句给mysql_server 

$pdo->execute();webserver 提交绑定的参数给mysql_server



第一步  mysql进行sql语句的预处理 （分析 优化）

第二部  进行参数匹配；





原理介绍： 

select * from user where name = :name

:name = ‘zhangsan’

:name = “‘zhangsan’ or 1 = 1"





mysql支持预处理，sql已经预编译好了，坑已经挖好了，来一个填一个，不会改变原本sql的意思；

那么后面的or 1=1;--根本不会被mysql编译成执行计划，被当作普通的字符串处理，没任何意义。

通俗的理解，就像是填空题，你只能在括号里面填写内容，并且这句话是预知的，如果这句话的意思都变了，那么就出现异常了，当然这样的理解不准确

获取姓名是（）的用户，

如果是sql注入，那么就变成，

获取姓名是（）或者 1=1 的所有用户

显然，这和我们预先设定的意思是不一样的





###### 需要注意的是  php的pdo不是真正意义上的预处理  需要设置参数 ATTR_EMULATE_PREPARES 为false 





## 7、count(*)  count(1)  count(field)

```
create table #bla(id int,id2 int)
insert #bla values(null,null)
insert #bla values(1,null)
insert #bla values(null,1)
insert #bla values(1,null)
insert #bla values(null,1)
insert #bla values(1,null)
insert #bla values(null,null)
```

使用语句count(*),count(id),count(id2)查询结果如下：

```
select count(*),count(id),count(id2)
from #bla
results 7 3 2
```

结论：

COUNT(expr) ，返回SELECT语句检索的行中expr的值不为NULL的数量     COUNT(*) 的统计结果中，会包含值为NULL的行数。

由于count(*)是sql标准中的统计行数的做法  mysql对count(*) 做咯很多优化  所以尽量使用count(*)  不要使用count(1)  这种



mysql 对count(*) 和 count(1) 的解释：

```
InnoDB handles SELECT COUNT(*) and SELECT COUNT(1) operations in the same way. There is no performance difference.      
```





## 8、如何进行字符的精确匹配

mysql 默认是对字符后面的空格进行忽略的 跟服务器配置 以及mysql版本都没有关系；

```
select * from table where name='zhangsan';

select * from table where name = 'zhangsan     ';
```

可以发现 两条语句查询结果一样

如何实现空格的精确匹配呢？

```
select * from table where name = BINARY 'zhangsan  '
```

