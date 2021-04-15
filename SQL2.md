# 1 平方，次方函数
POWER(num,2)  POWER(num,0.5)

# 2 日期格式的多种转化
DATE_FORMAT(date, '%Y-%m') ----->  可以将'2020-03-06'转化为'2020-03'且为日期格式

# 3 截取字符串的部分
SUBSTRING(str FROM a FOR b) ------>  从str的第a个字符开始，截取b各个字符

# 4 透视表操作
![image](https://user-images.githubusercontent.com/69565742/114264861-ffba5f80-9a1f-11eb-942d-68e5f5bba5e0.png)

![image](https://user-images.githubusercontent.com/69565742/114264873-119c0280-9a20-11eb-806a-18d9e3ff4055.png)

```{SQL}
#step1.构造基准id，使用row_number()窗口函数
SELECT *, ROW_NUMBER() OVER (PARTITION BY continent ORDER BY NAME) rk FROM student
```
![image](https://user-images.githubusercontent.com/69565742/114264915-57f16180-9a20-11eb-9db2-0bc959f0c776.png)

```{SQL}
#step2.：group by+sum/max/min(case when)

SELECT 
    MAX(CASE continent WHEN 'Asia'THEN NAME ELSE NULL END) AS Asia,
    MAX(CASE continent WHEN 'America'THEN NAME ELSE NULL END) AS America,
    MAX(CASE continent WHEN 'Europe'THEN NAME ELSE NULL END) AS Europe 
FROM 
    (SELECT *, ROW_NUMBER() OVER (PARTITION BY continent ORDER BY NAME) rk FROM student) t
GROUP BY rk;
```

# 5 空字符变成null
直接在内层select上套一层select，就可以自动在找不到的时候返回null。
```{SQL}
SELECT (SELECT  
    num
FROM
    my_numbers
GROUP BY
    num
HAVING
    COUNT(num) = 1
ORDER BY
    num DESC
LIMIT 1) AS num;
```

# 6 UNION ALL的妙用
编写一个 SQL 查询，以查找每个月和每个国家/地区的已批准交易的数量及其总金额、退单的数量及其总金额。

Transactions 表：

| id   | country | state    | amount | trans_date |
|-|-|-|-|-|
| 101  | US      | approved | 1000   | 2019-05-18 |
| 102  | US      | declined | 2000   | 2019-05-19 |
| 103  | US      | approved | 3000   | 2019-06-10 |
| 104  | US      | declined | 4000   | 2019-06-13 |
| 105  | US      | approved | 5000   | 2019-06-15 |



Chargebacks 表：

| trans_id   | trans_date |
|-|-|
| 102        | 2019-05-29 |
| 101        | 2019-06-30 |
| 105        | 2019-09-18 |



Result 表：

| month    | country | approved_count | approved_amount | chargeback_count  | chargeback_amount  |
|-|-|-|-|-|-|
| 2019-05  | US      | 1              | 1000            | 1                 | 2000               |
| 2019-06  | US      | 2              | 8000            | 1                 | 1000               |
| 2019-09  | US      | 0              | 0               | 1                 | 5000               |


```{SQL}
SELECT month, country,
COUNT(IF(tag=1, 1, NULL)) AS approved_count,
SUM(IF(tag=1, amount, 0)) AS approved_amount,
COUNT(IF(tag=0, 1, NULL)) AS chargeback_count,
SUM(IF(tag=0, amount, 0)) AS chargeback_amount
FROM (
    SELECT country, amount, 1 AS tag,
    date_format(trans_date, '%Y-%m') AS month
    FROM Transactions
    WHERE state='approved'
    
    UNION ALL

    SELECT country, amount, 0 AS tag,
    date_format(c.trans_date, '%Y-%m') AS month
    FROM Transactions AS t RIGHT OUTER JOIN Chargebacks AS c
    ON t.id = c.trans_id
) AS temp
GROUP BY  month, country;
```
# 7 MySQL自定义变量
设置用户变量的一个途径是执行SET语句：
```{SQL}
SET @var_name = expr [, @var_name = expr] ...
```

对于SET，可以使用=或:=作为分配符。分配给每个变量的expr可以为整数、实数、字符串或者NULL值。

也可以用语句代替SET来为用户变量分配一个值。在这种情况下，分配符必须为:=而不能用=，因为在非SET语句中=被视为一个比较操作符。
```{SQL}
SELECT @t1:=(@t2:=1)+@t3:=4,@t1,@t2,@t3;
```

设置行号（还能通过@rowNum:=@rowNum+a实现累加）
```{SQL}
SELECT 
    (@rowNum:=@rowNum+1) AS rowNum,
    test2.* 
FROM 
    test2,
    (SELECT(@rowNum:=0)) tmp
```
