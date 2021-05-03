# 1 报告系统状态的连续日期
系统 每天 运行一个任务。每个任务都独立于先前的任务。任务的状态可以是失败或是成功。

编写一个 SQL 查询 2019-01-01 到 2019-12-31 期间任务连续同状态 period_state 的起止日期（start_date 和 end_date）。即如果任务失败了，就是失败状态的起止日期，如果任务成功了，就是成功状态的起止日期。

最后结果按照起始日期 start_date 排序

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/report-contiguous-dates
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

Failed table:

| fail_date         |
|-|
| 2018-12-28        |
| 2018-12-29        |
| 2019-01-04        |
| 2019-01-05        |

Succeeded table:

| success_date      |
|-|
| 2018-12-30        |
| 2018-12-31        |
| 2019-01-01        |
| 2019-01-02        |
| 2019-01-03        |
| 2019-01-06        |


Result table:

| period_state | start_date   | end_date     |
|-|-|-|
| succeeded    | 2019-01-01   | 2019-01-03   |
| failed       | 2019-01-04   | 2019-01-05   |
| succeeded    | 2019-01-06   | 2019-01-06   |

思路：

1.先按照状态把两个表拼在一起；

2.在用窗口函数，把每个日期减去一个按顺序的数，来识别连续性：比如01-02和01-03，一个减去1，一个减去2，等于同样的结果，也就说明这两个日期连续；相反，若是01-04和01-06，一个减去3一个减去4，得到不同的结果，也就说明不是连续的日期；

3.再根据state和第2步的分类来分组，求每组的最大最小值，便可以得到。

```{SQL}
select type as period_state, min(date) as start_date, max(date) as end_date
from
(
    select type, date, subdate(date,row_number()over(partition by type order by date)) as diff
    from
    (
        select 'failed' as type, fail_date as date from Failed
        union all
        select 'succeeded' as type, success_date as date from Succeeded
    ) a
)a
where date between '2019-01-01' and '2019-12-31'
group by type,diff
order by start_date;
```

# 2 应该被禁止的Leetflex账户
编写一个SQL查询语句，查找那些应该被禁止的Leetflex帐户编号account_id。 如果某个帐户在某一时刻从两个不同的网络地址登录了，则这个帐户应该被禁止。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/leetflex-banned-accounts
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

LogInfo table:

| account_id | ip_address | login               | logout              |
|-|-|-|-|
| 1          | 1          | 2021-02-01 09:00:00 | 2021-02-01 09:30:00 |
| 1          | 2          | 2021-02-01 08:00:00 | 2021-02-01 11:30:00 |
| 2          | 6          | 2021-02-01 20:30:00 | 2021-02-01 22:00:00 |
| 2          | 7          | 2021-02-02 20:30:00 | 2021-02-02 22:00:00 |
| 3          | 9          | 2021-02-01 16:00:00 | 2021-02-01 16:59:59 |
| 3          | 13         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
| 4          | 10         | 2021-02-01 16:00:00 | 2021-02-01 17:00:00 |
| 4          | 11         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |

Result table:

| account_id |
|-|
| 1          |
| 4          |

```{SQL}
SELECT
    DISTINCT L1.account_id
FROM
    LogInfo L1 JOIN LogInfo L2 ON L1.account_id = L2.account_id
WHERE
    L1.ip_address <> L2.ip_address 
    AND 
    L1.login BETWEEN L2.login AND L2.logout;
```

# 3 正则表达式REGEXP
http://blog.itpub.net/352988/viewspace-702052/

找出符合的手机号码：

```{SQL}
SELECT
    *
FROM
    table
WHERE
    REGEXP_LIKE(phone_num, '^[1]{1}[35678]{1}[[:digit:]]{9}$');
```

^  ---  开始

[1]{1}  ---  数字1出现一次

[35678]{1}  ---  数字3或5或6或7或8出现一次

[[:digit:]]{9}  ---  9位数字

$  ---  结束

# 4 次日留存率计算
```{SQL}
select 
    k1.dt,
    count(distinct k1.tuid) dau,
    count(distinct case when date_diff('day',cast(k1.dt as date),cast(k2.dt as date))=1 then k2.tuid end)/count(distinct k1.tuid) day_rate_2 
from 
(--统计日期当天的用户id
    select 
        distinct dt, tuid
    from 
        dwd_dau_custom_di
    where 
        dt >= '2020-06-01'--想要计算的时间段选择，此处为6月1日至今
) k1
left join
(--之后又来的用户id
	select 
        distinct dt, tuid
	from 
        dwd_dau_custom_di
	where 
        dt >= '2020-06-01' 
)k2 
on k1.dt<=k2.dt and k1.tuid = k2.tuid --匹配条件：前后id相同，dt晚于统计日dt
group by 
    k1.dt
order by 
    k1.dt desc;
```
