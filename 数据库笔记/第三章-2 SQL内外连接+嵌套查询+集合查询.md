## 连接查询
![[连接查询.png]]
执行过程:
先在表一中找到第一个元组，然后从头开始扫描表二,逐一查找满足连接件的元组，找到后就将表一中的第一个元组拼接起来，形成结果表第一个元组
表二全部查找完后，再找表1中第二个元组,然后再从头开始扫描表二，查找满足条件的元组,找到后将表一中的第二个元组与其拼接，形成结果表中的一个元组，以此类推直到表一所有元组处理完毕
### 广义笛卡尔积
eg:
Select Student.\*,sc.* From Student,sc;
### 等值连接
![[等值连接.png]]
### 自然连接
选中重复项的一列即可
### 非等值连接
![[非等值连接.png]]
### 自身连接
- 一个表自己与自己连接
- 需要给表取别名
- 必须使用别名前缀
eg:
![[自身连接.png]]
### 外连接
[https://blog.csdn.net/plg17/article/details/78758593]
一、内连接
关键字：inner join on
语句：select * from a_table a inner join b_table bon a.a_id = b.b_id;
执行结果：
![[内连接.png]]
说明：组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。
![[内连接解析.png]]
二、左连接（左外连接）
关键字：left join on / left outer join on
语句：select * from a_table a left join b_table bon a.a_id = b.b_id;
执行结果：
![[左外连接DATA.png]]
说明：
left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。
![[左外连接.png]]
左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

三、右连接（右外连接）
关键字：right join on / right outer join on
语句：select * from a_table a right outer join b_table b on a.a_id = b.b_id;
执行结果：
![[右外连接 1.png]]
说明：
right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。
与左(外)连接相反，右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。
![[数据库/右外连接.png]]
总的来说，左右连接会在原有的基础上返回完整左/右表,且不满足条件的右/左表数据置NULL
## 嵌套查询
### 基础须知
- 一个SELECT-FROM-WHERE语句被称为一个查询块
- 将一个查询块嵌套在另外一个查询块的WHERE或HAVING短语中的查询被称为嵌套查询
- 子查询的限制:不能使用ORDER BY 
- 层层嵌套的方式反映了SQL语言的结构化
- 有些嵌套查询可以用连接运算替代
### 分类
不相关子查询:子查询的查询条件不依赖于父查询
相关子查询:子查询的查询条件依赖于父查询
### 使用原则
- 一个子查询必须放在圆括号中
- 将子查询放在比较条件右边可以增加可读性，子查询不包含ORDER BY 语句
- 在子查询中可以使用两种比较条件:单行运算法和多行运算符
### ANY与ALL
- ANY:任意一个值
- ALL:所有值
eg:
![[ANY.png]]
## 集合查询
标准SQL支持:UNION
商用SQL支持:UNION INTERSECT EXCEPT
#### UNION
![[union.png]]
![[UNION SAMPLE.png]]
#### INTERSECT
![[INTERSECT SAMPLE.png]]
#### EXCEPT
![[EXCEPT Sample.png]]
### 对集合操作结果的排序
- ORDER BY 子句只能用于对最终查询结果排序,不能对最终结果排序
- 任何情况下,ORDER BY 子句只能出现在最后
- 对集合操作结果排序时,ORDER BY子句中用数字指定排序属性
![[Pasted image 20251219212329.png]]
这里图中,左边是错的,右边是对的。或者只写一个ORDER BY Sno;
### 集函数
![[Pasted image 20251219212455.png]]