---
layout: post
title: 关于两个业务的表设计和查询
date: 2014-12-30 12:33:16
categories: development
tags: sql
author: Aisin
---

最近，在开发 OA 时遇到两个业务功能，使用的 SQL 比较有代表性，所以记录一下，方便日后查阅。

###1.系统配置项的表设计、查询

一般在开发一个模块时，通常都会有模块专属的表去存储该模块的数据。但是也有另一种情况，比如一些系统的维护数据、或者模块不同但拥有相同属性的这种。它们通常都是以 `属性名(key)` 和 `属性值(value)` 成对存储的。但是设想一下，有 50 个模块都有一个 `exampleKey : exampleValue` 的属性，因为属于配置数据，不是业务数据而单独的表存储，那我不可能为 50 个模块分别建 50 张表，这显然不合理。所以最终设计如下：

| ID        | META_KEY           |  META_KVALUE  | META_DESC |
| --------  | -----:             | :----:  | :----:|
| 1         | SYS_UPDATE_DEADLINE|   6     | 系统时间间隔 |
| 2         | SYS_WARNING_DEADLINE|   36   | 系统预警时间 |
| ...         | ...|   ...  | ... |

这样，表示固定的，并且多个配置项可存在一张表中，每个属性作为一条单独的记录存储。那么问题来了：这种存储方式我怎样取值和更新呢？

因为我如果想查询表里面多个字段时，需要这样的：

| SYS_UPDATE_DEADLINE|SYS_WARNING_DEADLINE  | ... |... |
| -----:            | :----:  				| :----:|:----:|
| 6					|   36     				| ... |... |

行转列的查询：

```sql
select 
 sum(decode(meta_key,'SYS_UPDATE_DEADLINE',META_VALUE,null)) as SYS_UPDATE_DEADLINE,
 sum(decode(meta_key,'SYS_WARNING_DEADLINE',META_VALUE,null)) as SYS_WARNING_DEADLINE
from SOA_SYS_OPTIONS
```

更新：

```sql
update SOA_SYS_OPTIONS set META_VALUE = 
case 
	when META_KEY = 'SYS_UPDATE_DEADLINE' then {?sys_update_deadline?} 
	when META_KEY = 'SYS_WARNING_DEADLINE' then {?sys_warning_deadline?}
end
```

###2.一个多表联表查询

A表（主数据）：

| id      | propose_dept |  PLAN_FINISH_TIME  | ... |
| --------| -----:       | :----:  			  | :----:|
| 1       | 研发部		| 		2014-12-12 	  | ... |
| 2       | 财务部		| 		2014-12-28 	  | ... |
| 3       | 项目部		| 		2014-12-17 	  | ... |
| 4       | 研发部		| 		2014-11-28    | ... |
| 5       | 客服部		|		2014-12-15    | ... |
| 6       | 财务部		| 		2014-11-23 	  | ... |
| ...      | ...		| 		... 	  | ... |

B表：

| ID        | META_KEY           |  META_KVALUE  | META_DESC |
| --------  | -----:             | :----:  | :----:|
| 1         | SYS_UPDATE_DEADLINE|   6     | 系统时间间隔 |
| 2         | SYS_WARNING_DEADLINE|   36   | 系统预警时间 |

业务需求：

简单说，需要三列数据：A表中部门，统计该部门的任务数，统计该部门的预警任务数。符合预警任务条件：当前日期与 `PLAN_FINISH_TIME` 做差小于B表中 `SYS_WARNING_DEADLINE` 的值。

最终SQL

```sql
select t1.propose_dept,t1.task_count,nvl(t2.task_count_d,0) as time_out_count
from 
(SELECT
 propose_dept,
 COUNT (*) AS task_count
FROM
 SOA_WORK_ORDERD
GROUP BY
 propose_dept
) t1,
(
SELECT
 propose_dept,
 COUNT (*) AS task_count_d
FROM
 SOA_WORK_ORDERD
where
 to_date(to_char(SYSDATE,'yyyy-MM-dd'),'yyyy-MM-dd') - to_date(PLAN_FINISH_TIME,'yyyy-MM-dd') < (SELECT meta_value FROM SOA_SYS_OPTIONS WHERE meta_key = 'SYS_WARNING_DEADLINE')
GROUP BY
 propose_dept
)t2
where 
t1.propose_dept = t2.propose_dept(+)
```
