# 写在前面
本笔记基于《MySQL必知必会》,[runoob](https://www.runoob.com/mysql/mysql-tutorial.html) 与我自己的理解，仅供学习交流用。
# 基础知识
表:特定类型数据的结构化清单  
列:表中的一个字段  
数据类型:容器中存储的数据的类型(int,char)  
行:表中的一个记录  
主键:行用来标识自己的列  
examples:  

| id  | 雇员名字 |
| --- | ---- |
| 1   | 大头儿子 |
| 2   | 小头爸爸 |
| 3   | 围裙妈妈 |
在这个表中,id可以看作是主键,[1,大头儿子]可以看作是一个行,[1,2,3]可以看作是一个列,[id,雇员名字]可以看作是一个表头
# 基础使用
#### 使用MySQL
在命令行中,登录到mysql的基础命令通常为  
`mysql -u [username] -p`  
如果有需要的话,可以加入参数 -h [ip or hostname]  
如果要使用某个数据库,其命令为  
`USE [database];`  
若要显示有哪些库,可以用命令  
`Show databses;`  
列出所选数据库中的所有表,可以用命令   
`Show tables`  
列出表中的列,可以用命令    
`Show columns from [table]`
#### 检索数据
在MySQL中,检索数据常用`SELECT`语句实现  
basic examples:  
`SELECT 列1, 列2, ... FROM 表名`
##### 关键字
 关键字是在用命令中可选的选项,下面给出几个，可以帮助检索数据  
 * DISTINCT  
	当采用`DISTINCT`关键词时，会折叠相同的内容  
	examples:  
	`SELECT price from food;`
	显示为
	
| price    |
| -------- |
| 3 dollar |
| 3 dollar |
| 4 dollar |
| 5 dollar |
| 5 dollar |
| 5 dollar |
采用`SELECT DISTINCT price from food;`  
则显示为

| price    |
| -------- |
| 3 dollar |
| 4 dollar |
| 5 dollar |
**注:** 当采用`DISTINCT`时,若对象包含多列，只有两列的组合完全一样时,才会被省略  
* LIMIT  
当采用`LIMIT+空格+数值`时,会限制输出的行数,默认从头部开始  
其余用法:  
当采用`LIMIT [start point],[length]`时,它代表从[start point]开始,向后索引[length]行  
**注:**有时候会出现关键词`OFFEST`,用法如下:  
`LIMIT A OFFEST B`  
这代表从行B开始取A行
##### 限定引用
* 完全限定引用
  我们可以通过同时使用表名和列名来进行引用  
  例如:  
  `SELECT products.name from market.products;`  
其中 products即为表名,name为列名,market为数据库名
* 通配符  
你可以用\*来表示通配符  
例如:  
`SELECT * from [tables];`  
这会展示[tables]里面的所有列  
*通配符的更多知识会在后面提到*
#### 排序数据
在SQL中使用`SELECT`语句,其返回默认为其添加到表的顺序,或者是经过排序后的底层表的顺序,我们可以用`ORDER BY`子句来排序返回的数据  
例如:  
`SELECT * FROM products ORDER BY product_name ASC;`  
这句话代表按照`product_name`展示所有数据,而关键词ASC代表升序,相反的,DESC代表降序。若为char型,则按字母排序,int型按照大小排序。  
**于此同时**,ORDER BY 可以**指定多列**,例如:  
`SELECT * FROM employees ORDER BY department_id ASC, hire_date DESC;`  
代表选择员工表 employees 中的所有员工，并先按部门 ID 升序 ASC 排序，然后在相同部门中按雇佣日期降序 DESC 排序。  
不仅如此,ORDER BY 也可以指定相对位置
例如:  
`SELECT first_name, last_name, salary FROM employees ORDER BY 3 DESC, 1 ASC;`  
代表选择员工表 employees 中的名字和工资列，并先按第三列降序 DESC 排序，然后按第一列升序 ASC 排序。  
更多用法:
通过**LIMIT**+**ORDER BY** 可以实现对最大值的查找:  
examples:`Select food_price from products order by food_price DESC LIMIT 1;`
#### 过滤数据
在SQL中,我们常使用WHERE语句进行过滤  
例如:  
`Select product_name,product_price from products where product_price = 2`  
这个语句代表了从`product`中筛选`product_price=2`的行，并且返回其`product_name`,`product_price`  
##### 条件运算符
下面是一些条件运算符,可以用于`WHERE`语句

| 运算符     | 说明     |
| ------- | ------ |
| =       | 等于     |
| <>      | 不等于    |
| !=      | 不等于    |
| <       | 小于     |
| >       | 大于     |
| <=      | 小于等于   |
| >=      | 大于等于   |
| BETWEEN | 在...之间 |
**注:** 在使用BETWEEN的时候,要搭配AND关键词  
例如:`Select product_name,product_price from products where product_price between 5 and 10;`就是在`products`中筛选`product_price`在`5-10`之间的行
##### 空值检查
在检查空值时,采用子句 `IS NULL`,用法与条件运算符类似  
examples:`Select product_name from products where products_price is null;`  
#### 高级数据过滤
* AND OR 与 ()
在过滤数据时,我们还可以用运算符` AND `和` OR `进行过滤  
examples:  
`select product_name,product_price from products where factory_id =1002 OR factory_id = 1003 AND product_price>5`  
**注:AND的优先级大于OR!!!**  
所以这句话代表筛选由`factory_id = 1003`生产且**价格大于5**和`factory_id=1002`生产的产品的`product_name`,`product_price`  
如果想筛选**价格大于5**的`factory_id = 1003`生产的和**价格大于5**的`factory_id = 1002`生产的可以用`括号()`进行分组  
代码如下:  
`select product_name,product_price from products where (factory_id =1002 OR factory_id = 1003) AND product_price>5`  
* IN
  IN的效果与OR类似,但是可以查找更多数据  
  例如:  
`  SELECT * FROM t_user WHERE uid IN (1,2,3,4,5,6,7,8,9);`  
可以查询t_user中uid=1,2,3,4,5,6,7,8,9的数据  
* NOT  
  NOT与其英文释义相同,代表否定
下面给出一个例子:  
  `SELECT * FROM Customers WHERE NOT Country='Germany';`  
代表了国家不为Germany的customers  
通过NOT和IN的结合,可以轻松实现数据的反选  
例如:  
`SELECT ID FROM city WHERE ID NOT IN (SELECT ID FROM city WHERE ID > 11 AND ID < 100);`  
实现了查询 city 表中 ID 不在 10 和 100 之间的所有值  
#### 通配符:
通配符使用来匹配值的一部分的字符  
***注:若要使用通配符进行匹配,就一定要使用`LIKE`谓词***
##### 百分号(%)通配符
%表示匹配任意数量的任意字符  
例如:  
`selsct product_name from products where product_name like jet%;`  
这会输出products中product_name以jet为开头的的product_name  
例如 jetwolf.jetcat,jetdog  
以此类推  
`%jet`会筛选结尾为jet的数据  
`%jet%`则会匹配字段中带有jet的部分  
`j%et`会匹配以**j**为开头,**et**为结尾的字段
  ##### 下划线(\_)通配符
  下划线通配符与%不同,它仅会匹配一个字符  
  例如:  
  `selsct product_name,product_num from products where product_num like % gallon%;`
  假设他的输出为
  

| product_name | product_num |
| ------------ | ----------- |
| gas          | 1 gallon    |
| water        | .5 gallon   |
| oil          | 9 gallon    |
那么,`selsct product_name,product_num from products where product_num like _ gallon%;`的输出为

| product_na | product_nu |
| ---------- | ---------- |
| gas        | 1 gallon   |
| oil        | 9 gallon   |

####  正则表达式
正则表达式在数据检索中，有着相当大的作用,在SQL中，可以用`REGEXP`替换`LIKE`来使用它  
##### 基本用法  
`REGEXP '.000'` 其中,`.`代表匹配任意字符,作用与%类似  
`REGEXP '1000|2000'` 其中,`|`代表和,会匹配1000和2000的数据  
`REGEXP '[123] TON'`,其中`[]`代表匹配1或者2或者3 + 后续内容 `[123]`等价于`[1|2|3]`,你也可以使用`^`来否定，例如`[^123]`代表匹配1,2,3之外的数  
你还可以使用数据范围例如`[0-9]`代表0,1,2,3,4,5,6,7,8,9。`[a-z]`代表所有字母。  
如果要匹配` . | `等字符,可以用转义符号`\\`,例如`\\. `代表搜索含.的项  
注:在正则表达式中,不区分大小写,若要区分,可以使用 `REGEXP BINARY`
##### 字符类
为了方便匹配常用的数字，字母，可以采用字符类
 **[:character_class:]**  
 alnum代表文字数字字符  
alpha代表文字字符 
blank代表空白字符  
cntrl代表控制字符  
digit代表数字字符  
graph代表图形字符  
lower代表小写文字字符  
print代表图形或空格字符  
punct代表标点字符  
space代表空格、制表符、新行、和回车  
upper代表大写文字字符  
xdigit代表十六进制数字字符  
##### 元字符
元字符可以匹配特定的内容  

| 元字符   | 作用          | 示例 SQL                                                   | 匹配示例                         | 不匹配示例                   |
| ----- | ----------- | -------------------------------------------------------- | ---------------------------- | ----------------------- |
| `*`   | 重复 0 次或多次   | `SELECT * FROM text WHERE content REGEXP 'go*gle';`      | "ggle"（o 重复 0 次）、"google"    | "gogle"（中间 o 不连续，需连续）   |
| `+`   | 重复 1 次或多次   | `SELECT * FROM products WHERE sku REGEXP '^[0-9]+$';`    | "123"、"4567"                 | "A12"（含字母）、"1A3"（含字母）   |
| `?`   | 重复 0 次或 1 次 | `SELECT * FROM docs WHERE title REGEXP 'colou?r';`       | "color"（u 省略）、"colour"（u 存在） | "coloor"（u 重复 2 次）      |
| `{n}` | 精确重复 n 次    | `SELECT * FROM codes WHERE value REGEXP '^[A-Z]{3}$';`   | "ABC"、"XYZ"（均为 3 个大写字母）      | "AB"（长度 2）、"abcd"（长度 4） |
| ^     | 匹配字符串开头     | `SELECT * FROM users WHERE name REGEXP '^A';`            | "Alice", "Amy"               | "Bob", "aaron"          |
| $     | 匹配字符串结尾     | `SELECT * FROM logs WHERE path REGEXP '\.html$';`        | "index.html", "file.html"    | "image.jpg", "page.htm" |
| .     | 匹配任意单个字符    | `SELECT * FROM data WHERE code REGEXP 'a.c';`            | "abc", "aXc", "a3c"          | "ac", "abbc"            |
| \|    | 逻辑“或”       | `SELECT * FROM colors WHERE name REGEXP '^(red\|blue)';` | "red", "blue"                | "green", "yellow"       |
