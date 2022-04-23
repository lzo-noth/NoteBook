# SQL必知必会语法基础篇1

## SQL语言

我们可以把 SQL 语言按照功能划分成以下的 4 个部分：

- DDL，英文叫做 Data Definition Language，也就是数据定义语言，它用来定义我们的数据库对象，包括数据库、数据表和列。通过使用 DDL，我们可以创建，删除和修改数据库和表结构。
- DML，英文叫做 Data Manipulation Language，数据操作语言，我们用它操作和数据库相关的记录，比如增加、删除、修改数据表中的记录。
- DCL，英文叫做 Data Control Language，数据控制语言，我们用它来定义访问权限和安全级别。DQL，英文叫做 Data Query Language，数据查询语言，我们用它查询想要的记录，它是 SQL 语言的重中之重。在实际的业务中，我们绝大多数情况下都是在和查询打交道，因此学会编写正确且高效的查询语句，是学习的重点。

## DBMS是什么

说到 DBMS，有一些概念你需要了解。DBMS 的英文全称是 DataBase Management System，数据库管理系统，实际上它可以对多个数据库进行管理，所以你可以理解为 DBMS = 多个数据库（DB） + 管理程序。

DB 的英文是 DataBase，也就是数据库。数据库是存储数据的集合，你可以把它理解为多个数据表。DBS 的英文是 DataBase System，数据库系统。它是更大的概念，包括了数据库、数据库管理系统以及数据库管理人员 DBA。这里需要注意的是，虽然我们有时候把 Oracle、MySQL 等称之为数据库，但确切讲，它们应该是数据库管理系统，即 DBMS。

## SQL是如何执行的

### oracle

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/oracle.png)


1. 语法检查：检查 SQL 拼写是否正确，如果不正确，Oracle 会报语法错误。
2. 语义检查：检查 SQL 中的访问对象是否存在。比如我们在写 SELECT 语句的时候，列名写错了，系统就会提示错误。语法检查和语义检查的作用是保证 SQL 语句没有错误。
3. 权限检查：看用户是否具备访问该数据的权限。
4. **共享池检查**：共享池（Shared Pool）是一块内存池，最主要的作用是缓存 SQL 语句和该语句的执行计划。Oracle 通过检查共享池是否存在 SQL 语句的执行计划，来判断进行软解析，还是硬解析。那软解析和硬解析又该怎么理解呢？在共享池中，Oracle 首先对 SQL 语句进行 Hash 运算，然后根据 Hash 值在库缓存（Library Cache）中查找，如果存在 SQL 语句的执行计划，就直接拿来执行，直接进入“执行器”的环节，这就是软解析。如果没有找到 SQL 语句和执行计划，Oracle 就需要创建解析树进行解析，生成执行计划，进入“优化器”这个步骤，这就是硬解析。优化器：优化器中就是要进行硬解析，也就是决定怎么做，比如创建解析树，生成执行计划。
5. 执行器：当有了解析树和执行计划之后，就知道了 SQL 该怎么被执行，这样就可以在执行器中执行语句了。

如何避免硬解析，尽量使用软解析呢？在 Oracle 中，绑定变量是它的一大特色。绑定变量就是在 SQL 语句中使用变量，通过不同的变量取值来改变 SQL 的执行结果。这样做的好处是能提升软解析的可能性，不足之处在于可能会导致生成的执行计划不够优化，因此是否需要绑定变量还需要视情况而定。

### MYSQL

首先 MySQL 是典型的 C/S 架构，即 Client/Server 架构，服务器端程序使用的 mysqld。整体的 MySQL 流程如下图所示：

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/mySQL.png)


你能看到 MySQL 由三层组成：

连接层：客户端和服务器端建立连接，客户端发送 SQL 至服务器端；

SQL 层：对 SQL 语句进行查询处理；

存储引擎层：与数据库文件打交道，负责数据的存储和读取。

其中 SQL 层与数据库文件的存储方式无关，我们来看下 SQL 层的结构：

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%202.png)


1. 查询缓存：Server 如果在查询缓存中发现了这条 SQL 语句，就会直接将结果返回给客户端；如果没有，就进入到解析器阶段。需要说明的是，因为查询缓存往往效率不高，所以在 MySQL8.0 之后就抛弃了这个功能。
2. 解析器：在解析器中对 SQL 语句进行语法分析、语义分析。
3. 优化器：在优化器中会确定 SQL 语句的执行路径，比如是根据全表检索，还是根据索引来检索等。
4. 执行器：在执行之前需要判断该用户是否具备权限，如果具备权限就执行 SQL 查询并返回结果。在 MySQL8.0 以下的版本，如果设置了查询缓存，这时会将查询结果进行缓存。

### 如何在 MySQL 中对一条 SQL 语句的执行时间进行分析。

首先我们需要看下 profiling 是否开启，开启它可以让 MySQL 收集在 SQL 执行时所使用的资源情况，命令如下：

```sql
mysql> select @@profiling;
```

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%203.png)


profiling=0 代表关闭，我们需要把 profiling 打开，即设置为 1：

```sql
mysql> set profiling=1;
```

然后我们执行一个 SQL 查询（你可以执行任何一个 SQL 查询）：

```sql
mysql> select * from wucai.heros;
```

查看当前会话所产生的所有 profiles：

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%204.png)


你会发现我们刚才执行了两次查询，Query ID 分别为 1 和 2。如果我们想要获取上一次查询的执行时间，可以使用：

```sql
mysql> show profile;
```

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%205.png)


当然你也可以查询指定的 Query ID，比如：

```sql
mysql> show profile for query 2;
```

## 检索数据

在这篇文章中，你需要重点掌握以下几方面的内容：

1. [SELECT 查询的基础语法；](https://www.notion.so/SQL-1-66168d65d4624e9ebf21b0da70ce31e7)
2. [如何排序检索数据；](https://www.notion.so/SQL-1-66168d65d4624e9ebf21b0da70ce31e7)
3. [什么情况下用SELECT*，如何提升 SELECT 查询效率？](https://www.notion.so/SQL-1-66168d65d4624e9ebf21b0da70ce31e7)

---

### SELECT 查询的基础语法

**查询列**

如果我们想要对数据表中的某一列进行检索，在 SELECT 后面加上这个列的字段名即可。比如我们想要检索数据表中都有哪些英雄。

```sql
SQL：SELECT name FROM heros
```

我们也可以对多个列进行检索，在列名之间用逗号 (,) 分割即可。

**取别名**

我们在使用 SELECT 查询的时候，还有一些技巧可以使用，比如你可以给列名起别名。我们在进行检索的时候，可以给英雄名、最大生命、最大法力、最大物攻和最大物防等取别名：

```sql
SQL：SELECT name AS n, hp_max AS hm, mp_max AS mm, attack_max AS am, defense_max AS dm FROM heros
```

**查询常数**

SELECT 查询还可以对常数进行查询。对的，就是在 SELECT 查询结果中增加一列固定的常数列。这列的取值是我们指定的，而不是从数据表中动态取出的。你可能会问为什么我们还要对常数进行查询呢？SQL 中的 SELECT 语法的确提供了这个功能，一般来说我们只从一个表中查询数据，通常不需要增加一个固定的常数列，但如果我们想整合不同的数据源，用常数列作为这个表的标记，就需要查询常数。

比如说，我们想对 heros 数据表中的英雄名进行查询，同时增加一列字段platform，这个字段固定值为“王者荣耀”，可以这样写：

```sql
SQL：SELECT '王者荣耀' as platform, name FROM heros
```

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%206.png)


**去除重复行**

使用DISTINCT关键字

```sql
SQL：SELECT DISTINCT attack_range FROM heros
```

### 如何排序检索数据

使用**ORDER BY**子句

注意：

1. 排序的列名：ORDER BY 后面可以有一个或多个列名，如果是多个列名进行排序，会按照后面第一个列先进行排序，当第一列的值相同的时候，再按照第二列进行排序，以此类推。
2. 排序的顺序：ORDER BY 后面可以注明排序规则，ASC 代表递增排序，DESC 代表递减排序。如果没有注明排序规则，默认情况下是按照 ASC 递增排序。我们很容易理解 ORDER BY 对数值类型字段的排序规则，但如果排序字段类型为文本数据，就需要参考数据库的设置方式了，这样才能判断 A 是在 B 之前，还是在 B 之后。比如使用 MySQL 在创建字段的时候设置为 BINARY 属性，就代表区分大小写。
3. 非选择列排序：ORDER BY 可以使用非选择列进行排序，所以即使在 SELECT 后面没有这个列名，你同样可以放到 ORDER BY 后面进行排序。
4. ORDER BY 的位置：ORDER BY 通常位于 SELECT 语句的最后一条子句，否则会报错。

### 约束返回结果的数量

使用**LIMIT**关键字

**MYSQL**

使用limit关键字时有两个参数，如：limit arg1,arg2；

其中：arg1表示从哪一行开始，初始行为0；

arg2表示显示多少行，

如：limit 0,2;　表示从第0行开始显示两条数据。

limit 1,2;　表示从第1行开始显示两条数据。

### SELECT的执行顺序

关键字的顺序，不能颠倒

```sql
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...
```

SELECT 语句的执行顺序（在 MySQL 和 Oracle 中，SELECT 执行顺序基本相同）：

```sql
FROM > WHERE > GROUP BY > HAVING > SELECT的字段 > DISTINCT > ORDER BY > LIMIT
```

### 什么情况下用SELECT*,如何提升SELECT 查询效率

当我们初学 SELECT 语法的时候,经常会使用SELECT*,因为使用方便。 实际上这样也增加了数据库的负担。 所以如***果我们不需要把所有列都检索出来,还是先指定出所需的列名,因为写清列名,可以减少数据表查询的网络传输量,而且考虑到在实际的工作中,我们往往不需要全部的列名,因此你需要养成良好的习惯,写出所需的列名***。
如果我们只是练习,或者对数据表进行探索,那么是可以使用SELECT *的。 它的查询效率和把所有列名都写出来再进行查询的效率相差并不大。 这样可以方便你对数据表有个整体的认知。但是在生产环境下,不推荐你直接使用SELECT *进行查询。

## 数据过滤

1. 学会使用 WHERE 子句，如何使用比较运算符对字段的数值进行比较筛选；
2. 如何使用逻辑运算符，进行多条件的过滤；
3. 学会使用通配符对数据条件进行复杂过滤。

### 比较运算符

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%207.png)


注：不同DBMS支持运算符不一样，如MYSQL不支持!<,!>等。

### 逻辑运算符

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%208.png)


### 通配符

```sql
SQL：SELECT name FROM heros WHERE name LIKE '%太%'
```

## SQL函数

常用的 SQL 函数有哪些 SQL 提供了一些常用的内置函数,当然你也可以自己定义 SQL 函数 SQL 的内置函数对于不同的数据库软件来说具有一定的通用性,我们可以把内置函数分成四类: 

1. 算术函数 
    
    ![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%209.png)

    
2. 字符串函数 
    
    ![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%2010.png)

    
3. 日期函数 
    
    ![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%2011.png)

    
4. 转换函数
    
    ![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%2012.png)

    

例子:

```sql
SQL：SELECT name, ROUND(attack_growth,1) FROM heros
```

代码中,ROUND(attack growth,1)中的attack growth代表想要处理的数据, 1 代表四舍五入的位数,也就是我们这里需要精确到的位数

假如想要提取英雄上线日期(对应字段 birthdate)的年份,只显示有上线日期的英雄即可(有些英雄没有上线日期的数据,不需要显示),这里我们需要使用 EXTRACT 函数,提取某一个时间元素 所以我们需要筛选上线日期不为空的英雄,即WHERE birthdate is not null,然后再显示他们的名字和上线日期的年份,即

```sql
SQL： SELECT name, EXTRACT(YEAR FROM birthdate) AS birthdate FROM heros WHERE birthdate is NOT NULL
```

[MYSQL所有函数](https://www.runoob.com/mysql/mysql-functions.html)

尽管 SQL 函数使用起来会很方便,但我们使用的时候还是要谨慎,因为你使用的函数很可能在运行环境中无法工作,这是为什么呢? 如果你学习过编程语言,就会知道语言是有不同版本的,比如 Python 会有 2.7 版本和 3.x 版本,不过它们之间的函数差异不大,也就在 10% 左右 但我们在使用 SQL 语言的时候,不是直接和这门语言打交道,而是通过它使用不同的数据库软件,即 DBMS DBMS 之间的差异性很大,远大于同一个语言不同版本之间的差异 实际上,只有很少的函数是被 DBMS 同时支持的 比如,大多数 DBMS 使用(||)或者(+)来做拼接符,而在 MySQL 中的字符串拼接函数为Concat() 大部分 DBMS 会有自己特定的函数,这就意味着采用 SQL 函数的代码可移植性是很差的,因此在使用函数的时候需要特别注意

<aside>
⛔ 大小写规范
在 SQL 中,你还是要确定大小写的规范,因为在 Linux 和 Windows 环境下,你可能会遇到不同的大小写问题 比如 MySQL 在 Linux 的环境下,数据库名 表名 变量名是严格区分大小写的,而字段名是忽略大小写的 而 MySQL 在 Windows 的环境下全部不区分大小写 这就意味着如果你的变量名命名规范没有统一,就可能产生错误 这里有一个有关命名规范的建议: 
**关键字和函数名称全部大写;** 
**数据库名 表名 字段名称全部小写;** 
**SQL 语句必须以分号结尾**
 虽然关键字和函数名称在 SQL 中不区分大小写,也就是如果小写的话同样可以执行,但是数据库名 表名和字段名在 Linux MySQL 环境下是区分大小写的,因此建议你统一这些字段的命名规则,比如全部采用小写的方式 同时将关键词和函数名称全部大写,以便于区分数据库名 表名 字段名

</aside>

## SQL的聚集函数

**聚集函数都有哪些,**能否在一条 SELECT 语句中使用多个聚集函数;   

![](https://raw.githubusercontent.com/lzo-noth/note_pics/main/Untitled%2013.png)


**如何对数据进行分组,并进行聚集统计;** 

我们在做统计的时候,可能需要先对数据按照不同的数值进行分组,然后对这些分好的组进行聚集统计 对数据进行分组,需要使用 GROUP BY 子句 比如我们想按照英雄的主要定位进行分组,并统计每组的英雄数量 

```sql
SQL: SELECT COUNT(*), role main FROM heros GROUP BY role main
```

**如何使用 HAVING 过滤分组,HAVING 和 WHERE 的区别是什么**

当我们创建出很多分组的时候,有时候就需要对分组进行过滤 你可能首先会想到 WHERE 子句,实际上过滤分组我们使用的是 HAVING HAVING 的作用和 WHERE 一样,都是起到过滤的作用,只不过 WHERE 是用于数据行,而 HAVING 则作用于分组 比如我们想要按照英雄的主要定位 次要定位进行分组,并且筛选分组中英雄数量大于 5 的组,最后按照分组中的英雄数量从高到低进行排序

```sql
SQL: SELECT COUNT(*) as num, role_main, role_assist FROM heros GROUP BY role_main, role_assist HAVING num > 5 ORDER BY num DESC
```