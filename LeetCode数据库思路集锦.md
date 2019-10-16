# LeetCode数据库思路集锦

## 简单题

1. 175组合两个表

   join 组合两张表 

2. 176第二高薪水

   ~~~mysql
   # 1
   select ifnull((select distinct salary from Employee 
                  order by salary desc limit 1,1),null) as SecondHighestSalary
   ## group by 快于distinct!
   # 2
   select max(salary) as SecondHighestSalary from salary
   where salary < (select max(salary) from salary)
   ~~~

3. 181超过经理收入的员工  **！！！**

   ~~~mysql
   #1
   select a.name as Employee from Employee a join  Employee b
   ON a.ManagerId = b.Id # 筛选出不是经理的人！
   # 普通员工后面的mannagerID相当于是上司！
   #所以筛选出来的是普通员工的人，而非是要筛选出经理！
   and  a.salary > b.salary
   
   #2
   select a.name from employee a,employee b
   where a.mannagerid = b.id and a.salary > b.salary
   ~~~

4. 182查找重复邮箱

   - having 

   - ~~~mysql
     # 曾经使用的方法！
     select Email from
     (select Email, count(Email) as num  from Person
       group by Email) as statistic
     where num > 1
     ~~~

5. 183从不订购的客户

   ~~~mysql
   # 1
   select  name from customers c 
   where c.id not in (select customerid from orders)
   # 在数据量大的表格中，not in的效率较低！
   
   # 2
   select name from customers c left join  orders o
   on c.id = o.customerid
   where  CustomerId is null
   # 多看看全表——
   #select name from customers c left join  orders o
   #on c.id = o.customerid
   ~~~

6. 196**删除重复**的电子邮箱！

   ~~~mysql
   DELETE FROM Person
   WHERE Id NOT IN (  -- 删除不在查询结果中的值
       SELECT id FROM
      (
          SELECT MIN(Id) AS Id -- 排除Email相同时中Id较大的行
          FROM Person
          GROUP BY Email
      ) AS temp    -- 此处需使用临时表，否则会发生报错
   )
   ~~~

7. 197上升的温度

   date**d**iff 用法！

8. 511游戏玩法分析1

   第一次登陆平台的日期

   min(event_date) + group by event_date

   group by 有**去重**的作用！

9. 512 玩法分析2

   每一个玩家首次登陆的设备名称

   ~~~mysql
   #1  效率可能有些低？
   select a.player_id ,a.device_id from Activity a where (a.player_id ,a.event_date) in 
   (select player_id,min(event_date) as first_login from Activity group by player_id)
   
   #2  初始想法——最小的时间对应最初的设备！——答案错误
   select player_id ,device_id from 
   ( select player_id,device_id,min(event_date) from Activity group by player_id ) s
   # 以player_id 分组 无device_id操作导致默认取同一id的第一个数据！
   ~~~

10. **577员工奖金**

    ~~~mysql
    # 区别左连接和连接！！！！
    select e.name,b.bonus from Employee e left join Bonus b
    on e.empId = b.empId
    where bonus < 1000 or bonus is null
    # 如果使用join 只会出现一个结果没有NULL值
    
    # 使用ifnull
    select name,bonus
    from Employee as a left join Bonus as b on a.empId=b.empId
    #where b.bonus<1000 or b.bonus is null
    where ifnull(bonus,0)<1000
    # ifnull也可以作为条件使用，不放在select语句中！
    ~~~

11. 584寻找用户推荐人

    - ```mysql
      referee_id is null
      ```

    - 或者使用子查询 id  不在referee_id= 2 的ID 中

12. 586订单最多的客户

    没感觉——pass

13. 595大的国家

    可以使用union会提升速度？

    ~~~mysql
    select name,population,area from World
    where area > 3000000
    union
    select name,population,area from World
    where population > 25000000;
    ~~~

14. 596超过5名学生的课

    注意note：学生在每个课中不应被重复计算。

    所以使用count时应该添加 distinct

15. 603连续空座位

    on后面跟的条件不止是ID相等，还可以是表达式！

    ~~~mysql
     select distinct  a.seat_id from cinema a left join cinema b
    on abs(a.seat_id - b.seat_id ) =1
    where  a.free = 1
    and b.free = 1
    order by a.seat_id
    
    ~~~

16. 销售员

    没感觉！

17. 610判断三角形

    两边之和大于第三边

    case when  then else end

    if(,,)

18. 613直线上的距离

    一张表用两次

    聚合函数

    min(abs())

19. 1069产品销售分析

    没有感觉q
    
20. 597好友申请——总体通过率

    emmm
    
21. 1050合作过至少三次的演员和导演对！

    count(*)和count（actor_id) 区别？？？

    **tip：MySQL参考书p538:  count(*) 统计总行数，count(score)统计没有缺失的分数**

    统计null个数：**sum(isnull(score))**

22. 1075 项目员工I

    group by 后面的可以使用 12 3 4 来代替前面select部分的参数！

    ~~~mysql
    SELECT project_id, ROUND(AVG(experience_years), 2) AS average_years
    FROM Project t1, Employee t2
    WHERE t1.employee_id = t2.employee_id
    GROUP BY 1
    ~~~

23. 1076 项目员工II

    考虑到多个项目并列

    ~~~mysql
    #1
    select project_id from project
    group by project_id 
    having count(project_id) = (select count(project_id) from project 
                                group by project_id 
                               order by count(project_id) desc limit 1)
                               
    #2
    select project_id
    from Project
    group by  project_id
    having count(employee_id)>=all(select count(employee_id) from Project group by  project_id);
    # any,all 和比较操作符联用
    #any关键词可以理解为“对于子查询返回的列中的任一数值，如果比较结果为true，则返回true”。all的意思是“对于子查询返回的列中的所有值，如果比较结果为true，则返回true”
    ~~~

24. 1082销售分析

    ~~~mysql
    #1
    select seller_id from sales group by seller_id
    having sum(price) = (select distinct sp from (select seller_id,sum(price) sp from sales 
    group by seller_id 
    order by sum(price) desc limit 1) t)
    
    # 2
    select 
    seller_id
    from sales
    group by seller_id
    having
    sum(price)>=all(select sum(price) from sales group by seller_id )
    # 方法1与方法2之间区别在于 having子查询语句的简洁性！
    # 和23题#2一样两个都是@西红柿鸡蛋面的写法
    ~~~

25. 627交换工资

26. 1083销售分析II

    ~~~mysql
    select distinct buyer_id from sales s left join product p
    on s.product_id = p.product_id
    where p.product_name = "S8"
    and s.buyer_id not in (select  s.buyer_id from sales s left join product p on 							s.product_id = p.product_id
                         	 where p.product_name = "iPhone" group by buyer_id )
    #不可以使用where p.product_id = 3代替
    #left join product p on s.product_id = p.product_id where p.product_name = "iPhone"
    ~~~




## 中等

1. 177第N高薪水   2019-09-24

   CREAT FUNCTION ——UDFs (user-defined functions) 创建函数

   ~~~
   # 题目多解
   # 1创建函数
   # 2取最小值（min(select salary  order by desc limit n))
   # 3考虑N不合理的情况
   ~~~
   
2. 178分数排名

   非null情况
   
   ~~~mysql
   #多方法
   # 1 西红柿鸡蛋面？
    select s1.Score,count(distinct s2.score)  Rank  from scores s1 left join scores s2
      on s1.score <= s2.score
      group by s1.Id
      order by Rank 
      
     
   
   # 2根据书p556
   
     	 set @a =0
   ~~~
   
3. 262非禁止用户取消率（DD）

   ~~~mysql
   SELECT 	a.Request_at AS Day,
   	ROUND( sum( CASE WHEN a.STATUS != "completed" THEN 1 ELSE 0 END ) / count( a.id ), 2 ) AS "Cancellation Rate"
   FROM Trips a LEFT JOIN Users b ON a.client_id = b.users_id 
   WHERE b.Banned = "No" 
   	AND a.Request_at BETWEEN "2013-10-01" 
   	AND "2013-10-03" 
   GROUP BY a.Request_at
       
   # 两个条件 ： t.c_id = u.u_id ;user_banned 剔除！
   ~~~

   





## Hard

1. 185部门工资前三高的所有员工

   ~~~mysql
   select d.name as Department,e.name as Employee,e.salary as Salary from employee as e inner join department as d 
   on e.DepartmentId=d.id 
   where (select count( distinct salary) from employee 
          where salary>e.salary and departmentid=e.DepartmentId )<3 
   order by e.departmentid,Salary desc
   ~~~

   针对where (select count( distinct salary) from employee 
          where salary>e.salary and departmentid=e.DepartmentId )<3 解释：

   统计employee表中的salary  > e.salary的人数

   如果不加部门id只会输出前三的工资！

   添加部门ID = 1/2则会将其余部门的值（大于目标salary）代入其中（employee .salary 选去的是 第三高的值！）

2. 601体育馆人流量

   ~~~mysql
   #1窗口函数——MySQL不支持
   # 和3相似
   SELECT DISTINCT S.*
   FROM 
   (
   	SELECT S1.id AS `id1`,S2.id AS `id2`,S3.id AS `id3`
   	FROM stadium AS S1
   	JOIN stadium AS S2 ON(S1.id +1 = S2.id AND S1.people >=100 AND S2.people >=100)
   	JOIN stadium AS S3 ON(S2.id +1 = S3.id AND S2.people >=100 AND S3.people >=100)
   ) AS A
   JOIN stadium AS S ON(A.id1 = S.id OR A.id2=S.id OR A.id3=S.id)
   
   作者：jason-2
   链接：https://leetcode-cn.com/problems/human-traffic-of-stadium/solution/san-chong-jie-fa-by-jason-2-2/
   来源：力扣（LeetCode）
   著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
   #2表三次连接
   
   select * from stadium s1 
   where people >= 100
   and (2=(select count(*) from stadium s where s.people >=100 and (s.id = s1.id -1 or s.id = s1.id -2))
          or 2=(select count(*) from stadium s where s.people >=100 and (s.id = s1.id -1 or s.id = s1.id +1))
          or 2=(select count(*) from stadium s where s.people >=100 and (s.id = s1.id +1 or s.id = s1.id +2)))
   
   
   #3自叉积三次
   
   SELECT DISTINCT S1.*
   FROM stadium AS S1,stadium AS S2,stadium AS S3
   WHERE S1.people>=100 AND S2.people>=100 AND S3.people>=100 AND (
   	S1.id +1 = S2.id AND S1.id+2=S3.id OR
   	S1.id +1 = S2.id AND S1.id-1=S3.id OR
   	S1.id -1 = S2.id AND S1.id-2=S3.id
   )
   ORDER BY S1.id
   #不添加order by s1.id 会造成顺序错乱，？？  运行过程？
   ~~~

   