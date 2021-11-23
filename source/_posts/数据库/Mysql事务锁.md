---
title: Mysql事务锁
date: 2021-11-23 16:42:32
tags: [SQL,数据库]
---

　　今天在修复审核状态的bug中遇到一个奇怪的问题，只是简单的添加一个`Omit`函数，如下所示，但是添加之后还是审核失败，报错是数据库连接超时。

<!--more-->

```go
if err := tx.Model(report).Where("report_id = ?", reportId).Omit("follow_time").Updates(&report).Error; err != nil {
	tx.Rollback()
	return errors.New("更新异常，项目审核失败")
}
```

```go
[mysql] read tcp 192.168.31.109:1160->192.168.1.100:13307: i/o timeout
invalid connection
```

　　这个错误十分古怪，如果更换其他的reportId仍然可以审核成功，但是某个固定id一直报这个错误。

　　经过反复检测，这个错是因为事务锁引起的，在添加`Omit`函数之前，之前也没有添加`tx.Rollback()`这部分，事务没有回滚或者提交，一直保持着对应记录的行锁，所以一直失败，报超时的错误。

```go
if err := tx.Model(report).Where("report_id = ?", reportId).Updates(&report).Error; err != nil {
	//tx.Rollback()
	return errors.New("更新异常，项目审核失败")
}
```


　　尝试复现这个bug，首先把`tx.Rollback()`这部分注释掉，然后把`Omit`函数去掉，这样会导致代码在更新这部分出错，如上述代码，然后在DataGrip中修改对应行的记录，发现无法提交更改，说明事务一直对记录保持行锁。

```sql
UPDATE smartservice.crm_project_report t SET t.review_status = 0 WHERE t.report_id = 10171
```

　　这种情况可以通过

```sql
SELECT * FROM information_schema.INNODB_TRX;
```

　　查看到当前的行级锁，发现确实有个行级锁。

　　可以通过`kill trx_mysql_thread_id`的方式杀死对应的线程，此时去执行刚刚失败的update命令就成功了。

　　这个bug真的找了很久，也不是我最终找到的，以后用事务的时候一定记得及时的进行`rollback`。
