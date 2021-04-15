# 1
请编写一个 SQL 查询，描述每一个玩家首次登陆的设备名称。

Activity table:


| player_id | device_id | event_date | games_played |
|-|-|-|-|
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |


Result table:


| player_id | device_id |
|-|-|
| 1         | 2         |
| 2         | 3         |
| 3         | 1         |


## Method 1 ---->  窗口函数
```{SQL}
SELECT player_id, device_id
FROM (SELECT player_id, device_id,
        RANK() OVER (PARTITION BY player_id ORDER BY event_date) as ranking
        FROM Activity) Table2
WHERE ranking = 1;
```

## Method 2 ---->  联合查询
```{SQL}
SELECT player_id, device_id
FROM Activity
WHERE (player_id, event_date) IN 
(SELECT player_id, MIN(event_date)
FROM Activity
GROUP BY player_id);
```
  
## Method 3 ---->  all
```{SQL}
SELECT player_id, device_id
FROM Activity a1
WWHERE a1.event_date<=all(SELECT a2.event_date FROM Activity a2 WHERE a1.player_id=a2.player_id);
```


# 2
编写一个 SQL 查询，同时报告每组玩家和日期，以及玩家到目前为止玩了多少游戏。也就是说，在此日期之前玩家所玩的游戏总数。

Result table:


| player_id | event_date | games_played_so_far |
|-|-|-|
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |


## Method 1 ----> 窗口函数
```{SQL}
SELECT player_id, event_date, 
SUM(games_played) OVER (PARTITION BY player_id ORDER BY event_date) AS games_played_so_far
FROM Activity;
```


## Method 2 ----> 自联结
```{SQL}
SELECT t1.player_id,
       t1.event_date,
       SUM(t2.games_played) games_played_so_far
FROM Activity t1,Activity t2
WHERE t1.player_id=t2.player_id
  AND t1.event_date>=t2.event_date
GROUP BY t1.player_id,t1.event_date;
```


# 3
编写一个 SQL 查询，报告在首次登录的第二天再次登录的玩家的比率，四舍五入到小数点后两位。换句话说，您需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数。

Result table:

| fraction  |
|-|
| 0.33      |


## Method 1 ----> INNER JOIN拼接表格
```{SQL}
SELECT
	ROUND(COUNT(DISTINCT a.player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) AS fraction
FROM
	Activity AS a
	INNER JOIN (
		SELECT
			player_id, MIN(event_date) AS first_login
		FROM
			Activity
		GROUP BY 
			player_id
	) AS b
	ON a.player_id = b.player_id AND DATEDIFF(a.event_date, b.first_login) = 1;
```

## Method 2 ----> WITH AS创建临时表
```{SQL}
WITH t1 AS
(
    SELECT a.*,
            LEAD(a.event_date,1) OVER W AS edate,
            MIN(event_date) OVER W AS mdate
        FROM Activity a
        WINDOW W AS (PARTITION BY player_id ORDER BY a.event_date)  --把窗口函数单独写出来避免代码冗余
)
SELECT ROUND
(
    (SELECT COUNT(DISTINCT player_id) FROM t1 WHERE DATEDIFF(t1.edate,t1.mdate) = 1)/(SELECT COUNT(DISTINCT player_id) FROM     t1),2
) AS fraction;
```

### TIPS ----> LEAD() 和 LAG()
```{SQL}
SELECT *,
       LEAD(Value, 1, 666) OVER (ORDER BY Value) AS LEADVALUE,  --提前1行，默认值666
       LAG(Value, 2, 888) OVER (ORDER BY Value) AS LAGVALUE  --滞后2行，默认值888
FROM Table1;
```
![image](https://user-images.githubusercontent.com/69565742/112946859-98bbc180-9168-11eb-92fd-59ca1f193c32.png)


# 4
请编写SQL查询来查找每个公司的薪水中位数。挑战点：你是否可以在不使用任何内置的SQL函数的情况下解决此问题。

## Method 1 ----> 窗口函数1
```{SQL}
SELECT
    Id, Company, Salary
FROM
    (
    SELECT Id, Company, Salary, 
        ROW_NUMBER() OVER(PARTITION BY Company ORDER BY Salary) AS ranking,
        COUNT(Id) OVER(PARTITION BY Company) AS cnt
    FROM
        Employee
    ) a
WHERE
    ranking>=cnt/2 AND ranking<=cnt/2+1;
```

方法的巧妙之处在于 WHERE 语句的 >= 和 <=，直接把两种情况包含了进来。

## Method 2 ----> 窗口函数2
```{SQL}
SELECT 
    Id, Company, Salary
FROM 
    (
    SELECT *, 
        RANK() OVER(PARTITION BY Company ORDER BY Salary, Id) AS R1, 
        RANK() OVER(PARTITION BY Company ORDER BY Salary DESC, Id DESC) AS R2
    FROM 
        Employee) AS A
WHERE 
    R1 BETWEEN R2-1 AND R2+1;
```

中位数的计算都可以按相同套路：按正序、逆序排列，然后取值。

# 5
输入：

| id   | action    | question_id  | answer_id  | q_num     | timestamp  |
|-|-|-|-|-|-|
| 5    | show      | 285          | null       | 1         | 123        |
| 5    | answer    | 285          | 124124     | 1         | 124        |
| 5    | show      | 369          | null       | 2         | 125        |
| 5    | skip      | 369          | null       | 2         | 126        |


输出：

| survey_log  |
|-|
|    285      |


```{SQL}
SELECT 
    question_id AS survey_log
FROM	
    survey_log
GROUP BY
    question_id
ORDER BY SUM(IF(action = 'answer', 1, 0)) / SUM(IF(action = 'show', 1, 0)) DESC
LIMIT 1;
```
IF语句比CASE更简洁；

可以在ORDER BY里直接写语句。









