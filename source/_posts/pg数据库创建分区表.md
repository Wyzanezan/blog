---
title: pg数据库创建分区表
date: 2019-11-29 16:29:00
tags: DB
categories: DB
---

提高应用程序查询数据库效率方面，除了优化程序中的代码，对数据库的管理与维护也是很重要的一方面。比如在开发阶段表中字段的类型、是否创建索引、大表是否创建分区表及是否单独指定表空间等。

<!--more-->

下面以postgresql 10.8(以下简称pg)为例，介绍下在pg中如何创建分区表。

## PostgreSQL中创建分区表

首先，贴出来pg的官方文档，上面讲的还是比较详细的：

[https://www.postgresql.org/docs/10/index.html](https://www.postgresql.org/docs/10/index.html)



pg中有两种分区方式，一个是pg提供的内建分区方式，另一种是自定义创建分区的方式。



### 内建分区方式

内建分区表类型：范围分区，列表分区。下面以范围分区为例。

```sql
-- 创建分区主表，以call_time作为范围分区
create table partition_test_table_master
(
    id  SERIAL not null,
    call_time   date not null,
    session_id  char(64),
    user_input  char(1024),
    intention   char(1024)
) PARTITION BY RANGE (call_time);
```

创建分区表时，也可以使用多个字段作为分区键。如果使用多个字段作为分区键，可能会创建大量子分区，每个子分区占用的空间会变小。
当使用较少的字段作为分区键时，可能会以粗粒度的标准创建若干分区，分区数量也会相对变少，当有查询满足条件时，扫描的分区也会减少。

当插入的数据没有被分到任何分区时，会报错，这时我们可以手工创建一个新分区存放这些数据。

```sql
-- 创建分区表
CREATE TABLE partition_2019_06 
PARTITION OF partition_test_table_master
FOR VALUES FROM ('2019-06-01 00:00:00') TO ('2019-07-01 00:00:00');

CREATE TABLE partition_2019_07
PARTITION OF partition_test_table_master
FOR VALUES FROM ('2019-07-01 00:00:00') TO ('2019-08-01 00:00:00');

-- 创建子分区并指定所在表空间，将大表单独保存在一个表空间中，也可以提高查询效率
CREATE TABLE partition_2019_08
PARTITION OF partition_test_table_master
FOR VALUES FROM ('2019-08-01 00:00:00') TO ('2019-09-01 00:00:00')
TABLESPACE tablespace_test;
```

```sql
-- 为子分区创建索引
CREATE INDEX ON partition_2019_06 (call_time);
CREATE INDEX ON partition_2019_06 (session_id);
```

```sql
-- 向分区表中插入数据
INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES (now(), 'wyzaneitnfyhwyq', '你好', '打招呼');

INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES ('2019-06-01 00:00:00', 'wyzaneitnfyhwyq', '你好', '打招呼');

INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES ('2019-07-01 00:00:00', 'wyzaneitnfyhwyq', '你好', '打招呼');


INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES ('2019-08-23 00:00:00', 'wyzaneitnfyhwyq', '你好', '打招呼');


INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES ('2019-04-23 00:00:00', 'wyzaneitnfyhwyq', '你好', '打招呼');

INSERT INTO public.partition_test_table_master(
    call_time, session_id, user_input, intention)
    VALUES ('2019-10-23 00:00:00', 'wyzaneitnf
```

```sql
-- 当你想删除不需要的数据时，可以直接删除子分区
DROP TABLE partition_2019_06;
-- 或者可以将子分区从当前分区表中移除，但是表中的数据仍然存在，这是一种不错的处理方式
ALTER TABLE partition_test_table_master 
DETACH PARTITION partition_2019_07;
```

```sql
-- 将一个已创建的表加入到子分区中
CREATE TABLE partition_2019_10
  (LIKE partition_test_table_master 
   INCLUDING DEFAULTS INCLUDING CONSTRAINTS);

ALTER TABLE partition_2019_10 ADD CONSTRAINT cons_partition_2019_10
   CHECK ( call_time &gt;= DATE '2019-10-01 00:00:00' 
          AND call_time &lt; DATE '2019-11-01 00:00:00' );

ALTER TABLE partition_test_table_master ATTACH PARTITION partition_2019_10
    FOR VALUES FROM ('2019-10-01 00:00:00') TO ('2019-11-01 00:00:00' );
```

上面创建分区表的缺点
1.必须分别对每个分区创建索引，也就是说不能在所有分区上创建主键，唯一约束，排他性约束
2.分区表不支持主键，也不支持外键索引
3.执行更新操作后，数据不能改变分区
4.行触发器应该单独定义在每个子分区，而不是在分区表上定义



### 自定义分区

使用表继承的方式创建分区表

自定义分区的优势
1.允许子分区含有额外的列
2.可以多继承
3.可以根据用户选择的任何方式进行分区

```sql
-- 创建主表
create table partition_test_table_master_2
(
    id  SERIAL not null,
    call_time   date not null,
    session_id  char(64),
    user_input  char(1024),
    intention   char(1024)
);
```

```sql
-- 创建分区表
CREATE TABLE partition_2019_11 (
    CHECK ( call_time &gt;= DATE '2019-11-01 00:00:00' 
           AND call_time &lt; DATE '2019-12-01 00:00:00' )
) INHERITS (partition_test_table_master_2);

CREATE TABLE partition_2019_12 (
    CHECK ( call_time &gt;= DATE '2019-12-01 00:00:00' 
           AND call_time &lt; DATE '2020-01-01 00:00:00' )
) INHERITS (partition_test_table_master_2);
```

```sql
-- 创建索引
CREATE INDEX idx_partition_2019_11 ON partition_2019_11 (call_time);
```

```sql
-- 创建函数，将对主表的insert操作映射到对应子分区上
-- NEW表示将要插入的那行数据
CREATE OR REPLACE FUNCTION func_partition_insert()
RETURNS TRIGGER AS $$
BEGIN
    IF ( NEW.call_time &gt;= DATE '2019-11-01 00:00:00' AND
         NEW.call_time &lt; DATE '2019-12-01 00:00:00' ) THEN
        INSERT INTO partition_2019_11 VALUES (NEW.*);
    ELSIF ( NEW.call_time &gt;= DATE '2019-12-01 00:00:00' AND
            NEW.call_time &lt; DATE '2020-01-01 00:00:00' ) THEN
        INSERT INTO partition_2019_12 VALUES (NEW.*);
    ELSE
        RAISE EXCEPTION 'Date out of range.  Fix the partition_test_table_master() function!';
    END IF;
    RETURN NULL;
END;
$$
LANGUAGE plpgsql;

-- 创建触发器，每次执行插入操作时调用上面的函数
CREATE TRIGGER trigger_partition_insert
    BEFORE INSERT ON partition_test_table_master_2
    FOR EACH ROW EXECUTE PROCEDURE func_partition_insert();
```

```sql
-- 插入数据，执行insert操作时，会触发trigger_partition_insert触发器
-- 触发器中会调用函数func_partition_insert()
INSERT INTO public.partition_test_table_master_2(
    call_time, session_id, user_input, intention)
    VALUES ('2019-11-23 00:00:00', 'wyzaneitnfyhwyq', '你好', '打招呼');
```

```sql
-- 也可以把上面的function和trigger换成rule
CREATE RULE rule_partition_insert_2019_11 AS
ON INSERT TO partition_test_table_master_2 WHERE
    ( call_time &gt;= DATE '2019-11-01 00:00:00' 
     AND call_time &lt; DATE '2019-12-01 00:00:00' )
DO INSTEAD
    INSERT INTO partition_2019_11 VALUES (NEW.*);

CREATE RULE rule_partition_insert_2019_12 AS
ON INSERT TO partition_test_table_master_2 WHERE
    ( call_time &gt;= DATE '2019-12-01 00:00:00' 
     AND call_time &lt; DATE '2020-01-01 00:00:00' )
DO INSTEAD
    INSERT INTO partition_2019_12 VALUES (NEW.*);
```



通过上面的介绍，可以看出自定义分区相比于内建分区还是复杂一些，需要自定义触发器和插入数据时的逻辑。对于一般的日志表、交易流水表等，可以使用内建方式以时间分区，对于更复杂的场景，还是需要使用自定义方式创建分区。对于不熟悉pg的存储过程或者函数语法的同学，可能编写函数或者触发器有些困难。pg的存储过程或者函数的语法与oracle和mysql的语法有很多相似之处。

