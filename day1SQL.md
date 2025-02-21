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
# 2
