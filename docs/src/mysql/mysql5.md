---
title: MySQL中GROUP BY后不允许SELECT中显示未聚合的列
shortTitle: 10.MySQL中GROUP BY后不允许SELECT中显示未聚合的列
category:
  - MySQL
tag:
  - MySQL
date: 2024-11-07
---

先来看一道SQL题：求每个城市下面年龄最大的员工的信息。

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051616.png)
一看题目，秒了，直接来托大的~

```sql
select u.name , max(u.age),u.city 
from `user` u 
group by u.city 
having max(u.age) 
```
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051878.png)
我一看结果![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051564.png)
这查出来的怎么是Jackson吗？我去，不应该是Lucas吗。我又反复试了几次，发现的确不对啊。怎么回事呢？

那就用子查询再试一下：

```sql
select u.name ,u.age ,u.city 
from `user` u 
where (
	select count(*)
	from `user` u1
	where u.city = u1.city and u.age < u1.age
) < 1;
```
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051106.png)
这样这个name字段，就正确了。咦、、、
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051462.png)
搜一下MySQL中文手册，才知道使用GROUP BY子句的查询，不能引用未在GROUP BY子句中命名的非聚合列。手册链接-> [https://mysql.net.cn/doc/refman/5.6/en/group-by-handling.html](https://mysql.net.cn/doc/refman/5.6/en/group-by-handling.html)

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182051415.png)
哦，原来是这里错误了。关于GROUP BY的使用，还是要注意下这个问题。
