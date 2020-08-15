# 常用sql


<!--more-->

https://www.jianshu.com/p/5a2dae144238

## mysql 行转列
```sql
select USER_NAME,
       max(case COURSE when '语文' then SCORE else 0 end) "语文",
       max(case COURSE when '数学' then SCORE else 0 end) '数学',
       max(case COURSE when '英语' then SCORE else 0 end) '英语',
       sum(SCORE)                                       '总分',
       avg(SCORE)                                       '平均分'
from student
group by USER_NAME;
```

## mysql 列转行

```sql
select USER_NAME, '语文' course, CN_SCORE score
from grade
union
select USER_NAME, '数学' course, MATH_SCORE score
from grade
union
select USER_NAME, '英语' course, EN_SCORE score
from grade
order by USER_NAME, score
```


