#### 统计n年之前的数据
|year|salary|
|--|--|
|2000|1000|
|2001|2000|
|2002|3000|
|2003|4000|

|year|salary|
|--|--|
|2000|1000|
|2001|3000|
|2002|6000|
|2003|10000|
```sql
SELECT 
    year, 
    (SELECT SUM(t2.salary) from t_year t2 where t1.`year` >= t2.`year`) salary 
FROM `t_year` t1;
```

### 库存
```sql
DROP TABLE IF EXISTS t_product;
CREATE TABLE `t_product` (
  `p_no` int(11) NOT NULL COMMENT '编号',
  `p_io` smallint(1) NOT NULL COMMENT '1：入仓；2：出仓',
  `amount` int(11) NOT NULL COMMENT '数量'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
```sql
select p_no,IFNULL(sum(case when p_io=1 then amount end)-sum(case when p_io=2 then amount end),sum(case when p_io=1 then amount end)) from t_p group by p_no
```
```sql
SELECT a.p_no, IFNULL(a.c - b.c,a.c) from 
(select p_no, SUM(amount) c from t_p p where p.p_io = 1 GROUP BY p.p_no) a
LEFT JOIN
(select p_no, SUM(amount) c from t_p p where p.p_io = 2 GROUP BY p.p_no) b
on a.p_no = b.p_no
```
```sql
SELECT
	SUM(CASE p_io	WHEN 1 THEN	quantity ELSE 0 END) - SUM(CASE p_io WHEN 2 THEN quantity ELSE 0 END)
FROM
	t_product
GROUP BY
	p_no;
```

### 删除重复数据





