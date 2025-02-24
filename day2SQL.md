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
其中
PARTITION BY department：将数据按部门分区。
ORDER BY salary DESC：在每个部门内，按薪水降序排列。
ROW_NUMBER() 将为每个部门的员工分配一个排名，薪水最高的员工排名为 1，依此类推。
