# mysql 初识

[TOC]



## mysql 架构图

![image_1c8d26fmg1af0ms81cpc7gm8lv39.png-97.9kB](https://user-gold-cdn.xitu.io/2018/12/28/167f4c7b99f87e1c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)











## 常用存储引擎

`MySQL`支持非常多种存储引擎，我这先列举一些：

|  存储引擎   |                             描述                             |
| :---------: | :----------------------------------------------------------: |
| `FEDERATED` |                  用来访问远程表（夸库查询）                  |
|  `InnoDB`   |          具备外键支持功能的事务存储引擎（默认引擎）          |
|  `MEMORY`   |              置于内存的表（速度快  不能持久化）              |
|   `MERGE`   | 用来管理多个MyISAM表构成的表集合（相同表结构的多张表当一张表处理  如日志表 log-10,log-11,log-12） |
|  `MyISAM`   |            主要的非事务处理存储引擎（不支持事务）            |





查看mysql支持的存储引擎:

```
SHOW ENGINES;
```

