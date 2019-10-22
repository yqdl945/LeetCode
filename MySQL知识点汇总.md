# Leetcode_Mysql

## 考虑代码的复用和套用！

1. case  when then else end和  if 条件句

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

     

2. rank 自定义变量（编序号）！

3. 

   
