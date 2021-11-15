---
title: mysql的应用（一）
date: 2021-11-15 17:00:33
tags: [SQL,数据库]
---

来自于实习时11.15号的一个数据修复问题。

<!--more-->

 `crm_project_report` 表修复 `follow_time` 数据

1. `follow_time` 时间初始化为同表的`create_at`时间
2. `follow_time` 时间更新为 `crm_followup` 表对应 `report_id` 的 `msg_type` 为 "normal" 的最新 `updated_at` 时间
   查询语句为：

```sql
select  crm_project_report.report_id, follow_time, cf.updated_at, created_at
from crm_project_report
         left join (select  report_id, max(updated_at) as updated_at
                    from crm_followup
                    where msg_type = 'normal'
                    group by report_id
                     ) cf
                   on crm_project_report.report_id = cf.report_id
order by report_id desc;
```

`crm_client_company` 表修复  `follow_time` 数据

1. `follow_time` 时间初始化为同表的`create_at`时间
2. `follow_time` 时间更新为 `crm_followup` 表对应 `client_id` 的 `msg_type` 为 "normal" 的最新 `updated_at` 时间。
   查询语句为：

```sql
select crm_client_company.client_id, follow_time, cf.updated_at
from crm_client_company
         left join (select client_id, max(updated_at) as updated_at
                    from crm_followup
                    where msg_type = 'normal'
                    group by client_id
) cf
                   on crm_client_company.client_id = cf.client_id
order by client_id desc;
```

以上是一种根据id进行分组后，分组内按照时间进行排序，然后取最新一条记录的方式，刚开始的时候我的想法是先`group by` 然后`order by`，最后进行`LIMIT`的思路进行，但是前两条语句就不能一起使用，在只取一条数据的情况下，可以采用`group by`之后直接取`max`的方式，但是如果取最新的两条数据，这种方法就不适用了，可以采取下面这种sql进行。

```sql
select crm_project_report.report_id, follow_time, cf.updated_at, crm_project_report.created_at
from crm_project_report
         left join (select *
                    from crm_followup a
                    where msg_type = 'normal'
                      and (
                              select count(*)
                              from crm_followup b
                              where a.updated_at <= b.updated_at
                                and a.report_id = b.report_id
                          ) <= 2
) cf
                   on crm_project_report.report_id = cf.report_id
order by created_at desc
```

如果我们取前两条记录，那么就相当于取最大（最小）的两条记录，就可以通过判断比当前记录还要大的记录的条数来进行where条件判断，具体而言，如果在当前记录内，在相同report_id的条件下，如果比当前记录时间还要大的记录的数目不超过2，那么当前记录就是前2的记录。

扩展：这种问题也可以类似于，在一个成绩表中有多种科目的成绩，表内具有科目id、学生id、学生成绩，查询某个科目的前n个学生记录。
