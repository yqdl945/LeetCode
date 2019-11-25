# Leetcode_Mysql

## 考虑复用、善用聚合函数

1. MySQL 执行顺序 from where select 

2. case  when then else end和  if 条件句

   ~~~mysql
   # 以262为例：
   case when a.Status ！= 'completed' then 1 else 0 end
   if(a.status != 'completed',1,0)
   ~~~

   - if 和 case when  双层嵌套

     ~~~mysql
     # 以626为例
     if (mod(id,2),id-1,(if id = (select max(id) from seat),id,id+1))
     
     case when mod(id,2) = 0 then id-1 when id = (select max(id) from seat then id else id+1 end)
     ~~~

     

3. rank 自定义变量（编序号）！

   https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/bb1b72a1623e

   - @ 用于声明变量

   - 3.1普通排名（连续不重复）

     - ~~~mysql
       # 1 在select 语句中使用，在from语句中
       # from 语句后跟两个表（别名表和子表没有任何关联）无需使用where 语句找两者之间的关系
       SELECT pid, name, age, @curRank := @curRank + 1 AS rank
       FROM players p, (
       SELECT @curRank := 0
       ) q
       ORDER BY age
       # 2 直接使用 set 在语句前声明变量
       SET @curRank := 0;
       SELECT pid, name, age, @curRank := @curRank + 1 AS rank
       FROM players
       ORDER BY age
       ~~~

   - 3.2 Rank普通并列排名函数(并列——连续)

     - ~~~mysql
       # 设立2 个变量 （看情况定义（正常情况下一个是0，一个null)
       #使用if/case when 结构
       SELECT pid, name, age, 
       CASE 
       WHEN @prevRank = age THEN @curRank #比较？？？
       WHEN @prevRank := age THEN @curRank := @curRank + 1 # 赋值
       END AS rank
       FROM players p, 
       (SELECT @curRank :=0, @prevRank := NULL) r
       ORDER BY age
       
       ~~~

   - 3.3 Rank高级并列排名函数（并列——不连续）

     - ~~~mysql
       # 设立三个变量 （0 null 1）
       SELECT pid, name, age, rank FROM
       (SELECT pid, name, age,#2 循环（顺序不能乱）
       @curRank := IF(@prevRank = age, @curRank, @incRank) AS rank, 
       @incRank := @incRank + 1, 
       @prevRank := age
       FROM players p, (
       SELECT @curRank :=0, @prevRank := NULL, @incRank := 1
       ) r #1 两个子表 r p
       ORDER BY age) 
       ~~~

       

   ~~~mysql
   # 178 分数排名
   # 1 两表比较 （超过b表score 的人数）
   select a.score as score ,(select count(distinct b.score) from scores b
                            where b.score > a.score) rank from scores a 
   order by a.score desc
   
   #2 
   ~~~

   := 和 = 的区别

   - := 用作赋值/变量实现行号（不会解释为比较运算符）

     [1]: https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.htm	"MySQL"

     

   - 仅在set 和update时与:= 作用相同，其余时间表示相等！

   4. cast用法

      字符串与其他等类型的交换

   ​	用法cast(字段 as 转换类型)

| char     | 字符      |
| -------- | --------- |
| date     | 日期      |
| datetime | 日期+时间 |
| decimal  | 浮点型    |
| signed   | 整型      |
| time     | 时间      |

5. 组信息匹配

   ~~~mysql
   # 以1077 项目员工II为例
   #选择目标信息——（先对数据进行预处理，最后进行匹配输出）
   
   
   ~~~

   