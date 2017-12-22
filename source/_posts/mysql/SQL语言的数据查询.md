---
title: SQL语言的数据查询
date: 2016-10-27 21:22:50
tags: 
- 数据库
categories: 总结
---
此篇为上一篇的姊妹篇，感觉一不小心记的又臭又长了，不过内容确实不是一篇文章能说的清的。

<!--more-->

SELECT语句的一般格式为:
```
//格式一
SELECT〈列名〉[{，〈列名〉}]
FROM〈表名或视图名〉[{，〈表名或视图名〉}]
[WHERE〈检索条件〉]
[GROUP BY <列名1>[HAVING <条件表达式>]]
[ORDER BY <列名2>[ASC|DESC]];
//格式二
SELECT  [ALL|DISTINCT][TOP N [PERCENT][WITH TIES]]
   列名1 [AS 别名1]
[, 列名2 [ AS 别名2]…]
[INTO 新表名]
FROM 表名 1[[AS] 表1别名]
[INNER|RIGHT|FULL|OUTER][OUTER]JOIN
    表名2 [[AS] 表2别名]
ON 条件
```
查询结束时仍然是一个表。

SELECT语句的执行过程是:
根据WHERE子句的检索条件，从FROM子句指定的基本表或视图中选取满足条件的元组，再按照SELECT子句中指定的列，投影得到结果表。
如果有GROUP子句，则将查询结果按照<列名1>相同的值进行分组。
如果GROUP子句后有HAVING短语，则只输出满足HAVING条件的元组。
如果有ORDER子句，查询结果还要按照<列名2>的值进行排序。

### 投影查询
在查询中不使用WHERE子句的无条件查询，也称投影查询。
```
//查询选修了课程的学生号
SELECT DISTINCT SNO FROM SC
```
利用投影查询可控制列名的顺序，并可通过指定别名改变查询结果的列标题的名字。
```
//查询全体学生的姓名、学号和年龄。
SELECT SNAME NAME, SNO, AGE FROM S
```
其中，NAME为SNAME的别名。
### 条件查询
当要在表中找出满足某些条件的行时，则需使用WHERE子句指定查询条件。

WHERE子句中，条件通常通过三部分来描述：
1.列名；
2.比较运算符；
3.列名、常数。

比较运算符，除了常规的大于小于等于之外，还有下列这些常用的比较运算符：

不等于(<>)；多重条件(and,or);BETWEEN AND(确定范围)；IN(确定集合)；LIKE(字符匹配)；IS NULL（空值）
#### 1.比较大小
```
SELECT SNO,CNO,SCORE FROM SC WHERE SCORE>85
```
#### 2.多重条件查询
当WHERE子句需要指定一个以上的查询条件时，则需要使用逻辑运算符AND、OR和NOT将其连结成复合的逻辑表达式。
其优先级由高到低为：NOT、AND、OR，用户可以使用括号改变优先级。
```
//查询选修C1或C2且分数大于等于85分学生的的学号、课程号和成绩。
SELECT SNO，CNO，SCORE
   FROM SC
WHERE（CNO=’C1’ OR CNO=’C2’） AND
 SCORE>=85 
```
#### 3.确定范围
```
//查询工资在1000至1500之间的教师的教师号、姓名及职称。
SELECT TNO,TN,PROF
   FROM T
WHERE SAL BETWEEN 1000 AND 1500
//等价于
SELECT TNO,TN,PROF
   FROM T
WHERE SAL>=1000 AND SAL<=1500
```
#### 4.确定集合
利用“IN”操作可以查询属性值属于指定集合的元组。

```
//查询选修C1或C2的学生的学号、课程号和成绩。
SELECT SNO, CNO, SCORE 
   FROM SC 
WHERE CNO IN(‘C1’, ‘C2’)
//此语句也可以使用逻辑运算符“OR”实现。
//利用“NOT IN”可以查询指定集合外的元组。
```
#### 5.部分匹配查询
上例均属于完全匹配查询，当不知道完全精确的値时，用户还可以使用LIKE或NOT LIKE进行部分匹配查询（也称模糊查询）。
LIKE定义的一般格式为：
```
<属性名> LIKE <字符串常量>
```
属性名必须为字符型，字符串常量的字符可以包含如下两个特殊符号：
%：表示任意知长度的字符串；

_：表示任意单个字符;

\[charlist]:字符列中的任何单一字符;

\[^charlist]或者[!charlist]:不在字符列中的任何单一字符。
```
//查询所有姓张但不包含国或辽的教师的教师号和姓名。
select  * 
 from S
where sname like '张[^国辽]_'
```
#### 6.空值查询
某个字段没有值称之为具有空值（NULL）。空值不同于零和空格，它不占任何存储空间。

例如，某些学生选课后没有参加考试，有选课记录，但没有考试成绩，考试成绩为空值，这与参加考试，成绩为零分的不同。
```
SELECT SNO, CNO
   FROM SC
WHERE SCORE IS NULL
```
### 常用库函数及统计汇总查询
|函数名称|功能
|------|------|
|AVG|按列计算平均值|
|SUM|按列计算值的总和|
|MAX|求一列中的最大值|
|MIN|求一列中的最小值|
|COUNT|按列值计个数|

```
//求学号为S1学生的总分和平均分。
SELECT SUM(SCORE) AS TotalScore, AVG(SCORE) AS AveScore
   FROM SC
WHERE (SNO = 'S1') 

//求学校中共有多少个系
SELECT COUNT(DISTINCT DEPT) AS DeptNum 
   FROM S
```
注意：
函数SUM和AVG只能对数值型字段进行计算。

COUNT函数对空值不计算，但对零进行计算。
### 分组查询
`GROUP BY`子句可以将查询结果按属性列或属性列组合在行的方向上进行分组，每组在属性列或属性列组合上具有相同的值。
```
查询各位教师的教师号及其任课的门数。
SELECT TNO,COUNT(*) AS C_NUM
  FROM TC
GROUP BY TNO 
```
若在分组后还要按照一定的条件进行筛选，则需使用HAVING子句。
```
SELECT SNO,COUNT(*) AS SC_NUM 
  FROM SC
GROUP BY SNO       
HAVING COUNT(*)>=2 

```
当在一个SQL查询中同时使用WHERE子句，`GROUP  BY `子句和`HAVING`子句时，其顺序是`WHERE－GROUP  BY－ HAVING`。

**`WHERE`与`HAVING`子句的根本区别在于作用对象不同。**

WHERE子句作用于基本表或视图，从中选择满足条件的元组；
HAVING子句作用于组，选择满足条件的组，必须用于GROUP BY子句之后，但GROUP BY子句可没有HAVING子句。

### 查询排序
当需要对查询结果排序时，应该使用ORDER BY子句
ORDER BY子句必须出现在其他子句之后
排序方式可以指定，DESC为降序，ASC为升序，缺省时为升序
```
//查询选修C2、C3、C4或C5课程的学号、课程号和成绩，查询结果按学号升序排列，学号相同再按成绩降序排列。
SELECT SNO,CNO, SCORE 
   FROM SC
WHERE CNO IN ('C2' ,'C3', 'C4','C5')
   ORDER BY SNO,SCORE DESC 
//求选课在三门以上且各门课程均及格的学生的学号及其总    成绩，查询结果按总成绩降序列出。

SELECT SNO,SUM(SCORE)AS TotalScore
FROM SC
WHERE SCORE>=60
GROUP BY SNO
HAVING COUNT(*)>=3
ORDER BY SUM(SCORE) DESC
```
上述执行过程如下：取出SC，筛选SCORE>=60 的元组，讲选出的元组按照SNO分组，筛选选课三门以上的分组，一剩下的组中提取学号和总成绩，将选取结果排序。

另外，`ORDER BY SUM(SCORE) DESC`可以改写成
`ORDER BY 2 DESC`,2 代表查询结果的第二列。 

### INTO 子句
在默认子段组中创建一个新表并将来自查询的结果行插入新表中。
```
SELECT *
     INTO new_Table
   FROM S
WHERE Sex = ‘男’

```
### COMPUTE子句
生成合计作为附加的汇总列出现在结果集的最后。
```
SELECT s.s#,sname,age,sex,c#,grade 
   FROM s,sc
WHERE s.s#=sc.s# and
               s.s#='200203080101'
COMPUTE sum(sc.grade),count(s.s#)
```
### 数据表连接及连接查询
数据表之间的联系是通过表的字段值来体现的，这种字段称为连接字段。
连接操作的目的就是通过加在连接字段的条件将多个表连接起来，以便从多个表中查询数据。
前面的查询都是针对一个表进行的，当查询同时涉及两个以上的表时，称为连接查询。

表的连接方法有两种：
1.表之间满足一定的条件的行进行连接，此时FROM子句中指明进行连接的表名，`WHERE`子句指明连接的列名及其连接条件。
2.利用关键字`JOIN`进行连接。

具体分为下列几种：
1.INNER JOIN：现在符合条件的记录，此为默认值。
2.LEFT （OUTER） JOIN：显示符合条件的数据行以及左边表中不符合条件的数据行，此时右边数据行会以NULL来显示，此称为左连接。
3.RIGHT （OUTER） JOIN：显示符合条件的数据行以及右边表中不符合条件的数据行，此时左边数据行会以NULL来显示，此称为右连接。
4.FULL （OUTER） JOIN：显示符合条件的数据行以及左边表和右边表中不符合条件的数据行，此时缺乏数据的数据行会以NULL来显示。
5.CROSS JOIN：会将一个表的每一笔数据和另一表的每笔数据匹配成新的数据行。

当将`JOIN`关键词放于FROM子句中时，应有关键词`ON`与之相对应，以表明连接的条件。

#### 等值连接与非等值连接
查询刘伟老师所讲授的课程。

方法1：
```
SELECT T.TNO ,TN,CNO
   FROM T,TC
WHERE (T.TNO = TC. TNO) AND (TN = ‘刘伟’)

```
上面的操作是将T表中的TNO 和TC表中的TNO相等的行连接，同时选取TN为“刘伟“的行，然后再在TN，CNO列上投影，这是连接、选取和投影的操作组合。

这里，TN=‘刘伟’为查询条件，而T.TNO = TC.TNO 为连接条件，TNO为连接字段。连接条件的一般格式为：
```
[<表名1>.] <列名1> <比较运算符> [<表名2>.] <列名2> 
```
其中,比较运算符主要有：＝、＞、＜、＞＝、＜＝、！＝。
当比较运算符为“＝“时，称为等值连接，其他情况为非等值连接。

当列名是唯一的，不需要加表名前缀。

方法2：
```
SELECT T.TNO,TN,CNO
   FROM T INNER JOIN TC 
    ON T.TNO=TC.TNO AND T.TN='刘伟'

```
方法3：
```
SELECT R2.TNO,R2.TN, R1.CNO 
   FROM
      (SELECT TNO,CNO 
          FROM TC ) AS R1
       INNER JOIN 
       (SELECT TNO ,TN 
            FROM T
         WHERE TN='刘伟') AS R2
        ON R1.TNO=R2.TNO

```
#### 自身连接

查询所有比刘伟工资高的教师姓名、性别、工资和刘伟的工资。

要查询的内容均在同一表T中，可以将表T分别取两个别名，一个是X，一个是Y。将X, Y 中满足比刘伟工资高的行连接起来。这实际上是同一表T的自身连接。
方法1：
```
SELECT X.TN,X.SAL AS SAL_a, Y.SAL AS SAL_b 
   FROM T AS X ,T AS Y 
WHERE X.SAL>Y.SAL AND 
    Y.TN='刘伟'

```
方法2：
```
SELECT X.TN, X.SAL,Y.SAL 
   FROM T AS X INNER JOIN T AS Y
      ON X.SAL>Y.SAL AND Y.TN='刘伟'
```
方法3：
```
SELECT R1.TN,R1.SAL, R2.SAL 
   FROM 
    (SELECT TN,SAL 
         FROM T ) AS R1
    INNER JOIN 
    (SELECT SAL 
         FROM T
      WHERE TN='刘伟') AS R2
     ON R1.SAL>R2.SAL

```
#### 外连接
在上面的连接操作中，不满足连接条件的元组不能作为查询结果输出。
```
//查询所有学生的学号、姓名、选课名称及成绩。（没有选课的同学的选课信息显示为空）则应写成如下的SQL语句。
SELECT S.SNO,SN,CN,SCORE
   FROM S
       LEFT OUTER JOIN SC
       ON S.SNO=SC.SNO
       LEFT OUTER JOIN C
       ON C.CNO=SC.CNO
```
则查询结果只包括所有的学生，没有选课的同学的选课信息显示为空。

### 子查询
在WHERE子句中包含一个形如SELECT-FROM-WHERE的查询块，此查询块称为子查询或嵌套查询，包含子查询的语句称为父查询或外部查询。

子查询的嵌套层次最多可达到255层，以层层嵌套的方式构造查询充分体现了SQL“结构化”的特点。

嵌套查询在执行时由里向外处理，每个子查询是在上一级外部查询处理之前完成，父查询要用到子查询的结果。

#### 返回一组值的子查询
如果子查询的返回值不止一个，而是一个集合时，则不能直接使用比较运算符，可以在比较运算符和子查询之间插入ANY或ALL。
```
//查询讲授课程号为C5的教师姓名。
SELECT TN
   FROM T
WHERE TNO=ANY
      (SELECT TNO 
          FROM TC
       WHERE CNO='C5')
//该例也可以使用前面所讲的连接操作来实现：
SELECT TN
   FROM T,TC
WHERE T.TNO=TC.TNO AND
               TC.CNO='C5‘

```
可见，对于同一查询可使用子查询和连接两种方法来解决，可根据习惯任意选用。 

另外，还可以使用IN代替“=ANY”。
```
SELECT TN
   FROM T
WHERE TNO IN
    (SELECT TNO 
        FROM TC
     WHERE CNO='C5')
```

```
SELECT DISTINCT TN
   FROM T
WHERE 'C5' !=ALL
     ( SELECT CNO 
           FROM TC
        WHERE TNO=T.TNO)
```
`!=ALL`的含义为不等于子查询结果中的任何一个值，也可使用`NOT IN`代替`!=ALL`。

前面所讲的子查询均为普通子查询，而本例中子查询的查询条件引用了父查询表中的属性值（T表的TNO值），我们把这类查询称为相关子查询。

二者的执行方式不同，相关子查询并不是先执行子查询，而是首先选取父查询表的第一行记录，内部的子查询利用此行中相关的属性值进行查询。
然后父查询根据子查询返回的结果判断此行是否满足查询条件。如果满足条件，则把该行放入父查询的查询结果集合中。重复执行这一过程，直到处理完父查询表中的每一行数据。

如上行所述，表T中每一行都要执行一次子查询。
因此，相关子查询的执行次数是由父查询表的行数决定的。

除了之前说的ALL,ANY,IN之外，还可以使用EXISTS来表示存在与否。带有EXISTS的子查询不返回任何实际数据，它只得到逻辑值“真”或“假”。

```
查询讲授课程号为C5的教师姓名。
SELECT TN
   FROM T
WHERE EXISTS
    (SELECT * 
        FROM TC
     WHERE TNO=T.TNO
    AND CNO='C5')
```
当子查询TC表存在一行记录满足其WHERE子句中的条件时，则父查询便得到一个TN值，重复执行以上过程，直到得出最后结果。

### 集合查询
集合操作分为并操作UNION,交操作INTERSECT,差操作EXCEPT。

参加集合操作的各查询结果的列数必须相同；对应项的数据类型也必须相同。
```
SELECT *
   FROM S
WHERE dept= 'CS'
  UNION
SELECT *
   FROM S
WHERE age<=19
//等于
SELECT DISTINCT  *
   FROM S
WHERE dept= 'CS'  OR  age<=19；

```
UNION：将多个查询结果合并起来时，系统自动去掉重复元组。  `or`
UNION ALL：将多个查询结果合并起来时，保留重复元组
INTERSECT：求两个结果的交集 `and`
EXCEPT:差集

### SQL数据更新 
SQL语言的数据更新语句DML主要包括插入数据、修改数据和删除数据三种语句。

#### 插入数据记录
```
//插入单行记录
INSERT INTO <表名>[(<列名1>[,<列名2>…])] VALUES(<值>)
//插入多行记录
INSERT INTO <表名> [(<列名1>[,<列名2>…])] 子查询
```

#### 修改数据记录
```
//UPDATE <表名> SET <列名>=<表达式> [,<列名>=<表达式>] … [WHERE <条件>]
//利用子查询提供要修改的值
UPDATE T
SET SAL =
(SELECT 1.2*AVG(SAL) 
FROM T)
```
当需要多行修改多行时，
```
//多行修改多行
UPDATE <表1> a
inner join <表2> b ON <两表的关系>
SET a.<列名>=b.<列名>
WHERE
<条件>

//记得提交！
COMMIT;

```

#### 删除数据记录
```
DELETE 
    FROM <表名>
[WHERE <条件>]
//利用子查询选择要删除的行
DELETE
    FROM TC 
 WHERE TNO=  
(SELECT TNO
    FROM T
 WHERE TN=’ 刘伟’)
```
### 正则匹配

MYSQL可以使用正则表达式进行匹配。使用的运算符是 REGEXP。

|模式 |模式匹配对象
|-------|----------|
|^  |字符串的开始位置|
|$  |字符串的结尾|
|.  |单个字符|
|[...]  |一对方括号之间的字符|
|[^...] |未在一对方括号之间的字符|
|p1丨p2丨p3|  交替匹配模式1、模式2或模式3|
|*  |匹配前面元素的零个或多个实例|
|+  |匹配前面元素的一个或多个实例|
|{n}  |匹配前面元素的n个实例|
|{m,n}  |匹配前面元素的m~n个实例，m <= n|

### 大小写敏感

要想让搜索对大小写敏感，可以如下例一般，使用 BINARY 关键字。
```
SELECT p_Name name FROM persons WHERE p_Name REGEXP BINARY 'j';
```