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



2、where 语句  where一般存在代码末端，用来加限制条件的时候使用

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



5、like 、 =语句、in语句（条件查询，主要用在where后面）

精准查询： =(等于)  <>(不等于)  !=(不等于) 

```mysql
select * from debit_order where date(final_input_time)='2020-06-17' ;
select * from debit_order where date(final_input_time) != '2020-06-17' ;
select * from debit_order where date(final_input_time) <> '2020-06-17' ;

select * from debit_order where date(final_input_time) in ('2020-06-17') ;
select * from debit_order where date(final_input_time)  not in ('2020-06-17') ;
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

9、连接

9.1 表连接

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

9.2 等值联接  等值连接一般把条件放在where里面 使用等号“=”连接相关的表

```MySQL
select 
debit.id  debit_id
,debit.shop_id
,shop.id shop_id_new
,shop.name
from
debit_order  debit
,shop shop
where date(final_input_time)=date(now())
and shop.id=debit.shop_id
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

11、子查询

子查询是一个嵌套查询  一般在一张基础表上面在做聚合。



```mysql
select
shop_id
,count(*) cnt
from
(
select 
debit.id  debit_id
,debit.shop_id
from
debit_order  debit
where date(final_input_time)=date(now())
) a
group by 1
```

12、case  when...then... end  或者if() 等条件语句

 CASE WHEN  [expr] T HEN [result1]… ELSE [default] END 

 IF（[expr],[result1],[result2] ）

```mysql
select 
date(odr.final_input_time) dateon
,count(case when approve_state=1 then id end) 通过单
,sum(if( approve_state=1,1,0 )) 通过单
from duckchatdb.debit_order odr
where 
date(odr.final_input_time) >='2020-06-01'
group by 1

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


3、可以将a列的不同b列的值放在同一行里面，用逗号隔开，且取最值    某个订单有很多人做过回访，查询哪些人做过回访，并且第一次回访的时间是什么时候，第一次回访的人是谁

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



4、拆分字段：把同一行里面有逗号隔开的内容，拆分成多行 即group_concat的逆写法

```mysql
法一：
select 
distinct substring_index(substring_index(shop_id,',', b.help_topic_id+1),',',-1) shop_id
from 
(
    select distinct replace(refuse_name,'商户号=','') shop_id
from credit_auto_check_errinfo
where refuse_name like '商户号=%'
) a
join mysql.help_topic b on b.help_topic_id < (length(a.shop_id) - length(REPLACE(a.shop_id, ',', '')) + 1)

(length(a.shop_id) - length(REPLACE(a.shop_id, ',', '')) + 1):计算字符串中被逗号隔开的字符数
```

```sql
法二：
select 
distinct substring_index(substring_index(shop_id,',', b.int_value+1),',',-1) shop_id
from (select distinct replace(refuse_name,'商户号=','') shop_id
   from credit_auto_check_errinfo
   where refuse_name like '商户号=%') a
join (
SELECT x1.N + x10.N*10 + x100.N*100 + x1000.N*1000 as int_value
FROM (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) x1,
       (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) x10,
       (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) x100,
       (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9
       ) x1000
 WHERE x1.N + x10.N*10 + x100.N*100 + x1000.N*1000 <= 10000)
 b on b.int_value < (length(a.shop_id) - length(REPLACE(a.shop_id, ',', '')) + 1);
 
```

参考文档： https://blog.csdn.net/qq_31780525/article/details/54416320 



```mysql
5、随机取数1000条 order by rand() limit 1000
 
6、从第2条记录开始读1条，取第2高的记录    select * from tablename limit 1,1 
```





## 高阶

自己写function。感兴趣的阔以自己做研究。



## 附录

常用的日期函数

```mysql
今天：select now ()
今天：select current_date()
昨天：select date_sub(current_date(),interval 1 day)
本周周一：select date_sub(curdate(),interval weekday(curdate()) + 0 day)
上周周一：select date_sub(curdate(),interval weekday(curdate()) + 7 day)
上周周日：select date_sub(curdate(),interval weekday(curdate()) + 1 day)
本月1号：select date_add(curdate()-day(curdate())+1,interval 0 month)
本月1号：select date_add(curdate(),interval -day(curdate())+1 day)
本月的最后一天：select last_day(curdate())
上个月的最后一天：select last_day(DATE_SUB(CURRENT_DATE(), INTERVAL 1 month))
上个月的今天：select date_add(now(), interval -1 month) 

今天是2013年5月20日。
date_sub('2012-05-25',interval 1 day) 表示 2012-05-24
date_sub('2012-05-25',interval 0 day) 表示 2012-05-25
date_sub('2012-05-25',interval -1 day) 表示 2012-05-26
date_sub('2012-05-31',interval -1 day) 表示 2012-06-01
date_sub(curdate(),interval 1 day) 表示 2013-05-19
date_sub(curdate(),interval -1 day) 表示 2013-05-21
date_sub(curdate(),interval 1 month) 表示 2013-04-20
date_sub(curdate(),interval -1 month) 表示 2013-06-20
date_sub(curdate(),interval 1 year) 表示 2012-05-20
date_sub(curdate(),interval -1 year) 表示 2014-05-20


date_add(date,interval num unit)
#向后偏移时间
select
 "2019-01-01" as col1
 ,date_add("2019-01-01",interval 7 year) as col2
 ,date_add("2019-01-01",interval 7 month) as col3
 ,date_add("2019-01-01",interval 7 day) as col4
select
 "2019-01-01 01:01:01" as col1
 ,date_add("2019-01-01 01:01:01",interval 7 hour) as col2
 ,date_add("2019-01-01 01:01:01",interval 7 minute) as col3
 ,date_add("2019-01-01 01:01:01",interval 7 second) as col4
#向前偏移时间
select
 "2019-01-01" as col1
 ,date_sub("2019-01-01",interval 7 year) as col2
 ,date_sub("2019-01-01",interval 7 month) as col3
 ,date_sub("2019-01-01",interval 7 day) as col4

select
"2019-01-01" as col1
 ,date_add("2019-01-01",interval -7 year) as col2
 ,date_add("2019-01-01",interval -7 month) as col3
 ,date_add("2019-01-01",interval -7 day) as col4

#两日期做差
datediff(end_date,start_date)
select datediff("2019-01-07","2019-01-01")


#两时间做差
timestampdiff(second, create_time, update_time)

```











































