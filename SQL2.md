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
