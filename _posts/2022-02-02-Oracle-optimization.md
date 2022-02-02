---
title: "Oracle性能优化"
date: 2022-02-02 21:53:00 +0800
last_modified_at: 2022-02-02 21:53:00 +0800
categories: [IT, Database]
tags: [Oracle,SQl,优化]
---

- [思路](#思路)
- [索引](#索引)
- [分区表](#分区表)
- [DBlink的优化](#dblink的优化)
- [统计信息收集](#统计信息收集)

# 思路 

> 只针对表、索引层面优化，不考虑数据库、操作系统层面优化
> 
对Oracle SQL的优化的古董套路：
1. 查看执行计划，重点关注耗费较大的SQL语句或没有使用索引的语句
2. 对没有使用索引的SQL，检查索引是否合理/检查SQL语句的写法是否存在优化空间
3. 对使用了索引，但是耗费依然很大的SQL语句：
   1. 检查表数据量，表大小超过2G或数据记录超过5千万考虑分区表（大小及数据量可酌情考虑）
   2. 检查索引或SQL写法是否合理，比如字段是否使用函数计算等
4. 检查是否使用了DBlink
5. 部分情况下，统计信息失效，重新收集统计信息


# 索引 

一般情况下，Oracle中表的索引数量通过控制在3-5个以内，组合索引的字段数也应该控制在5个以内，过多的索引对表的写入性能有较大影响。

（我的）索引的设置的一般原则：
    1. 常用的区分度高的查询字段。（举例，一个查询字段能将数据量限制到原表数据量的百分之一以上就是比较好的索引字段）
    2. 合理选取索引类型，唯一、普通、位图等。 

# 分区表

在Oracle 11g（好象是，需确认）之后，支持动态分区表，其语法如下：

关键词： `numtoyminterval` 、`NUMTODSINTERVAL` 
示例：

```sql
--按年创建分区表
create table test_part
(
 ID NUMBER(20) not null,
 REMARK VARCHAR2(1000),
 create_time DATE
)
PARTITION BY RANGE (CREATE_TIME) INTERVAL (numtoyminterval(1, 'year'))
(partition part_t01 values less than(to_date('2018-11-01', 'yyyy-mm-dd')));

```

```sql
--按月创建分区表
create table test_part
(
 ID NUMBER(20) not null,
 REMARK VARCHAR2(1000),
 create_time DATE
)
PARTITION BY RANGE (CREATE_TIME) INTERVAL (numtoyminterval(1, 'month'))
(partition part_t01 values less than(to_date('2018-11-01', 'yyyy-mm-dd')));
--创建主键
alter table test_part add constraint test_part_pk primary key (ID) using INDEX;
```

```sql
--按天创建分区表
create table test_part
(
 ID NUMBER(20) not null,
 REMARK VARCHAR2(1000),
 create_time DATE
)
PARTITION BY RANGE (CREATE_TIME) INTERVAL (NUMTODSINTERVAL(1, 'day'))
(partition part_t01 values less than(to_date('2018-11-12', 'yyyy-mm-dd')));
```


# DBlink的优化

如果条件允许，尽量不使用dblink，如果必须使用的话，可以在SQL语句中添加hint：
`/*+remote_mapping(db_link)*/`或`/*+driving_site(table_name)*/` 
用法:   /*+driving_site(table_name)*/  : table_name 一般是大表！这样一来就可以避免大表所在库的全表扫描，查询速度将成级数级提高。

```sql
select /*+driving_site(main)*/  a.*,b.* from A.a main@BigTableDB,B.b minor 
where main.id=minor.id and .......
```

# 统计信息收集
注：请不要使用下面命令收集分区表的统计信息（信息收集不完整）。

```sql
analyze table scott.emp compute statistics; --收集所有的统计信息和直方图信息，包括表、列、索引。
analyze table scott.emp compute statistics for table; --收集emp表的统计信息，不含列、索引统计信息和直方图。
analyze table scott.emp compute statistics for all columns;  --收集所有列的统计信息和直方图（超大表较耗资源，因为只要列中有非空值，那么就会收集这个列的统计信息和直方图）。
analyze table scott.emp compute statistics for all indexed columns;  --收集所有索引列的统计信息和直方图。
analyze table scott.emp compute statistics for all indexes; --收集所有索引统计信息，不含列的统计信息和直方图。
analyze table scott.emp compute statistics for columns 列1,列2； --收集2个列的统计信息和直方图。
analyze index idx_ename delete statistics; --删除索引idx_ename的统计信息。
analyze table scott.emp delete statistics; --删除表t1所有的表，列，索引的统计信息和列直方图。
analyze table scott.emp estimate statistics sample 15 percent for table; --收集emp表的统计信息，以估算模式采样比例为15%进行收集，不含列、索引统计信息和直方图。
```


