## SQL的基本概念与特点
- 综合统一
- 高度非过程化
- 面向集合的操作方式
- 以同一种语法结构提供两种使用方法
- 语言简洁，易学易用
![[SQL语言的动词.png]]
## 创建和使用数据表
![[SQL的数据定义语句.png]]
### 创建
![[CREATE TABLE.png]]
- **列级约束**：是针对**单个列**定义的约束，直接写在该列的字段定义（类型、长度等）之后。
- **表级约束**：是针对**一个或多个列**定义的约束，写在所有列的字段定义之后（通常用 `CONSTRAINT` 显式声明）。
#### 常用完整性约束:
- 主码约束:PRIMARY KEY
- **作用**：唯一标识表中的**每一行数据**（相当于 “行的唯一身份证”），自带`NOT NULL + UNIQUE`的特性。
- 唯一性约束:UNIQUE
- 非空值约束:NOT NULL
- 参照完整性约束(外键约束，`FOREIGN KEY`)
- **示例**：`SC`表的`Foreign Key(Sno) REFERENCES Student(Sno)` → `SC`表的`Sno`（选课学生的学号），必须是`Student`表中已存在的学号（不能填一个不存在的学生学号）。
##### PRIMARY KEY与UNIQUE 异同
- 相同点:通过建立唯一索引来保证基本表在主键列取值的唯一性
- 不同点:
1. 一个基本表中只能定义一个PRIMARY KEY约束;但可以定义多个UNIQUE约束
2. 指定为PRIMARY KEY的一个列或多个列的组合,其中任何一列都不能出现NULL值;而对于UNIQUE所约束的唯一键,则允许为NULL;
3. 不能为同一列或一组既定义为UNIQUE约束,又定义为PRIMARY KEY约束
### 修改表
![[修改表.png]]
ADD新增列均为NULL
##### 删除列
删除属性列分2种
1. 间接删除属性列
2. 直接删除
###### 间接删除:
- 第一步：新建一个表（比如叫 New_Student），只把**想保留的列**（Sno、Sname、Ssex）和对应的数据复制过去：  
 ```sql
CREATE TABLE New_Student AS
SELECT Sno, Sname, Ssex FROM Student;  -- 只复制要保留的列
```  
- 第二步：删除原来的 Student 表： 
```sql
DROP TABLE Student;
```
- 第三步：把新表改回原表名（New_Student 改叫 Student）：
 ```sql
ALTER TABLE New_Student RENAME TO Student;
```    
这样操作后，Student 表就没有`Sfin`列了 —— 相当于 “间接删掉了 Sfin”。
###### 直接删除:
```sql
ALTER TABLE Student DROP Sfin;
```
##### 删除表
```sql
DROP TABLE <表名>;
```
## 创建和使用索引
![[数据库/索引.png]]
#### 注意事项:
1. 对于已含重复值的属性列不能建UNIQUE索引
2. 对某个列建立UNIQUE索引后，插入新记录时，DBMS会自动检查新纪录再该列上是否取了重复值。
#### 删除索引
```sql
DROP INDEX <索引名>
```
## 数据查询
![[查询语句.png]]
![[查询续.png]]

#### 单表查询
##### 函数

| `LOWER(str)`                | 将字符串转为小写                 | `SELECT LOWER(Sname) AS name_lower FROM Student;`<br><br>（把学生姓名转为小写）                                                                                             |
| --------------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `UPPER(str)`                | 将字符串转为大写                 | `SELECT UPPER(name) AS name_upper FROM Customer;`<br><br>（把客户姓名转为大写）                                                                                             |
| `LENGTH(str)`               | 计算字符串长度（字符数）             | `SELECT Sname, LENGTH(Sname) AS name_len FROM Student;`<br><br>（统计姓名长度）                                                                                          |
| `CONCAT(str1, str2, ...)`   | 拼接多个字符串                  | `SELECT CONCAT(Sname, '(', Sdept, ')') AS name_dept FROM Student;`<br><br>（拼接姓名 + 院系）                                                                            |
| `SUBSTRING(str, pos, len)`  | 截取子串（pos 从 1 开始）         | `SELECT SUBSTRING(Sno, 1, 4) AS sno_prefix FROM Student;`<br><br>（截取学号前 4 位）                                                                                     |
| `REPLACE(str, old, new)`    | 替换字符串中的指定字符              | `SELECT REPLACE(Sdept, '计算机', 'CS') AS dept_short FROM Student;`<br><br>（把 “计算机” 替换为 CS）                                                                         |
| `ABS(num)`                  | 取绝对值                     | `SELECT Sage, ABS(Sage - 20) AS diff_20 FROM Student;`<br><br>（计算年龄与 20 的绝对差值）                                                                                   |
| `ROUND(num, n)`             | 四舍五入（n 为小数位）             | `SELECT score, ROUND(score, 1) AS score_round FROM Score;`<br><br>（分数保留 1 位小数）                                                                                   |
| `CEIL(num)`                 | 向上取整（进一）                 | `SELECT price, CEIL(price) AS price_ceil FROM Product;`<br><br>（商品价格向上取整）                                                                                        |
| `FLOOR(num)`                | 向下取整（去尾）                 | `SELECT price, FLOOR(price) AS price_floor FROM Product;`<br><br>（商品价格向下取整）                                                                                      |
| `MOD(num1, num2)`           | 取模（余数）                   | `SELECT Sage, MOD(Sage, 2) AS is_odd FROM Student;`<br><br>（判断年龄奇偶：1 = 奇，0 = 偶）                                                                                  |
| `SQRT(num)`                 | 平方根                      | `SELECT score, SQRT(score) AS score_sqrt FROM Score;`<br><br>（计算分数的平方根）                                                                                          |
| `IS NULL/IS NOT NULL`       | 判断字段是否为 NULL（运算符，非函数）    | `SELECT name FROM Customer WHERE referee_id IS NULL;`<br><br>（查询无推荐人的客户）                                                                                         |
| `IF(condition, val1, val2)` | 三元判断：满足条件返回 val1，否则 val2 | `SELECT Sname, Sage, IF(Sage >= 18, '成年', '未成年') AS age_type FROM Student;`<br><br>（判断是否成年）                                                                      |
| `IFNULL(str, val)`          | 字段为 NULL 时返回 val，否则返回原字段 | `SELECT name, IFNULL(referee_id, '无推荐人') AS referee FROM Customer;`<br><br>（替换 NULL 为 “无推荐人”）                                                                    |
| `CASE WHEN`                 | 多条件判断（类似 if-else if）     | `SELECT Sname, Sage, <br> CASE <br> WHEN Sage < 18 THEN '未成年' <br> WHEN Sage BETWEEN 18 AND 22 THEN '青年' <br> ELSE '成年' <br> END AS age_group <br>FROM Student;` |
| `COUNT(*)`                  | 统计总行数（含 NULL）            | `SELECT COUNT(*) AS total_student FROM Student;`<br><br>（统计学生总数）                                                                                                 |
| `COUNT(col)`                | 统计字段非 NULL 的行数           | `SELECT COUNT(referee_id) AS has_referee FROM Customer;`<br><br>（统计有推荐人的客户数）                                                                                     |
| `SUM(col)`                  | 求和                       | `SELECT SUM(score) AS total_score FROM Score WHERE Sno = '2023001';`<br><br>（统计某学生总分）                                                                            |
| `AVG(col)`                  | 求平均值                     | `SELECT AVG(Sage) AS avg_age FROM Student WHERE Sdept = '计算机';`<br><br>（统计计算机系平均年龄）                                                                              |
| `MAX(col)/MIN(col)`         | 求最大 / 最小值                | `SELECT MAX(Sage) AS max_age, MIN(Sage) AS min_age FROM Student;`<br><br>（学生最大 / 最小年龄）                                                                           |
| DISTINCT                    | 筛选非重复                    | 注意:DISTINCT作用范围是所有目标列,不能DISTINCT Cno,DISTINCT Grade                                                                                                              |
#### WHERE
![[WHERE.png]]
#### IN
![[IN.png]]
#### BETWEEN
![[BETWEEN.png]]
#### \[NOT] LIKE '<字符串>'
由此引出通配符
'%'代表任意长度
'\_'代表任意单个字符,注意中文占2个字符!!!
#### 空值
用IS \[NOT] NULL
#### 多重查询
AND OR NOT
#### 排序
ORDER BY --> ASC升序DESC降序,默认升序
多个列在ORDER BY 内有先后
eg:
![[ORDER.png]]
#### GROUP BY
用于分组
- 作用对象是查询的中间结果
- 分组方法:指定一列或多列分组，相同为一组
- 使用GROUP BY子句后，SELECT子句的列名列表中只能出现分组属性和集函数
#### HAVING
对最终结果再次筛选
![[HAVING.png]]
#### WHERE与HAVING的不同
WHERE作用于基表或视图，从中选择满足条件的元组
HAVING短语作用于组，从中选择满足条件的组