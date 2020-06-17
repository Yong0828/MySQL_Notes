# MySQL_Notes

- ## [初级](#初级.md)



## 待更新



## 中阶

### 10、NULL值的处理

- 在列中的处理

```mysql
ifnull(column_name,0)

ifnull(column_name,999)
```

- 在筛选条件里的处理

```mysql
where  column_name is  (not) null
```

参考文档：

 https://www.runoob.com/mysql/mysql-null.html 



<br/>

一些比较生僻少用的语法

### 1、csum实现累加功能：将每日的cnt累加求和

```mysql
SET @csum := 0;
SELECT dateon, cnt, (@csum := @csum + cnt) AS 累计单量
FROM 
(
select 
date(final_input_time) dateon
,count(distinct id) cnt
from tablename
where approve_state=1
and hidden=0
and date(final_input_time) between '2019-04-01' and '2019-05-01'
group by 1
) a
```

### 2、用if实现row_number() over(partion by columns order by cloumns asc) 即每行面前按自己想要的逻辑添加序号 ：计算出每日单量第一的大区

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
	from  tablename debit
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

### 3、可以将a列的不同b列的值放在同一行里面，用逗号隔开，且取最值    某个订单有很多人做过回访，查询哪些人做过回访，并且第一次回访的时间是什么时候，第一次回访的人是谁

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



### 4、拆分字段：把同一行里面有逗号隔开的内容，拆分成多行 即group_concat的逆写法

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

### 5、随机取数1000条

```mysql

```



### 6、从第2条记录开始读1条，取第2高的记录

```myslq
select * from tablename limit 1,1;
```



### 创建或删除（只需了解即可）

- 创建表或者视图  

表和视图对于查询者来说区别不大，主要是本身的差别。储存数据是通过表来实现的，有物理存储空间。视图是物理不存在的，相当于子查询。

```mysql
create view IF NOT EXISTS view_01 as
(select * from tablename limit 100);
```

- 删除表或者视图

```mysql
drop view view_01;
```

<br/>



## 高阶

未完待续……



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











































