# 1or与union
or自带去重，而union等价于or，但union all 可以不去重
```
select 
    device_id , 
    gender , 
    age , 
    gpa 
from user_profile 
where university = '山东大学' 
UNION ALL
select 
    device_id , 
    gender , 
    age , 
    gpa 
from user_profile 
where gender = 'male'
```

输出
```
5432|male|25|3.8
2131|male|28|3.3
2138|male|21|3.4
3214|male|None|4.0
5432|male|25|3.8
2131|male|28|3.3
4321|male|28|3.6
```
# 2 case-when 语句
```
select
    device_id,
    gender,
    case
        when age<20 then '20岁以下'
        when age<25 then '20-24岁'
        when age>=25 then '25岁及以上'
        else '其他'
    end age_cut
from user_profile;
```

# 3 计算平均次日留存率问题
首先明确核心公式：次日平均留存率=两天都在人数/后一天人数
做法如下：
1. 用datediff区分第一天和第二天在线的device_id \
datediff(data1,data2) = 1 表示条件为：data1-data2=1
2. 用left outer join做自表联结，即在原表左侧新加一表
3. 用distinct q2.device_id,q2.date做双重去重，找到符合条件的当天在线人数
```
SELECT
    COUNT(DISTINCT q2.device_id, q2.date) / COUNT(DISTINCT q1.device_id, q1.date) AS avg_ret
FROM
    question_practice_detail AS q1
LEFT OUTER JOIN
    question_practice_detail AS q2 ON q1.device_id = q2.device_id
    AND DATEDIFF(q2.date, q1.date) = 1;
```

# 4统计性别的人数，
profile基本信息为：180cm,75kg,27,male\
substring_index(profile,',',-1)\
-1是指从后往前\
','是分隔符
```
select 
    substring_index(profile,',',-1) as gender,
    count(device_id)
from user_submit
group by gender;
```









