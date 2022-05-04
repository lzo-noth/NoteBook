# 总结


## 1. 标签


|功能|标签名|
|--|--|
|定义SQL语句| insert|
||select|
||update|
||delete|
|配置java对象属性与查询结果集中列名对应关系|resultMap|
|控制动态SQL拼接|if|
||forEach|
||choose|
|格式化输出|set|
||where|
||trim|
|配置关联关系|collection|
||association|
|定义常量|sql|
||bind|
### (1) trim

mybatis的trim标签一般用于去除sql语句中多余的and关键字，逗号，或者给sql语句前拼接 “where“、“set“以及“values(“ 等前缀，或者添加“)“等后缀，可用于选择性插入、更新、删除或者条件查询等操作。

以下是trim标签中涉及到的属性

|属性|描述|
|--|--|
|prefix|给sql语句拼接的前缀|
|suffix|给sql语句拼接的后缀|
|prefixOverrides|去除sql语句前面的关键字或者字符，该关键字或者字符由prefixOverrides属性指定，假设该属性指定为"AND"，当sql语句的开头为"AND"，trim标签将会去除该"AND"|
|suffixOverrides|去除sql语句后面的关键字或者字符，该关键字或者字符由suffixOverrides属性指定下面使用几个例子来说明trim标签的使用|

### (2) set

在 Mybatis 中，**update 语句**可以使用 set 标签动态更新列。set 标签可以为 SQL 语句动态的添加 set 关键字，**剔除追加到条件末尾多余的逗号**。

### (3) choose

标签是按顺序判断其内部标签中的test条件出否成立，如果有一个成立，则  结束。当  中所有 的条件都不满则时，则执行  中的sql。类似于Java 的 switch 语句， 为 switch， 为 case， 则为 default。

### (4) sql

sql标签主要是为了避免在项目开发的过程中重复编写大量相同的sql语句

定义：

```xml
<sql id="all_columns">         
country.Code as country_code,         
country.Name AS country_name </sql>
```

引用：

```xml
SELECT <include refid="all_columns" />
```

### (5) foreach

使用 foreach 来实现 SQL 条件的迭代。

```xml
<foreach item="item" index="index" collection="list|array|map key" open="(" separator="," close=")">     
参数值 
</foreach>
```

### (5) collection

在 MyBatis 中，通过  元素的子元素  处理一对多级联关系，collection 可以将关联查询的多条记录映射到一个 **list** 集合属性中。

```xml
<!-- 一对多 根据id查询用户及其关联的订单信息：级联查询的第一种方法（分步查询） -->
<resultMap type="net.biancheng.po.User" id="userAndOrder1">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="pwd" column="pwd" />
    <!-- 一对多级联查询，ofType表示集合中的元素类型，将id传递给selectOrderById -->
    <collection property="orderList"
        ofType="net.biancheng.po.Order" column="id"
        select="net.biancheng.mapper.OrderMapper.selectOrderById" />
</resultMap>
<select id="selectUserOrderById1" parameterType="Integer"
    resultMap="userAndOrder1">
    select * from user where id=#{id}
</select>
```

实际应用中，由于多对多的关系比较复杂，会增加理解和关联的复杂度，所以应用较少。MyBatis 没有实现多对多级联，推荐通过两个一对多级联替换多对多级联，以降低关系的复杂度，简化程序。

例如，一个订单可以有多种商品，一种商品可以对应多个订单，订单与商品就是多对多的级联关系。可以使用一个中间表（订单记录表）将多对多级联转换成两个一对多的关系。

```xml
<mapper namespace="net.biancheng.mapper.OrderMapper">
    <resultMap type="net.biancheng.po.Order" id="orderMap">
        <id property="oid" column="oid" />
        <result property="ordernum" column="ordernum" />
        <collection property="products"
            ofType="net.biancheng.po.Product">
            <id property="pid" column="pid" />
            <result property="name" column="name" />
            <result property="price" column="price" />
        </collection>
    </resultMap>
    <select id="selectAllOrdersAndProducts" parameterType="Integer"
        resultMap="orderMap">
        SELECT o.oid,o.`ordernum`,p.`pid`,p.`name`,p.`price` FROM
        `order` o
        INNER JOIN orders_detail od ON o.oid=od.`orderId`
        INNER JOIN
        product p
        ON p.pid = od.`productId`
    </select>
</mapper>
```

### (6) association

一对一级联关系在现实生活中是十分常见的，例如一个大学生只有一个学号，一个学号只属于一个学生。同样，人与身份证也是一对一的级联关系。

在 MyBatis 中，通过  元素的子元素  处理一对一级联关系。

```xml
<mapper namespace="net.biancheng.mapper.StudentMapper">
    <!-- 一对一根据id查询学生信息：级联查询的第一种方法（嵌套查询，执行两个SQL语句） -->
    <resultMap type="net.biancheng.po.Student" id="cardAndStu1">
        <id property="id" column="id" />
        <result property="name" column="name" />
        <result property="sex" column="sex" />
        <!-- 一对一级联查询 -->
        <association property="studentCard" column="cardId"
            javaType="net.biancheng.po.StudentCard"
            select="net.biancheng.mapper.StudentCardMapper.selectStuCardById" />
    </resultMap>
    <select id="selectStuById1" parameterType="Integer"
        resultMap="cardAndStu1">
        select * from student where id=#{id}
    </select>
</mapper>
```

### (7)bind

每个数据库的拼接函数或连接符号都不同，例如 MySQL 的 concat 函数、Oracle 的连接符号“||”等。这样 SQL 映射文件就需要根据不同的数据库提供不同的实现，显然比较麻烦，且不利于代码的移植。幸运的是，MyBatis 提供了 bind 标签来解决这一问题。

## 2.分页

MyBatis 的分页功能是**基于内存**的分页，即先查询出所有记录，再按起始位置和页面容量取出结果。

## 3.缓存

MyBatis 提供了一级缓存和二级缓存的支持。默认情况下，MyBatis 只开启一级缓存。

一级缓存是基于 PerpetualCache（MyBatis自带）的 HashMap 本地缓存，作用范围为 session 域内。当 session flush（刷新）或者 close（关闭）之后，该 session 中所有的 cache（缓存）就会被清空。

在参数和 SQL 完全一样的情况下，我们使用同一个 SqlSession 对象调用同一个 mapper 的方法，往往只执行一次 SQL。因为使用 SqlSession 第一次查询后，MyBatis 会将其放在缓存中，再次查询时，如果没有刷新，并且缓存没有超时的情况下，SqlSession 会取出当前缓存的数据，而不会再次发送 SQL 到数据库。

由于 SqlSession 是相互隔离的，所以如果你使用不同的 SqlSession 对象，即使调用相同的 Mapper、参数和方法，MyBatis 还是会再次发送 SQL 到数据库执行，返回结果。

二级缓存是全局缓存，作用域超出 session 范围之外，可以被所有 SqlSession 共享。

一级缓存缓存的是 SQL 语句，二级缓存缓存的是结果对象。

对于 MyBatis 缓存仅作了解即可，因为面对一定规模的数据量，内置的 Cache 方式就派不上用场了，并且对查询结果集做缓存并不是 MyBatis 所擅长的，它专心做的应该是 SQL 映射。对于缓存，采用 OSCache、Memcached 等专门的缓存服务器来做更为合理。

## 4. 逆向工程

[MyBatis Generator Core – Introduction to MyBatis Generator](https://mybatis.org/generator/)

## 5. 注解

为了简化 XML 的配置，MyBatis 提供了注解。

[MyBatis注解](http://c.biancheng.net/mybatis/annotation.html)