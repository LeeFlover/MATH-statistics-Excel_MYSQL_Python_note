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
