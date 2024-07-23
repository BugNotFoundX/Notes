---
title: SQL常用语法
mathjax: true
abbrlink: 39073
date: 2022-09-28 22:46:22
tags:
categories:
cover:
---

## 零、示例用表

- 学生表S

  <figure><table>
  <thead>
  <tr><th>S#</th><th>SN</th><th>SA</th><th>SD</th></tr></thead>
  <tbody><tr><td>s1</td><td>李勇</td><td>20</td><td>CS</td></tr><tr><td>s2</td><td>刘晨</td><td>19</td><td>IS</td></tr><tr><td>s3</td><td>王敏</td><td>18</td><td>MA</td></tr><tr><td>s4</td><td>张立</td><td>19</td><td>IS</td></tr></tbody>
  </table></figure>
  
- 课程表C

  <figure><table>
  <thead>
  <tr><th>C#</th><th>CN</th><th>PC#</th></tr></thead>
  <tbody><tr><td>C1</td><td>数据库</td><td>CS</td></tr><tr><td>C2</td><td>数学</td><td>&nbsp;</td></tr><tr><td>C3</td><td>信息系统</td><td>C1</td></tr><tr><td>C4</td><td>操作系统</td><td>C6</td></tr><tr><td>C5</td><td>数据结构</td><td>C7</td></tr><tr><td>C6</td><td>数据处理</td><td>&nbsp;</td></tr><tr><td>C7</td><td>Pascal语言</td><td>C6</td></tr></tbody>
  </table></figure>

- 学生选课表SC

  <figure><table>
  <thead>
  <tr><th>S#</th><th>C#</th><th>G</th></tr></thead>
  <tbody><tr><td>s1</td><td>C1</td><td>92</td></tr><tr><td>s1</td><td>C2</td><td>85</td></tr><tr><td>s1</td><td>C3</td><td>88</td></tr><tr><td>s2</td><td>C2</td><td>90</td></tr><tr><td>s2</td><td>C3</td><td>80</td></tr></tbody>
  </table></figure>

## 一、SQL数据查询功能

- 查询的基本结构：`SELECT-FROM-WHERE`
  - `SELECT`目标列
  - `FROM`基本表（或视图）
  - `WHERE`检索条件

### （一）投影检索

- 含义：`SELECT-FROM-WHERE`查询语句中没有`WHERE`子句

    ```sql
    #例: 查询学生的姓名和学号
    SELECT SN, SA
    FROM S;
    ```

- 采用`DISTINCT`消去结果中的重复行

    ```sql
    #例: 检索学生所选修课程的课程号
    SELECT DISTINCT C#
    FROM SC;
    ```

### （二）选取检索

- 由`WHERE`子句指出查询条件

- 检索条件可以包含以下的运算符

  - 比较运算符：`=, <>(!=), >, >=, <=​`
  - 布尔运算符：`AND, OR, NOT`
  - `()`

  ```sql
  #例1: 检索选修C2课程的所有学生的学号和成绩
  SELECT S#, G
  FROM SC
  WHERE C# = 'C2';
  
  #例2: 检索选修C1或C2且成绩高于70分的学生学号、课程号和成绩
  SELECT S#, C#, G
  FROM SC
  WHERE (C# = 'C1' OR C# = 'C2') AND G >= 70;
  
  #例3: 检索成绩在70至85分之间的学生学号、课程号和成绩。
  SELECT S#, C#, G
  FROM SC
  WHERE G BETWEEN 70 AND 85;
  ```

### （三）排序检索

- 在SELECT-FROM-WHERE查询块后接ORDER BY子 句，将结果按指定列排序。 

- 格式：`ORDER BY 列名 ASC 或 DESC`

  - `ASC`为升序；`DESC`为降序，缺省为升序 
  - 可以是单列排序或多列排序 
  - 该子句在`SELECT`语句中作为最后一个子句出现

  ```sql
  #例: 检索全体学生信息，并按系号升序，同一个系
  按年龄降序排列。
  SELECT *
  FROM S
  ORDER BY SD, SA DESC;
  ```

### （四）连表检索

- 将多个相互关联的表按照一定条件连接起来，实现多表数据检索
- SELECT-FROM-WHERE语句块结构：
  - SELECT—— 指明选取的列名（来自多个表） 
  - FROM—— 指明要进行连接的表名 
  - WHERE——指明连接条件（连接谓词）与选取条件。

#### 1、一般格式

- 连接条件一般格式为：`[<表名> .] <列名> <比较运算符> [<表名> .] <列名>`

  其中， `<列名>`称为连接字段； `<比较运算符>`主要有：`=，>，<， >=，<=，!=`

  ```sql
  #例1: 检索学生张华所学课程的成绩。
  SELECT SN, C#, G
  FROM S, SC
  WHERE S.S#=SC.S# AND SN='张华';
  # 注：如果连接的表中有属性名相同，要用表名作前缀加以区分。
  ```

#### 2、自身连接

- 表自身的连接：通过定义别名，将一个表看成两个表，进行连接。

  ```sql
  #例2: 检索所有比李勇年龄大的学生姓名、年龄。
  SELECT X.SN, X.SA
  FROM S X, S Y
  WHERE X.SA > Y.SA AND Y.SN = ‘李勇’;
  ```

#### 3、外连接

- 外连接：在连接谓词某一边加（\*或+），则逻辑上为\*所在边的表增加了一个空行。它可以与另一个表中所有不满足连接条件的元组进行连接，使这些元组能够输出。

  ```sql
  #例3: 检索所有学生的全部信息。
  SELECT *
  FROM S, SC
  WHERE S.S# = SC.S#(*);
  ```

### （五）子查询嵌套检索

- WHERE子句中可以包含另一个查询块，该查询块称为**子查询或嵌套查询**，包含子查询的语句称为**外部查询** 
- 外部查询利用子查询来获取检索条件的条件值，检索条件根据子查询的结果来确定外部查询的结果数据 
- 子查询按照与外部查询的联系不同，分为普通子查询和相关子查询 
  - 普通子查询：与外部查询无关，可单独执行得一组值
  - 相关子查询：把外查询的列值作为检索条件的条件值

#### 1、涉及同一个表的子查询

- 如果子查询返回单值，可以直接用比较运算符`=,<>,>,>=,<,<=`等连接子查询。

- 如果子查询返回一组值，则必须在比较运算符和子 查询之间插入`ANY、ALL`等操作符。

  ```sql
  #例1: 检索与李勇同岁的学生姓名。
  SELECT SN
  FROM S
  WHERE S.SA = (
  	SELECT SA FROM S 
   	WHERE SN='李勇');
  #例2: 检索选修C2课程的学生姓名。
  SELECT SN
  FROM S
  WHERE S# = ANY(
  	SELECT S# FROM SC 
  	WHERE C#=‘C2’);
  #例3: 检索选修C2课程的成绩最高的学生学号。
  SELECT S#
  FROM SC
  WHERE C# = ‘C2’ AND G >= ALL(
  	SELECT G FROM SC 
  	WHERE C# = 'C2');
  ```

#### 2、用IN/NOT IN检索

- IN在嵌套子查询中最常使用。可代替`=ANY`，是集合运算 中的$\in$运算。

- NOT IN表示不在集合中，与`!=ALL`相同，是集合运算 中的$\notin$运算。

  ```sql
  #例4: 检索选修C2课程的学生姓名。
  SELECT SN
  FROM S
  WHERE S# IN (
  	SELECT S# FROM SC 
  	WHERE C#=‘C2’);
  ```

#### 3、用EXISTS检索

- EXISTS表示存在量词$\exists$

- 表达式 EXISTS（子查询）当且仅当子查询结果为**非空**时为真

  ```sql
  #例5: 检索选修C2课程的学生姓名。
  SELECT SN
  FROM S
  WHERE EXISTS(
  	SELECT * FROM SC 
  	WHERE S#=S.S# AND C#=‘C2’);
  ```

#### 4、用NOT EXISTS 检索

- 表示“不存在”。 

- 表达式NOT EXISTS（子查询）在子查询结果为**空**时为真。

- 用NOT EXISTS表达全称量词（$\forall$）

  - 任何一个带有全称量词的谓词总可以转换为等价的带存在量词的谓词：$(\forall x)P \equiv \neg(\exists x(\neg P))$

  ```sql
  #例6：检索选修所有课程的学生姓名。
  # 本题等价于“检索这样的学生的姓名，不存在他不选修的课程”。
  SELECT SN
  FROM S
  WHERE NOT EXISTS( # S.S#不选修的课程(不存在S.S#选修记录的课程） 
      SELECT * FROM C 
   	WHERE NOT EXISTS( # S.S#选修C.C#的记录
          SELECT * FROM SC
   		WHERE S#=S.S# AND C#=C.C#));
  ```

- 用NOT EXISTS 表达蕴涵

  - $p \rightarrow q \equiv \neg p \vee q$
  - $(\forall y) p \rightarrow q \equiv \neg(\exists y(\neg(p \rightarrow q))) \equiv \neg(\exists y(\neg(\neg p \vee q))) \equiv \neg \exists y(p \wedge \neg q)$

  ```sql
  # 例7：检索至少选修了学生S2选修的全部课程的学生学号。
  # 本题用蕴含表达：“检索这样学生的学号，对于任意课程，只要S2选修，他就选修。”
  # 本题等价于“检索这样学生的学号，不存在S2选修而他没有选修的课程”。
  SELECT DISTINCT S#
  FROM SC SCX
  WHERE NOT EXISTS( # S2选修而SCX.S#没有选修的课程（S2选修的课程并且不存在SCX.S#选修该课程的记录）
      SELECT * FROM SC SCY
   	WHERE SCY.S# = ‘S2’ AND NOT EXISTS ( # SCX.S#选修SCY.C#的记录
      	SELECT * FROM SC SCZ
   		WHERE S# = SCX.S# AND C# = SCY.C#));
  ```


### （六）并差交检索

- 并、差、交的SQL运算符： 

  - 并：UNION 
  - 差：MINUS 
  - 交：INTERSECT

-  并、差、交检索的操作对象必须是相容的， 是同类关系，即必须有相同数量的属性列， 且相应属性列的域也必须相同

  ```sql
  # eg1：检索选修了C1或C2课程的学生学号。
  SELECT S# FROM SC WHERE C#='C1'
  UNION 
  SELECT S# FROM SC WHERE C#='C2';
  
  #例2：检索无人选修的课程号和名称。
  SELECT C#，CN FROM C WHERE C# IN(
      SELECT C# FROM C 
   	MINUS
   	SELECT DISTINCT C# FROM SC);
  ```

### （七）库函数检索

- 库（集）函数 

  - `COUNT()` 按列值计个数，`COUNT(*)`对行记数 
  - `SUM()` 对数值列求总和
  - `AVG() `求数值列的平均值
  - `MAX() `在列中找出最大值
  - `MIN() `在列中找出最小值

- 只能在SELECT子句以及HAVING子句中出现

  ```sql
  #例1：检索学生总数。
  SELECT COUNT(*) FROM S;
  
  #例2：检索选修了课程的学生人数。
  SELECT COUNT(DISTINCT S#) FROM SC;
  
  #例3：求学号为S4的学生的总分和平均分。
  SELECT SUM(G)，AVG(G)
  FROM SC 
  WHERE S#='S4';
  
  #例4：检索选修了C1课程的学生最高分。
  SELECT MAX(G) FROM SC WHERE C#='C1';
  ```

### （八）分组检索

- 按属性列（列组）将关系的元组分组，每组在这些分组属性列（列组）上具有相同值，对每一组执行SELECT操作。 

- 分组子句： `GROUP BY 列名 [HAVING 条件表达式] ——分组条件 `

- WHERE子句与HAVING子句 

  - WHERE 子句是针对“行”进行，用于去掉不符合条件的若干行
  - HAVING子句针对“分组”进行，必须和GROUP BY 连用，用 于去掉不符合条件的若干分组。
  - 在查询块中出现的顺序：`WHERE — GROUP BY — HAVING`

  ```sql
  #例1：检索至少选修三门课程的学生学号和选课门数。
  SELECT S#，COUNT(*)
  FROM SC 
  GROUP BY S#
  HAVING COUNT(*) >=3;
  
  #例2：求选修四门以上课程的学生学号和总成绩（不统计不及格的课程）。最后按降序列出总成绩排序名单。
  SELECT S#，SUM（G）
  FROM SC 
  WHERE G>=60
  GROUP BY S#
  HAVING COUNT(*) >=4
  ORDER BY SUM(G) DESC;
  ```

### （九）算术表达式值的检索

- SELECT子句中，可包括由属性列、常数、库函数、算 术运算符`+-*/`等组成的算术表达式。 

- 检索结果数据项名可用表达式表示或用“别名”来表示。

  ```sql
  #例1：有职工表EMP( EMP#，EMPN，JOB，SALARY，BONUS，DEPT )，
  要求检索所有PROGRAMMER的奖金大于工资25%的职工姓名和一年的总收入，并按奖金与工资之比的降序排列。
  SELECT EMPN，BONUS/SALARY BS, 12*（SALARY+BONUS） TOTAL
  FROM EMP
  WHERE JOB='PROGRAMMER'
  AND BONUS>0.25*SALARY 
  ORDER BY BONUS/SALARY DESC;
  ```

### （十）部分匹配查询

- 使用谓词LIKE 或NOT LIKE，一般形式： `<列名> LIKE/NOT LIKE <字符串常量>` 

  - – <列名>必须为字符型或变长字符型。 
  - – <字符串常量>可包含两个特殊符号 % 与_ 
    - %：代表任意序列的0个或多个字符； 
    - _ ：代表任意单个字符

  ```sql
  #例1：检索所有姓刘的学生的学号、姓名。
  SELECT S#, SN
  FROM S
  WHERE SN LIKE '刘%';
  ```

### （十一）基于派生表的查询

- 当子查询出现在FROM子句中，则子查询生成的表称 为临时派生表，该表也可作为主查询的操作对象

- AS 关键字可以省略，但必须为派生表指定别名

- 如果子查询中没有库函数，则派生表可不指定列

  ```sql
  #例1：检索每个学生超出自己平均成绩的课程号。
  SELECT S#, C#
  FROM SC, (
  	SELECT S#, AVG(G) 
  	FROM SC 
  	GROUP BY S#) AS AVG_SC(AVG_S#,AVG_G)
  WHERE SC.S#=AVG_SC.AVG_S# AND SC.G>=AVG_SC.AVG_G
  ```

## 二、SQL数据定义功能

### （一）定义基本表

- 基本语句

  ```
  Create Table <表名> (
  	<列名><数据类型>[<列级完整性约束>] 
  	[{,<列名><数据类型>[<列级完整性约束>]}] 
  	[{, [<表级完整性约束>]}]
  ); 
  ```

- 完整性约束 

  - NULL/NOT NULL
  - UNIQUE
  - PRIMARY KEY
  - FOREIGN KEY
  - CHECK

- SQL92的数据类型

  - char（n）：固定长度的字符串
  - varchar（n）：可变长字符串
  - int：整数
  - smallint：小整数类型
  - numeric（p，q）：定点数共p位，小数点右边q位
  - Real, double precision ：浮点数与双精度浮点数,精 度与机器有关
  - Float(n)：n位的精度浮点数。 
  - date：日期（年、月、日）
  - time：时间（小时、分、秒）
  - interval：两个date或time类型数据之间的差

  ```sql
  Create table S(
  	S# char(5) not null unique,
  	SN char(20) not null,
  	SA int,
  	SD char(15),
  	Primary key(S#),
  	Check (SA >=18 and SA <=45)
  );
  
  Create table SC(
  	S# char(5) not null,
  	C# char(5) not null,
  	G number(4,2),
  	Primary key (S#, C#),
  	Foreign key (S#) references S (S#),
  	Foreign key (C#) references C (C#)
  );
  ```

### （二）修改、删除基本表定义

- 基本语句

  ```sql
  #修改基本表
  Alter Table <表名>
  [Add <新列名><数据类型>[<完整性约束>]]
  [Drop <完整性约束名>]
  [Modify <列名><数据类型>]
  
  #删除基本表
  Drop Table <表名>
  ```

  ```sql
  #例1：S表增加“入学时间”属性
  Alter Table S Add Scome Date;
  #例2：将SA的数据类型改为半字长整数
  Alter Table S Modify SA Smallint;
  #例3：删除s表
  Drop Table S;
  ```

### （三）定义、删除索引

- 基本语法

  ```sql
  #定义索引
  Create [Unique][Cluster] Index <索引名> On <表名> (<列名>[次序][, <列名>[次序]] …);
  #删除索引
  Drop Index <索引名>
  ```

  ```sql
  Create Unique Index Scno On SC(S# ASC, C# DESC)
  
  Drop Index Stusno;
  ```

### （四）定义视图

- 基本语法及示例

  ```sql
  #定义视图
  ##语法： 
  Create View <视图名>
  [(<列名>[,<列名>] …)]
  As <子查询>
  [With Check Option]
  
  ##示例：建立计算机系的学生视图
  Create View CS_Student
  As
  SELECT S#, SN, SA FROM S
  WHERE SD = ‘CS’ ；
  
  #删除视图
  Drop View <视图名>
  ```

### （五）查询视图

- 参考基本表的查询



## 三、SQL数据更新

### （一）插入数据

- 基本语法及示例

  ```sql
  #插入单个元组
  Insert Into <表名>[(<属性列>[{,<属性列>}])]
  Values(<值>[{,<值>}])
  ##示例：Insert Into S Values (‘S10’, ‘陈冬’, ‘IS’, 18);
  
  #插入子查询结果
  Insert Into <表名>[(<属性列>[{,<属性列>}])] <子查询>
  ##示例
  Insert Into Dept_Age (Sdept, Avgage)
  Select SD, AVG(SA) From S
  Group By SD;
  ```

### （二）修改数据

- 基本语法及示例

  ```sql
  #语法
  Update <表名>
  Set <列名>=<表达式>[{, <列名>=<表达式>}]
  [Where <条件>];
  #示例：将学生S1的年龄改为22岁
  Update S
  Set SA = 22
  Where S#=‘S1’;
  ```

### （三）删除数据

- 基本语法及示例

  ```sql
  #语法 
  Delete From <表名> [Where <条件>];
  #示例：
  – 例1：删除学号为S19的学生的记录
  Delete From S Where S#=‘S19’;
  – 例2：删除所有学生的选课记录
  Delete From SC;
  – 例3：删除计算机系所有学生的选课记录
  Delete From SC
  Where ‘CS’ = (
  	Select SD From S
  	Where S .S# = SC.S#);
  ```

## 四、SQL触发器（MySql）

- 基本语法及示例

	```sql
	#语法
	DELIMITER //
	CREATE TRIGGER [触发器的名字]
	[触发器执行时机] [触发器监测的对象]
	ON [表名]
	FOR EACH ROW 
	begin
	[触发器主体代码]
	end //
	DELIMITER ;
	
	#示例
	DELIMITER //
	CREATE TRIGGER `20373821`.`Tage_trigger`
	BEFORE INSERT ON `20373821`.`teacher`
	FOR EACH ROW
	begin
		if new.Tage <= 24 then
			SIGNAL SQLSTATE '45000'
			set MESSAGE_TEXT = '教师年龄应大于24';
		end if;
	end //
	DELIMITER;
	```
	
- 基本语法解释

	- `DELIMITER //`：MySQL 默认分隔符是`;` 但在触发器中，我们使用 `//` 表示触发器的开始与结束。
	- `[触发器的名字]`：这里填写触发器的名字
	- `[触发器执行时机]`：这里设置触发器是在关键动作执行之前触发，还是执行之后触发。
	- `[触发器监测的对象]`：触发器可以监测 `INSERT`、`UPDATE`、`DELETE` 的操作，当监测的命令对触发器关联的表进行操作时，触发器就被激活了。
	- `[表名]`：将这个触发器与数据库中的表进行关联，触发器定义在表上，也附着在表上，如果这个表被删除了，那么这个触发器也随之被删除。
	- `FOR EACH ROW`：这句表示只要满足触发器触发条件，触发器都会被执行，也就是说带上这个参数后，触发器将监测每一行对关联表操作的代码，一旦符合条件，触发器就会被触发。
	- `[触发器主体代码]`：这里是当满足触发条件后，被触发执行的代码主体。这里可以是一句 SQL 语句，也可以是多行命令。如果是多行命令，那么这些命令要写在 `BEGIN...END` 之间。
- 在创建触发器主体时，还可以使用`OLD`和`NEW` 来获取 SQL 执行`INSERT`，`UPDATE`和`DELETE` 操作前后的写入数据。
	
- 参考链接https://zhuanlan.zhihu.com/p/439273702

## 五、SQL数据控制功能

- 定义完整性约束条件：支持事务操作、提供安全控制功能 
  - 授权 `GRANT <权限> [ON <对象类型> <对象名>] TO <用户>`
  - 收回权限 `REVOKE <权限> [ON <对象类型> <对象名>]  FROM <用户>`