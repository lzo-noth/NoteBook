# SQL必知必会语法基础篇2

## 子查询

### 什么是关联子查询,什么是非关联子查询

子查询虽然是一种嵌套查询的形式,不过我们依然可以**依据子查询是否执行多次**,从而**将子查询划分为关联子查询和非关联子查询** 

子查询从数据表中查询了数据结果,如果这个数据结果只执行一次,然后这个数据结果作为主查询的条件进行执行,那么这样的子查询叫做非关联子查询 同样,如果子查询需要执行多次,即采用循环的方式,先从外部查询开始,每次都传入子查询进行查询,然后再将结果反馈给外部,这种嵌套的执行方式就称为关联子查询

```sql
SQL: SELECT player_name, height FROM player WHERE height = (SELECT max(height) FROM player)
```

```sql
SELECT player_name, height, team_id FROM player AS a WHERE height > (SELECT avg(height) FROM player AS b WHERE a.team_id = b.team_id)
```

### EXISTS 子查询

关联子查询通常也会和 EXISTS 一起来使用,EXISTS 子查询用来判断条件是否满足,满足的话为 True,不满足为 False

```sql
SQL：SELECT player_id, team_id, player_name FROM player WHERE EXISTS (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)
```

### 集合比较子查询
![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled.png)


```sql
SELECT player_id, team_id, player_name FROM player WHERE player_id in (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)
```

### 将子查询作为计算字段

```sql
SQL: SELECT team_name, (SELECT count(*) FROM player WHERE player.team_id = team.team_id) AS player_num FROM team
```

## SQL连接

### 常用的 SQL 标准有哪些

在正式开始讲连接表的种类时,我们首先需要知道 SQL 存在不同版本的标准规范,因为不同规范下的表连接操作是有区别的 SQL 有两个主要的标准,分别是 **SQL92 和 SQL99** 

92 和 99 代表了标准提出的时间,SQL92 就是 92 年提出的标准规范 当然除了 SQL92 和 SQL99 以外,还存在 SQL-86 SQL-89 SQL:2003 SQL:2008 SQL:2011 和 SQL:2016 等其他的标准 这么多标准,到底该学习哪个呢?实际上最重要的 SQL 标准就是 SQL92 和 SQL99 一般来说 SQL92 的形式更简单,但是写的 SQL 语句会比较长,可读性较差 而 SQL99 相比于 SQL92 来说,语法更加复杂,但可读性更强 我们从这两个标准发布的页数也能看出,SQL92 的标准有 500 页,而 SQL99 标准超过了 1000 页 实际上你不用担心要学习这么多内容,基本上从 SQL99 之后,很少有人能掌握所有内容,因为确实太多了 就好比我们使用 Windows Linux 和 Office 的时候,很少有人能掌握全部内容一样 我们只需要掌握一些核心的功能,满足日常工作的需求即可

### 在 SQL92 中是如何使用连接的

**笛卡尔积**

笛卡尔乘积是一个数学运算。假设我有两个集合 X 和 Y，那么 X 和 Y 的笛卡尔积就是 X 和 Y 的所有可能组合，也就是第一个对象来自于 X，第二个对象来自于 Y 的所有可能。

```sql
SQL: SELECT * FROM player, team
```

**等值连接**

两张表的等值连接就是用两张表中都存在的列进行连接。我们也可以对多张表进行等值连接。

针对 player 表和 team 表都存在 team_id 这一列，我们可以用等值连接进行查询。

```sql
SQL: SELECT player_id, player.team_id, player_name, height, team_name FROM player, team WHERE player.team_id = team.team_id
```

**非等值连接**

当我们进行多表查询的时候，如果连接多个表的条件是等号时，就是等值连接，其他的运算符连接就是非等值查询。

```sql
SQL：SELECT p.player_name, p.height, h.height_level
FROM player AS p, height_grades AS h
WHERE p.height BETWEEN h.height_lowest AND h.height_highest
```

**外连接**

除了查询满足条件的记录以外，外连接还可以查询某一方不满足条件的记录。两张表的外连接，会有一张是主表，另一张是从表。如果是多张表的外连接，那么第一张表是主表，即显示全部的行，而第剩下的表则显示对应连接的信息。在 SQL92 中采用（+）代表从表所在的位置，而且在 SQL92 中，只有左外连接和右外连接，没有全外连接。

什么是左外连接，什么是右外连接呢？

左外连接，就是指左边的表是主表，需要显示左边表的全部行，而右侧的表是从表，（+）表示哪个是从表。

```sql
SQL：SELECT * FROM player, team where player.team_id = team.team_id(+)
```

相当于 SQL99 中的：

```sql
SQL：SELECT * FROM player LEFT JOIN team on player.team_id = team.team_id
```

右外连接，指的就是右边的表是主表，需要显示右边表的全部行，而左侧的表是从表。

```sql
SQL：SELECT * FROM player, team where player.team_id(+) = team.team_id
```

相当于 SQL99 中的：

```sql
SQL：SELECT * FROM player RIGHT JOIN team on player.team_id = team.team_id
```

需要注意的是，LEFT JOIN 和 RIGHT JOIN 只存在于 SQL99 及以后的标准中，在 SQL92 中不存在，只能用（+）表示。

**自连接**

**自连接可以对多个表进行操作，也可以对同一个表进行操作**。也就是说查询条件使用了当前表的字段。

比如我们想要查看比布雷克·格里芬高的球员都有谁，以及他们的对应身高

```sql
SQL：SELECT b.player_name, b.height FROM player as a , player as b WHERE a.player_name = '布雷克-格里芬' and a.height < b.height
```

### SQL99是如何使用连接的，与SQL92的区别是什么？

**交叉连接**

交叉连接实际上就是 SQL92 中的笛卡尔乘积，只是这里我们采用的是 CROSS JOIN。

我们可以通过下面这行代码得到 player 和 team 这两张表的笛卡尔积的结果：

```sql
SQL: SELECT * FROM player CROSS JOIN team
```

**自然连接**

你可以把自然连接理解为 SQL92 中的等值连接。它会帮你自动查询两张连接表中所有相同的字段，然后进行等值连接。

如果我们想把 player 表和 team 表进行等值连接，相同的字段是 team_id。还记得在 SQL92 标准中，是如何编写的么？

```sql
SELECT player_id, a.team_id, player_name, height, team_name FROM player as a, team as b WHERE a.team_id = b.team_id
```

在 SQL99 中你可以写成：

```sql
SELECT player_id, team_id, player_name, height, team_name FROM player NATURAL JOIN team
```

实际上，在 SQL99 中用 NATURAL JOIN 替代了 WHERE player.team_id = team.team_id。

**ON 连接**

ON 连接用来指定我们想要的连接条件，针对上面的例子，它同样可以帮助我们实现自然连接的功能：

```sql
SELECT player_id, player.team_id, player_name, height, team_name FROM player JOIN team ON player.team_id = team.team_id
```

这里我们指定了连接条件是ON player.team_id = team.team_id，相当于是用 ON 进行了 team_id 字段的等值连接。

当然你也可以 ON 连接进行非等值连接，比如我们想要查询球员的身高等级，需要用 player 和 height_grades 两张表：

```sql
SQL99：SELECT p.player_name, p.height, h.height_level
FROM player as p JOIN height_grades as h
ON height BETWEEN h.height_lowest AND h.height_highest
```

这个语句的运行结果和我们之前采用 SQL92 标准的查询结果一样。

```sql
SQL92：SELECT p.player_name, p.height, h.height_level
FROM player AS p, height_grades AS h
WHERE p.height BETWEEN h.height_lowest AND h.height_highest
```

一般来说在 SQL99 中，我们需要连接的表会采用 JOIN 进行连接，ON 指定了连接条件，后面可以是等值连接，也可以采用非等值连接。

**USING 连接**

当我们进行连接的时候，可以用 USING 指定数据表里的同名字段进行等值连接。比如：

```sql
SELECT player_id, team_id, player_name, height, team_name FROM player JOIN team USING(team_id)
```

你能看出与自然连接 NATURAL JOIN 不同的是，USING 指定了具体的相同的字段名称，你需要在 USING 的括号 () 中填入要指定的同名字段。同时使用 JOIN USING 可以简化 JOIN ON 的等值连接，它与下面的 SQL 查询结果是相同的：

```sql
SELECT player_id, player.team_id, player_name, height, team_name FROM player JOIN team ON player.team_id = team.team_id
```

### **外连接**

SQL99 的外连接包括了三种形式：

左外连接：LEFT JOIN 或 LEFT OUTER JOIN

右外连接：RIGHT JOIN 或 RIGHT OUTER JOIN

全外连接：FULL JOIN 或 FULL OUTER JOIN

我们在 SQL92 中讲解了左外连接、右外连接，在 SQL99 中还有全外连接。全外连接实际上就是左外连接和右外连接的结合。在这三种外连接中，我们一般省略 OUTER 不写。

1. 左外连接

**SQL92**

```sql
SELECT * FROM player, team where player.team_id = team.team_id(+)
```

**SQL99**

```sql
SELECT * FROM player LEFT JOIN team ON player.team_id = team.team_id
```

2. 右外连接

**SQL92**

```sql
SELECT * FROM player, team where player.team_id(+) = team.team_id
```

**SQL99**

```sql
SELECT * FROM player RIGHT JOIN team ON player.team_id = team.team_id
```

3. 全外连接

**SQL99**

```sql
SELECT * FROM player FULL JOIN team ON player.team_id = team.team_id
```

需要注意的是 **MySQL 不支持全外连接**，否则的话全外连接会返回左表和右表中的所有行。当表之间有匹配的行，会显示内连接的结果。当某行在另一个表中没有匹配时，那么会把另一个表中选择的列显示为空值。

也就是说，全外连接的结果 = 左右表匹配的数据 + 左表没有匹配到的数据 + 右表没有匹配到的数据。

**自连接**

自连接的原理在 SQL92 和 SQL99 中都是一样的，只是表述方式不同。

比如我们想要查看比布雷克·格里芬身高高的球员都有哪些，在两个 SQL 标准下的查询如下。

**SQL92**

```sql
SELECT b.player_name, b.height FROM player as a , player as b WHERE a.player_name = '布雷克-格里芬' and a.height < b.height
```

**SQL99**

```sql
SELECT b.player_name, b.height FROM player as a JOIN player as b ON a.player_name = '布雷克-格里芬' and a.height < b.height
```

### **SQL99 和 SQL92 的区别**

至此我们讲解完了 SQL92 和 SQL99 标准下的连接查询，它们都对连接进行了定义，只是操作的方式略有不同。我们再来回顾下，这些连接操作基本上可以分成三种情况：

内连接：将多个表之间满足连接条件的数据行查询出来。它包括了等值连接、非等值连接和自连接。

外连接：会返回一个表中的所有记录，以及另一个表中匹配的行。它包括了左外连接、右外连接和全连接。

交叉连接：也称为笛卡尔积，返回左表中每一行与右表中每一行的组合。在 SQL99 中使用的 CROSS JOIN。

不过 SQL92 在这三种连接操作中，和 SQL99 还存在着明显的区别。

首先我们看下 SQL92 中的 WHERE 和 SQL99 中的 JOIN。

你能看出在 SQL92 中进行查询时，会把所有需要连接的表都放到 FROM 之后，然后在 WHERE 中写明连接的条件。而 SQL99 在这方面更灵活，它不需要一次性把所有需要连接的表都放到 FROM 之后，而是采用 JOIN 的方式，每次连接一张表，可以多次使用 JOIN 进行连接。

```sql
SELECT ...
FROM table1
    JOIN table2 ON table1和table2的连接条件
        JOIN table3 ON table2和table3的连接条件
```

## 视图

### **如何创建，更新和删除视图**

视图作为一张虚拟表，帮我们封装了底层与数据表的接口。它相当于是一张表或多张表的数据结果集。视图的这一特点，可以帮我们简化复杂的 SQL 查询，比如在编写视图后，我们就可以直接重用它，而不需要考虑视图中包含的基础查询的细节。同样，我们也可以根据需要更改数据格式，返回与底层数据表格式不同的数据。

通常情况下，小型项目的数据库可以不使用视图，但是在大型项目中，以及数据表比较复杂的情况下，视图的价值就凸显出来了，它可以帮助我们把经常查询的结果集放到虚拟表中，提升使用效率。理解和使用起来都非常方便。

### **创建视图：CREATE VIEW**

那么该如何创建视图呢？创建视图的语法是：

```sql
CREATE VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

当视图创建之后，它就相当于一个虚拟表，可以直接使用.

**嵌套视图**

当我们创建好一张视图之后，还可以在它的基础上继续创建视图，比如我们想在虚拟表 player_above_avg_height 的基础上，找到比这个表中的球员平均身高高的球员，作为新的视图 player_above_above_avg_height，那么可以写成：

```sql
CREATE VIEW player_above_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player_above_avg_height)
```

### 修改视图：ALTER VIEW

```sql
ALTER VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

### 删除视图：DROP VIEW

```sql
DROP VIEW view_name
```

### 如何使用视图简化 SQL 操作

利用视图完成复杂的连接

利用视图对数据进行格式化

使用视图与计算字段

正确使用视图可以简化复杂的 SQL 查询，让 SQL 更加清爽易用。不过有一点需要注意，视图是虚拟表，它只是封装了底层的数据表查询接口，因此有些 RDBMS 不支持对视图创建索引（有些 RDBMS 则支持，比如新版本的 SQL Server）。

## 存储过程

### 什么是存储过程，如何创建一个存储过程

存储过程的英文是 Stored Procedure。它的思想很简单，就是 SQL 语句的封装。一旦存储过程被创建出来，使用它就像使用函数一样简单，我们直接通过调用存储过程名即可。存储过程实际上由 SQL 语句和流控制语句共同组成。

如何定义一个存储过程：

```sql
CREATE PROCEDURE 存储过程名称([参数列表])
BEGIN
    需要执行的语句
END
```

和视图一样，我们可以删除已经创建的存储过程，使用的是 DROP PROCEDURE。如果要更新存储过程，我们需要使用 ALTER PROCEDURE。

讲完了如何创建，更新和删除一个存储过程，下面我们来看下如何实现一个简单的存储过程。比如我想做一个累加运算，计算 1+2+…+n 等于多少，我们可以通过参数 n 来表示想要累加的个数，那么如何用存储过程实现这一目的呢？这里我做一个 add_num 的存储过程，具体的代码如下：

```sql
CREATE PROCEDURE `add_num`(IN n INT)
BEGIN
       DECLARE i INT;
       DECLARE sum INT;
       
       SET i = 1;
       SET sum = 0;
       WHILE i <= n DO
              SET sum = sum + i;
              SET i = i +1;
       END WHILE;
       SELECT sum;
END
```

当我们需要再次使用这个存储过程的时候，直接使用 CALL add_num(50);即可。

这就是一个简单的存储过程，除了理解 1+2+…+n 的实现过程，还有两点你需要理解，一个是 DELIMITER 定义语句的结束符，另一个是存储过程的三种参数类型。

我们先来看下 DELIMITER 的作用。如果你使用 Navicat 这个工具来管理 MySQL 执行存储过程，那么直接执行上面这段代码就可以了。如果用的是 MySQL，你还需要用 DELIMITER 来临时定义新的结束符。因为默认情况下 SQL 采用（；）作为结束符，这样当存储过程中的每一句 SQL 结束之后，采用（；）作为结束符，就相当于告诉 SQL 可以执行这一句了。但是存储过程是一个整体，我们不希望 SQL 逐条执行，而是采用存储过程整段执行的方式，因此我们就需要临时定义新的 DELIMITER，新的结束符可以用（//）或者（$$）。如果你用的是 MySQL，那么上面这段代码，应该写成下面这样：

```sql
DELIMITER //
CREATE PROCEDURE `add_num`(IN n INT)
BEGIN
       DECLARE i INT;
       DECLARE sum INT;
       
       SET i = 1;
       SET sum = 0;
       WHILE i <= n DO
              SET sum = sum + i;
              SET i = i +1;
       END WHILE;
       SELECT sum;
END //
DELIMITER ;
```

首先我用（//）作为结束符，又在整个存储过程结束后采用了（//）作为结束符号，告诉 SQL 可以执行了，然后再将结束符还原成默认的（;）。

需要注意的是，如果你用的是 Navicat 工具，那么在编写存储过程的时候，Navicat 会自动设置 DELIMITER 为其他符号，我们不需要再进行 DELIMITER 的操作。

我们再来看下存储过程的 3 种参数类型。在刚才的存储过程中，我们使用了 IN 类型的参数，另外还有 OUT 类型和 INOUT 类型，作用如下：

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%201.png)


IN 和 OUT 的结合，既用于存储过程的传入参数，同时又可以把计算结果放到参数中，调用者可以得到返回值。

你能看到，IN 参数必须在调用存储过程时指定，而在存储过程中修改该参数的值不能被返回。而 OUT 参数和 INOUT 参数可以在存储过程中被改变，并可返回。

举个例子，这里会用到我们之前讲过的王者荣耀的英雄数据表 heros。假设我想创建一个存储类型 get_hero_scores，用来查询某一类型英雄中的最大的最大生命值，最小的最大魔法值，以及平均最大攻击值，那么该怎么写呢？

```sql
CREATE PROCEDURE `get_hero_scores`(
       OUT max_max_hp FLOAT,
       OUT min_max_mp FLOAT,
       OUT avg_max_attack FLOAT,  
       s VARCHAR(255)
       )
BEGIN
       SELECT MAX(hp_max), MIN(mp_max), AVG(attack_max) FROM heros WHERE role_main = s INTO max_max_hp, min_max_mp, avg_max_attack;
END
```

你能看到我定义了 4 个参数类型，其中 3 个为 OUT 类型，分别为 max_max_hp、min_max_mp 和 avg_max_attack，另一个参数 s 为 IN 类型。

这里我们从 heros 数据表中筛选主要英雄定位为 s 的英雄数据，即筛选条件为 role_main=s，提取这些数据中的最大的最大生命值，最小的最大魔法值，以及平均最大攻击值，分别赋值给变量 max_max_hp、min_max_mp 和 avg_max_attack。

然后我们就可以调用存储过程，使用下面这段代码即可：

```sql
CALL get_hero_scores(@max_max_hp, @min_max_mp, @avg_max_attack, '战士');
SELECT @max_max_hp, @min_max_mp, @avg_max_attack;
```

### 流控制语句

流控制语句是用来做流程控制的，我刚才讲了两个简单的存储过程的例子，一个是 1+2+…+n 的结果计算，一个是王者荣耀的数据查询，你能看到这两个例子中，我用到了下面的流控制语句：

BEGIN…END：BEGIN…END 中间包含了多个语句，每个语句都以（;）号为结束符。

DECLARE：DECLARE 用来声明变量，使用的位置在于 BEGIN…END 语句中间，而且需要在其他语句使用之前进行变量的声明。

SET：赋值语句，用于对变量进行赋值。

SELECT…INTO：把从数据表中查询的结果存放到变量中，也就是为变量赋值。

除了上面这些用到的流控制语句以外，还有一些常用的流控制语句：

1.IF…THEN…ENDIF：条件判断语句，我们还可以在 IF…THEN…ENDIF 中使用 ELSE 和 ELSEIF 来进行条件判断。

2.CASE：CASE 语句用于多条件的分支判断，使用的语法是下面这样的。

```sql
CASE 
  WHEN expression1 THEN ...
  WHEN expression2 THEN ...
  ...
    ELSE 
    --ELSE语句可以加，也可以不加。加的话代表的所有条件都不满足时采用的方式。
END
```

3.LOOP、LEAVE 和 ITERATE：LOOP 是循环语句，使用 LEAVE 可以跳出循环，使用 ITERATE 则可以进入下一次循环。如果你有面向过程的编程语言的使用经验，你可以把 LEAVE 理解为 BREAK，把 ITERATE 理解为 CONTINUE。

4.REPEAT…UNTIL…END REPEAT：这是一个循环语句，首先会执行一次循环，然后在 UNTIL 中进行表达式的判断，如果满足条件就退出，即 END REPEAT；如果条件不满足，则会就继续执行循环，直到满足退出条件为止。

5.WHILE…DO…END WHILE：这也是循环语句，和 REPEAT 循环不同的是，这个语句需要先进行条件判断，如果满足条件就进行循环，如果不满足条件就退出循环。

我们之前说过 SQL 是声明型语言，使用 SQL 就像在使用英语，简单直接。今天讲的存储过程，尤其是在存储过程中使用到的流控制语句，属于过程性语言，类似于 C++ 语言中函数，这些语句可以帮我们解决复杂的业务逻辑。

### 关于存储过程使用的争议

尽管存储过程有诸多优点，但是对于存储过程的使用，一直都存在着很多争议，比如有些公司对于大型项目要求使用存储过程，而有些公司在手册中明确禁止使用存储过程，为什么这些公司对存储过程的使用需求差别这么大呢？

我们得从存储过程的特点来找答案。

你能看到存储过程有很多好处。

首先存储过程可以一次编译多次使用。存储过程只在创造时进行编译，之后的使用都不需要重新编译，这就提升了 SQL 的执行效率。其次它可以减少开发工作量。将代码封装成模块，实际上是编程的核心思想之一，这样可以把复杂的问题拆解成不同的模块，然后模块之间可以重复使用，在减少开发工作量的同时，还能保证代码的结构清晰。还有一点，存储过程的安全性强，我们在设定存储过程的时候可以设置对用户的使用权限，这样就和视图一样具有较强的安全性。最后它可以减少网络传输量，因为代码封装到存储过程中，每次使用只需要调用存储过程即可，这样就减少了网络传输量。同时在进行相对复杂的数据库操作时，原本需要使用一条一条的 SQL 语句，可能要连接多次数据库才能完成的操作，现在变成了一次存储过程，只需要连接一次即可。

基于上面这些优点，不少大公司都要求大型项目使用存储过程，比如微软、IBM 等公司。但是国内的阿里并不推荐开发人员使用存储过程，这是为什么呢？

存储过程虽然有诸如上面的好处，但缺点也是很明显的。

它的可移植性差，存储过程不能跨数据库移植，比如在 MySQL、Oracle 和 SQL Server 里编写的存储过程，在换成其他数据库时都需要重新编写。

其次调试困难，只有少数 DBMS 支持存储过程的调试。对于复杂的存储过程来说，开发和维护都不容易。

此外，存储过程的版本管理也很困难，比如数据表索引发生变化了，可能会导致存储过程失效。我们在开发软件的时候往往需要进行版本管理，但是存储过程本身没有版本控制，版本迭代更新的时候很麻烦。

最后它不适合高并发的场景，高并发的场景需要减少数据库的压力，有时数据库会采用分库分表的方式，而且对可扩展性要求很高，在这种情况下，存储过程会变得难以维护，增加数据库的压力，显然就不适用了。

## 事务处理

### **事务的特性：ACID**

我刚才提到了事务的特性：要么完全执行，要么都不执行。不过要对事务进行更深一步的理解，还要从事务的 4 个特性说起，这 4 个特性用英文字母来表达就是 ACID。

A，也就是原子性（Atomicity）。原子的概念就是不可分割，你可以把它理解为组成物质的基本单位，也是我们进行数据处理操作的基本单位。

C，就是一致性（Consistency）。一致性指的就是数据库在进行事务操作后，会由原来的一致状态，变成另一种一致的状态。也就是说当事务提交后，或者当事务发生回滚后，数据库的完整性约束不能被破坏。

I，就是隔离性（Isolation）。它指的是每个事务都是彼此独立的，不会受到其他事务的执行影响。也就是说一个事务在提交之前，对其他事务都是不可见的。

最后一个 D，指的是持久性（Durability）。事务提交之后对数据的修改是持久性的，即使在系统出故障的情况下，比如系统崩溃或者存储介质发生故障，数据的修改依然是有效的。因为当事务完成，数据库的日志就会被更新，这时可以通过日志，让系统恢复到最后一次成功的更新状态。

ACID 可以说是事务的四大特性，在这四个特性中，原子性是基础，隔离性是手段，一致性是约束条件，而持久性是我们的目的。原子性和隔离性比较好理解，这里我讲下对一致性的理解（国内很多网站上对一致性的阐述有误，具体你可以参考 Wikipedia 对[Consistency](https://en.wikipedia.org/wiki/ACID)的阐述）。

我之前讲到过数据表的 7 种常见约束。这里指的一致性本身是由具体的业务定义的，也就是说，任何写入数据库中的数据都需要满足我们事先定义的约束规则。

比如说，在数据表中我们将姓名字段设置为唯一性约束，这时当事务进行提交或者事务发生回滚的时候，如果数据表中的姓名非唯一，就破坏了事务的一致性要求。所以说，事务操作会让数据表的状态变成另一种一致的状态，如果事务中的某个操作失败了，系统就会自动撤销当前正在执行的事务，返回到事务操作之前的状态。

事务的另一个特点就是持久性，持久性是通过事务日志来保证的。日志包括了回滚日志和重做日志。当我们通过事务对数据进行修改的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。这样做的好处是，即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执行，从而使事务具有持久性。

### **事务的控制**

当我们了解了事务的特性后，再来看下如何使用事务。我们知道 Oracle 是支持事务的，而在 MySQL 中，则需要选择适合的存储引擎才可以支持事务。如果你使用的是 MySQL，可以通过 SHOW ENGINES 命令来查看当前 MySQL 支持的存储引擎都有哪些，以及这些存储引擎是否支持事务。

事务的常用控制语句都有哪些。

1. START TRANSACTION 或者 BEGIN，作用是显式开启一个事务。
2. COMMIT：提交事务。当提交事务后，对数据库的修改是永久性的。
3. ROLLBACK 或者 ROLLBACK TO [SAVEPOINT]，意为回滚事务。意思是撤销正在进行的所有没有提交的修改，或者将事务回滚到某个保存点。
4. SAVEPOINT：在事务中创建保存点，方便后续针对保存点进行回滚。一个事务中可以存在多个保存点。
5. RELEASE SAVEPOINT：删除某个保存点。
6. SET TRANSACTION，设置事务的隔离级别。

需要说明的是，使用事务有两种方式，分别为隐式事务和显式事务。隐式事务实际上就是自动提交，Oracle 默认不自动提交，需要手写 COMMIT 命令，而 MySQL 默认自动提交，当然我们可以配置 MySQL 的参数：

```sql
mysql> set autocommit =0;  //关闭自动提交
```

```sql
mysql> set autocommit =1;  //开启自动提交
```

我们看下在 MySQL 的默认状态下，下面这个事务最后的处理结果是什么：

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB;
BEGIN;
INSERT INTO test SELECT '关羽';
COMMIT;
BEGIN;
INSERT INTO test SELECT '张飞';
INSERT INTO test SELECT '张飞';
ROLLBACK;
SELECT * FROM test;
```

运行结果

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/name%E5%85%B3%E4%BA%8E.png)



如果我们进行下面的操作又会怎样呢？

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB;
BEGIN;
INSERT INTO test SELECT '关羽';
COMMIT;
INSERT INTO test SELECT '张飞';
INSERT INTO test SELECT '张飞';
ROLLBACK;
SELECT * FROM test;
```

运行结果：你能看到这次数据是 2 行，上一次操作我把两次插入“张飞”放到一个事务里，而这次操作它们不在同一个事务里，那么对于 MySQL 来说，默认情况下这实际上就是两个事务，因为在 autocommit=1 的情况下，MySQL 会进行隐式事务，也就是自动提交，因此在进行第一次插入“张飞”后，数据表里就存在了两行数据，而第二次插入“张飞”就会报错：1062 - Duplicate entry '张飞' for key 'PRIMARY'。最后我们在执行 ROLLBACK 的时候，实际上事务已经自动提交了，就没法进行回滚了


![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/%E7%9A%84.png)


## 事务隔离

### 事务并发处理可能存在的异常都有哪些？

在了解数据库隔离级别之前，我们需要了解设定事务的隔离级别都要解决哪些可能存在的问题，也就是事务并发处理时会存在哪些异常情况。实际上，SQL-92 标准中已经对 3 种异常情况进行了定义，这些异常情况级别分别为脏读（Dirty Read）、不可重复读（Nonrepeatable Read）和幻读（Phantom Read）。

1. 脏读：读到了其他事务还没有提交的数据。
2. 不可重复读：对某数据进行读取，发现两次读取的结果不同，也就是说没有读到相同的内容。这是因为有其他事务对这个数据同时进行了修改或删除。
3. 幻读：事务 A 根据条件查询得到了 N 条数据，但此时事务 B 更改或者增加了 M 条符合事务 A 查询条件的数据，这样当事务 A 再次进行查询的时候发现会有 N+M 条数据，产生了幻读。

### 事务隔离的级别有哪些？

脏读、不可重复读和幻读这三种异常情况，是在 SQL-92 标准中定义的，同时 SQL-92 标准还定义了 4 种隔离级别来解决这些异常情况。

解决异常数量从少到多的顺序（比如读未提交可能存在 3 种异常，可串行化则不会存在这些异常）决定了隔离级别的高低，这四种隔离级别从低到高分别是：读未提交（READ UNCOMMITTED ）、读已提交（READ COMMITTED）、可重复读（REPEATABLE READ）和可串行化（SERIALIZABLE）。这些隔离级别能解决的异常情况如下表所示：


![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/%E5%95%8A.png)


读未提交，也就是允许读到未提交的数据，这种情况下查询是不会使用锁的，可能会产生脏读、不可重复读、幻读等情况。

读已提交就是只能读到已经提交的内容，可以避免脏读的产生，属于 RDBMS 中常见的默认隔离级别（比如说 Oracle 和 SQL Server），但如果想要避免不可重复读或者幻读，就需要我们在 SQL 查询的时候编写带加锁的 SQL 语句（我会在进阶篇里讲加锁）。

可重复读，保证一个事务在相同查询条件下两次查询得到的数据结果是一致的，可以避免不可重复读和脏读，但无法避免幻读。MySQL 默认的隔离级别就是可重复读。

可串行化，将事务进行串行化，也就是在一个队列中按照顺序执行，可串行化是最高级别的隔离等级，可以解决事务读取中所有可能出现的异常情况，但是它牺牲了系统的并发性。

---

## 游标

关于游标，你需要掌握以下几个方面的内容：

1. 什么是游标？我们为什么要使用游标？
2. 如何使用游标？使用游标的常用步骤都包括哪些？
3. 如何使用游标来解决一些常见的问题？

### **什么是游标？**

在数据库中，游标是个重要的概念，它提供了一种灵活的操作方式，可以**让我们从数据结果集中每次提取一条数据记录进行操作**。游标让 SQL 这种面向集合的语言有了面向过程开发的能力。可以说，游标是面向过程的编程方式，这与面向集合的编程方式有所不同。

在 SQL 中，游标是一种临时的数据库对象，可以指向存储在数据库表中的数据行指针。这里游标充当了指针的作用，我们可以通过操作游标来对数据行进行操作。

比如我们查询了 heros 数据表中最大生命值大于 8500 的英雄都有哪些：

```sql
SELECT id, name, hp_max FROM heros WHERE hp_max > 8500
```
![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/%E7%88%B1%E7%9A%84.png)



这里我们就可以通过游标来操作数据行，如图所示此时游标所在的行是“白起”的记录，我们也可以在结果集上滚动游标，指向结果集中的任意一行。

### **如何使用游标？**

游标实际上是一种控制数据集的更加灵活的处理方式。

如果我们想要使用游标，一般需要经历五个步骤。不同 DBMS 中，使用游标的语法可能略有不同。

第一步，定义游标。

```sql
DECLARE cursor_name CURSOR FOR select_statement
```

这个语法适用于 MySQL，SQL Server，DB2 和 MariaDB。如果是用 Oracle 或者 PostgreSQL，需要写成：

```sql
DECLARE cursor_name CURSOR IS select_statement
```

要使用 SELECT 语句来获取数据结果集，而此时还没有开始遍历数据，这里 **select_statement 代表的是 SELECT 语句**。

下面我用 MySQL 举例讲解游标的使用，如果你使用的是其他的 RDBMS，具体的游标语法可能略有差异。我们定义一个能够存储 heros 数据表中的最大生命值的游标，可以写为：

```sql
DECLARE cur_hero CURSOR FOR 
  SELECT hp_max FROM heros;
```

第二步，打开游标。

```sql
OPEN cursor_name
```

当我们定义好游标之后，如果想要使用游标，必须先打开游标。打开游标的时候 SELECT 语句的查询结果集就会送到游标工作区。

第三步，从游标中取得数据。

```sql
FETCH cursor_name INTO var_name ...
```

这句的作用是使用 cursor_name 这个游标来读取当前行，并且将数据保存到 var_name 这个变量中，游标指针指到下一行。如果游标读取的数据行有多个列名，则在 INTO 关键字后面赋值给多个变量名即可。

第四步，关闭游标。

```sql
CLOSE cursor_name
```

有 OPEN 就会有 CLOSE，也就是打开和关闭游标。当我们使用完游标后需要关闭掉该游标。关闭游标之后，我们就不能再检索查询结果中的数据行，如果需要检索只能再次打开游标。

最后一步，释放游标。

```sql
DEALLOCATE cursor_namec
```

有 DECLARE 就需要有 DEALLOCATE，DEALLOCATE 的作用是释放游标。我们一定要养成释放游标的习惯，否则游标会一直存在于内存中，直到进程结束后才会自动释放。当你不需要使用游标的时候，释放游标可以减少资源浪费。

上面就是 5 个常用的游标步骤。我来举一个简单的例子，假设我想用游标来扫描 heros 数据表中的数据行，然后累计最大生命值，那么该怎么做呢？

我先创建一个存储过程 calc_hp_max，然后在存储过程中定义游标 cur_hero，使用 FETCH 获取每一行的具体数值，然后赋值给变量 hp，再用变量 hp_sum 做累加求和，最后再输出 hp_sum，代码如下：

```sql
CREATE PROCEDURE `calc_hp_max`()
BEGIN
       -- 创建接收游标的变量
       DECLARE hp INT;  
       -- 创建总数变量 
       DECLARE hp_sum INT DEFAULT 0;
       -- 创建结束标志变量  
       DECLARE done INT DEFAULT false;
       -- 定义游标     
       DECLARE cur_hero CURSOR FOR SELECT hp_max FROM heros;
       
       OPEN cur_hero;
       read_loop:LOOP 
       FETCH cur_hero INTO hp;
       SET hp_sum = hp_sum + hp;
       END LOOP;
       CLOSE cur_hero;
       SELECT hp_sum;
END
```

你会发现执行call calc_hp_max()这一句的时候系统会提示 1329 错误，也就是在 LOOP 中当游标没有取到数据时会报的错误。

当游标溢出时（也就是当游标指向到最后一行数据后继续执行会报的错误），我们可以定义一个 continue 的事件，指定这个事件发生时修改变量 done 的值，以此来判断游标是否已经溢出，即：

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;
```

同时在循环中我们需要加上对 done 的判断，如果游标的循环已经结束，就需要跳出 read_loop 循环，完善的代码如下：

```sql
CREATE PROCEDURE `calc_hp_max`()
BEGIN
       -- 创建接收游标的变量
       DECLARE hp INT;  

       -- 创建总数变量 
       DECLARE hp_sum INT DEFAULT 0;
       -- 创建结束标志变量  
     DECLARE done INT DEFAULT false;
       -- 定义游标     
       DECLARE cur_hero CURSOR FOR SELECT hp_max FROM heros;
       -- 指定游标循环结束时的返回值  
     DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;  
       
       OPEN cur_hero;
       read_loop:LOOP 
       FETCH cur_hero INTO hp;
       -- 判断游标的循环是否结束  
       IF done THEN  
                     LEAVE read_loop;
       END IF; 
              
       SET hp_sum = hp_sum + hp;
       END LOOP;
       CLOSE cur_hero;
       SELECT hp_sum;
END
```

### 使用游标来解决一些常见的问题

我刚才讲了一个简单的使用案例，实际上如果想要统计 hp_sum，完全可以通过 SQL 语句来完成，比如：

```sql
SELECT SUM(hp_max) FROM heros
```

那么游标都有什么用呢？

当你需要处理一些复杂的数据行计算的时候，游标就会起到作用了。我举个例子，还是针对 heros 数据表，假设我们想要对英雄的物攻成长（对应 attack_growth）进行升级，在新版本中大范围提升英雄的物攻成长数值，但是针对不同的英雄情况，提升的幅度也不同，具体提升的方式如下。

如果这个英雄原有的物攻成长小于 5，那么将在原有基础上提升 7%-10%。如果物攻成长的提升空间（即最高物攻 attack_max- 初始物攻 attack_start）大于 200，那么在原有的基础上提升 10%；如果物攻成长的提升空间在 150 到 200 之间，则提升 8%；如果物攻成长的提升空间不足 150，则提升 7%。

如果原有英雄的物攻成长在 5—10 之间，那么将在原有基础上提升 5%。

如果原有英雄的物攻成长大于 10，则保持不变。

以上所有的更新后的物攻成长数值，都需要保留小数点后 3 位。

你能看到上面这个计算的情况相对复杂，实际工作中你可能会遇到比这个更加复杂的情况，这时你可以采用面向过程的思考方式来完成这种任务，也就是说先取出每行的数值，然后针对数值的不同情况采取不同的计算方式。

针对上面这个情况，你自己可以用游标来完成转换，具体的代码如下：

```sql
CREATE PROCEDURE `alter_attack_growth`()
BEGIN
       -- 创建接收游标的变量
       DECLARE temp_id INT;  
       DECLARE temp_growth, temp_max, temp_start, temp_diff FLOAT;  

       -- 创建结束标志变量  
       DECLARE done INT DEFAULT false;
       -- 定义游标     
       DECLARE cur_hero CURSOR FOR SELECT id, attack_growth, attack_max, attack_start FROM heros;
       -- 指定游标循环结束时的返回值  
       DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;  
       
       OPEN cur_hero;  
       FETCH cur_hero INTO temp_id, temp_growth, temp_max, temp_start;
       REPEAT
                     IF NOT done THEN
                            SET temp_diff = temp_max - temp_start;
                            IF temp_growth < 5 THEN
                                   IF temp_diff > 200 THEN
                                          SET temp_growth = temp_growth * 1.1;
                                   ELSEIF temp_diff >= 150 AND temp_diff <=200 THEN
                                          SET temp_growth = temp_growth * 1.08;
                                   ELSEIF temp_diff < 150 THEN
                                          SET temp_growth = temp_growth * 1.07;
                                   END IF;                       
                            ELSEIF temp_growth >=5 AND temp_growth <=10 THEN
                                   SET temp_growth = temp_growth * 1.05;
                            END IF;
                            UPDATE heros SET attack_growth = ROUND(temp_growth,3) WHERE id = temp_id;
                     END IF;
       FETCH cur_hero INTO temp_id, temp_growth, temp_max, temp_start;
       UNTIL done = true END REPEAT;
       
       CLOSE cur_hero;
END
```

这里我创建了 alter_attack_growth 这个存储过程，使用了 REPEAT…UNTIL…的循环方式，针对不同的情况计算了新的物攻成长 temp_growth，然后对原有的 attack_growth 进行了更新，最后调用 call alter_attack_growth(); 执行存储过程。

有一点需要注意的是，我们在对数据表进行更新前，需要备份之前的表，我们可以将备份后的表命名为 heros_copy1。更新完 heros 数据表之后，你可以看下两张表在 attack_growth 字段上的对比，我们使用 SQL 进行查询：

```sql
SELECT heros.id, heros.attack_growth, heros_copy1.attack_growth FROM heros JOIN heros_copy1 WHERE heros.id = heros_copy1.id
```
![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/%E9%98%BF%E6%96%AF%E8%BE%BE.png)


通过前后两张表的 attack_growth 对比你也能看出来，存储过程通过游标对不同的数据行进行了更新。

需要说明的是，以上代码适用于 MySQL，如果在 SQL Server 或 Oracle 中，使用方式会有些差别。