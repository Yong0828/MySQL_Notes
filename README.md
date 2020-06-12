# MySQL_Notes

目录

## 常用术语

数据库： 数据库是一些关联表的集合 。比如duckchatdb（业务库）、hawaiidb（逾期库）

数据表： 数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。 

​                debit_order(订单表)、debit_overdue（逾期表）、debit_loan（在贷表） 

主键：主键是唯一的。一个表只有一个主键，可以用主键查询数据。 

外键：用于关联两个表。比如debit_overdue关联debit_order  外键就是debit_id



常用表

业务表  ：duckchatdb.debit_order

账单表 ：duckchatdbdebit_detail

​                duckchatdb. pay_repayment_detail

逾期在贷表：hawaiidb.debit_loan

​                       hawaiidb.debit_overdue



语法:

1、查询 where 语句

select  *  from  debit_order where final_input_time='2020-06-19'































