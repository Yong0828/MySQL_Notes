# MySQL_Notes

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



## 初阶

1、创建或删除 （只需了解即可）

创建表或者视图  

表和视图对于查询者来说区别不大，主要是本身的差别。储存数据是通过表来实现的，有物理存储空间。视图是物理不存在的，相当于子查询。

```mysql
create view IF NOT EXISTS dw.fy_01 as
(select * from dw.dw_debit_info where dt='20200614' limit 100)

create table IF NOT EXISTS dw.fy_0101 as
(select * from dw.dw_debit_info where dt='20200614' limit 100)
```

删除表或者视图

```mysql
drop view dw.fy_01

drop table dw.fy_0101
```



2、查询 where 语句

```mysql
select  *  from  debit_order where final_input_time='2020-06-19'
```



3、聚合语句

sum( )、count（）、count(distinct )、avg() 

```mysql
select 

sum(apply_amount) 申请金额

,count( id)  申通人数

,count(distinct shop_id)  活跃商户

from debit_order

where final_input_state=1

and hidden=0
```



4、limit语句(想看表和字段结构的时候用的比较多)

```mysql
select * from debit_order limit 100
```



5、like 或者 =语句（条件查询，主要用在where后面）

精准查询： =(等于)  <>(不等于)  !=(不等于)

```mysql
select * from debit_order where date(final_input_time)='2020-06-17' ;

select * from debit_order where date(final_input_time) != '2020-06-17' ;

select * from debit_order where date(final_input_time) <> '2020-06-17' ;
```

模糊匹配："like"会与''%''和 "__"结合使用 其中%居多。''%''可以表示0个或者多个字符，匹配任意字符长度。_"_" 只会表示单个字符，用的比较少，了解即可。

```mysql
select * from debit_order where shop_name like  '%%威利斯%%' ;
```



6、union语句 纵列关联，并且列名称要都相同

​     union主要用于两张表的上下关联，如果遇有一样的同一行数据，则自动去重

​     union all主要用于两张表的上下关联，如果遇有一样的同一行数据，不会去重

  union

```mysql
select 'M1' as overdue_type union
select 'M2' as overdue_type union
select 'M3' as overdue_type union
select 'M3+' as overdue_type 
```

union all

```mysql
select 'M1' as overdue_type union all
select 'M2' as overdue_type union all
select 'M3' as overdue_type union all
select 'M3' as overdue_type union all
select 'M3+' as overdue_type 
```



7、order by 语句  排序 默认升序 (asc)    降序desc 

```mysql
select 

date(final_input_time) dateon

,count( id)  申通人数

from debit_order

where final_input_state=1

and hidden=0

and date(final_input_time) >='2020-06-01'

group by dateon 

order by dateon 
```



8、group by 语句 结合聚合语句使用 相当于excel的数据透视功能，对对透视列进行group by

9、表连接

left join        左关联： 获取左表所有记录，即使右表没有对应匹配的记录。用的比较多 .

 ![img](https://www.runoob.com/wp-content/uploads/2014/03/img_leftjoin.gif) 

right join     右关联： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录 

 ![img](https://www.runoob.com/wp-content/uploads/2014/03/img_rightjoin.gif) 

inner join    内关联： 取交集，获取两个表中字段匹配关系的记录 。主要用于共债查询。

 ![img](https://www.runoob.com/wp-content/uploads/2014/03/img_innerjoin.gif) 

full join  全关联：取并集。主要用于Lagged



```mysql
select 
odr.id
,odr.user_name
,odr.shop_id
,channel .channel_code
,channel .channel_desc
from duckchatdb.debit_order odr
left join duckchatdb.shop_debit_channel channel on  odr.channel_id=channel.id
where 
odr.shop_id in (19211762,19213905)
and channel .channel_code is not null
```



10、NULL值的处理

在列中的处理

```mysql
ifnull(column_name,0)

ifnull(column_name,999)
```

在筛选条件里的处理

```mysql
where  column_name is  (not) null
```



参考文档：

 https://www.runoob.com/mysql/mysql-null.html 



## 中阶

一些比较生僻少用的语法

1、csum实现累加功能：将每日的cnt累加求和

```mysql
SET @csum := 0;
SELECT dateon, cnt, (@csum := @csum + cnt) AS 累计单量
FROM 
(
select 
date(final_input_time) dateon
,count(distinct id) cnt
from debit_order
where approve_state=1
and hidden=0
and date(final_input_time) between '2019-04-01' and '2019-05-01'
group by 1
) a
```



2、用if实现row_number() over(partion by columns order by cloumns asc) 即每行面前按自己想要的逻辑添加序号 ：计算出每日单量第一的大区

```mysql
SELECT * FROM (
SELECT a.*
, IF(@city=a.dateon, @rank:=@rank+1, @rank:=1) rank, @city:=a.dateon
 FROM 
	 (
select 
	shop.area_type
	,date(final_input_time) dateon
	,count(*) cnt
	from  debit_order debit
	left join shop on shop.id=debit.shop_id
	where 
	debit.final_input_state=1
	and date(final_input_time) >='2020-06-01'
	and debit.hidden=0
group by
	area_type
	,dateon
	order by dateon,cnt desc
) a, (SELECT @city:=0,@rank:=0) b

) c 
WHERE rank =1;	
	
```


3、去最值

```mysql
select
a.order_id,
a.mobile,
a.final_input_time apply_time,
substring_index(group_concat(b.worke_name order by b.create_time desc),',',1) marketer,
group_concat(b.worke_name order by b.create_time desc) total_marketers,
group_concat(distinct b.worke_name order by b.create_time desc) total_distinct_marketers, -- == 去重并列
substring_index(group_concat(b.create_time order by b.create_time desc),',',1) marketing_time
from duckchatdb.order_info a
left join duckchatdb.customer_market_record b on a.mobile=b.call_phone and a.final_input_time>b.create_time and a.user_accept=1 and a.order_type=2
where a.user_accept=1 and a.order_type=2 -- and datediff(date(a.user_accept_time),date(now()))=0
group by 1

group_concat：可以将a列的不同b列的值放在同一行里面，用逗号隔开
substring_index：截取
```

4、



## 高阶

自己写function。感兴趣的阔以自己做研究。

















