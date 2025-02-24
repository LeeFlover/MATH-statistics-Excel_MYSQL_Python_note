# 1正确率问题
使用avg语句是快速的解决方法
```
SELECT 
    difficult_level,
    AVG(IF(result='right',1,0)) AS correct_rate
FROM 
    user_profile u, 
    question_practice_detail qpd, 
    question_detail qd
WHERE 
    u.device_id = qpd.device_id 
    AND qpd.question_id = qd.question_id
    AND university='浙江大学'
GROUP BY difficult_level
ORDER BY correct_rate;
```

#2 周杰伦音乐排序
```
with
    a1 as (
        select
            fdate,
            song_name,
            a.song_id
        from
            play_log a
            join song_info b on a.song_id = b.song_id
            join user_info c on a.user_id = c.user_id
            and c.age >= 18
            and c.age <= 25
            and b.singer_name = '周杰伦'
        where
            year (fdate) = 2022
    ),
    a2 as (
        select
            month (fdate) as month,
            song_name,
            song_id,
            count(1) as play_pv
        from
            a1
        group by
            1,
            2,
            3
    ),
    a3 as (
        select
            month,
            row_number() over (
                partition by
                    month
                order by
                    play_pv desc,
                    song_id asc
            ) as ranking,
            song_name,
            play_pv
        from
            a2
    )
select
    *
from
    a3
where
    ranking <= 3
ordeR by
    month,
    ranking asc
```
1. 其中 with a1 as===CTE 代表 Common Table Expression（公共表表达式）\
用于复杂的表查询，每一次处理一个条件
2. ROW_NUMBER() 是一个窗口表达式，作用是为结果集的每一行分配一个唯一的序列号，使用 ROW_NUMBER() 时，通常需要配合 OVER()\
例如
```
SELECT 
    employee_id,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department ORDER BY salary DESC) AS rank
FROM 
    employees;
```
其中\
PARTITION BY department：将数据按部门分区。\
ORDER BY salary DESC：在每个部门内，按薪水降序排列。\
ROW_NUMBER() 将为每个部门的员工分配一个排名，薪水最高的员工排名为 1，依此类推。

# 3 查询最大连续登录天数问题
关键词： row_number() over() 以及interval语句
```
select
    user_id,
    max(连续登录天数) as max_consec_days
# 只用计算登录天数
from
    (select
        user_id,
        max(日期排序)-min(日期排序)+1 as 连续登录天数
# 核心问题在于此公式，连续情况下，最大日期减最小日期减1
    from
        (select 
            user_id,
            fdate,
            row_number() over(partition by user_id order by fdate) as 日期排序,
# 将1号2号4号转化为123.
            date_sub(fdate, interval row_number()over(partition by user_idorder by fdate) day) as 初始日期
# date_sub计算时间差，如date_sub('2023-01-10', interval 5 day)结果为1月5号interval表示时间间隔
        from tb_dau
        where fdate between '2023-01-01' and '2023-01-31') as t1
    group by user_id, 初始日期) as t2

group by user_id
```

另一种方法由GPT给出
```
WITH ranked_logins AS (
    SELECT
        user_id,
        fdate,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY fdate) AS rn
    FROM
        tb_dau
    WHERE
        fdate BETWEEN '2023-01-01' AND '2023-01-31'
),
consecutive_days AS (
    SELECT
        user_id,
        COUNT(*) AS num_consecutive_days
    FROM
        ranked_logins
    GROUP BY
        user_id,
        DATE_SUB(fdate, INTERVAL rn DAY)
)
SELECT
    user_id,
    MAX(num_consecutive_days) AS max_consec_days
FROM
    consecutive_days
GROUP BY
    user_id;
```

第二个gpt思路
```
WITH ranked_logins AS (
    -- 获取2023年1月1日至2023年1月31日的用户登录记录，并为每个登录日期分配一个行号
    SELECT
        user_id,
        fdate,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY fdate) AS rn
    FROM
        tb_dau
    WHERE
        fdate BETWEEN '2023-01-01' AND '2023-01-31'
),
-- 计算连续天数的标识
date_groups AS (
    SELECT
        user_id,
        fdate,
        DATE_SUB(fdate, INTERVAL rn DAY) AS date_group
    FROM
        ranked_logins
)
-- 计算每个用户的最大连续登录天数
SELECT
    user_id,
    COUNT(*) AS max_consec_days
FROM
    date_groups
GROUP BY
    user_id,
    date_group
ORDER BY
    user_id;
```

# 4查询倒数第三排
核心要点：limit函数,limit 2,1表示2+1排，第一个
```
select *
from employees
where hire_date = (
    select distinct hire_date from employees order by hire_date desc limit 2,1
)
```
