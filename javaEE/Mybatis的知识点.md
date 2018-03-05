1.mybatis介绍
mybatis就是一个封装了jdbc的持久层框架，它的前身是ibatis。
Mybatis与hibernate一样都是持久层框架，但是它与hibernate不同的是，它不是一个完全的orm框架。
Mybatis只需要让程序员去关注sql本身。对于数据库的创建及关闭，statement的创建等都由mybatis进行封装。
Mybatis可以对输入的参数进行映射，可以对输出的结果进行映射。
 
2.分析原生jdbc的问题
（1、创建数据库连接时存在硬编码
---配置文件
（2、执行statement时存在硬编码
---配置文件
（3、频繁的开启和关闭数据库连接存在性能浪费
---连接池

3.mybatis开发dao方式
原始dao开发方式
即开发dao接口和实现类
问题分析：
（1、存在大量的模板代码
（2、存在硬编码
4.mapper代理开发方式
即开发mapper接口即可，mapper接口，也就是dao接口。
接口开发规范：
（1、mapper接口的类名的全限定名和mapper映射文件的namespace值一致。
（2、mapper接口的方法名称要和mapper映射文件的statement的id一致。
（3、mapper接口的方法参数类型要和mapper映射文件的statement的parameterType的值一致。
（4、mapper接口的方法返回值类型要和mapper映射文件的statement的resultType的值一致。
mapper接口

mapper映射文件

全局配置文件加载mapper

测试代码

 
5.全局配置文件SqlMapConfig.xml
Properties（属性）
加载java的配置文件的信息到mybatis配置文件中进行使用

Properties标签的加载顺序如下：
（1、先加载《property》标签声明的变量
（2、再加载properties标签引入的变量
（3、最后加载的就是statement标签中parameterType的值
Settings（全局参数设置）
配置了mybatis的全局参数，该参数会影响整个mybatis的运行行为
typeAliases（类型别名）
类型的别名，它只对po类进行别名的定义
自定义别名
 
6.mappers（映射器）
（1.<mapper resource=’’/>
使用相对于类路径的资源
如：<mapper resource="sqlmap/User.xml" />
（2.<mapper url=’’/>
使用完全限定路径
如：<mapper url="file:///D:\workspace_spingmvc\mybatis_01\config\sqlmap\User.xml" />
（3.<mapper class=’’/>
使用mapper接口的全限定名
如：<mapper class="cn.itcast.mybatis.mapper.UserMapper"/>
注意：此种方法要求mapper接口和mapper映射文件要名称相同，且放到同一个目录下；
（4.<package name=’’/>（推荐）
注册指定包下的所有映射文件
如：<package name="cn.itcast.mybatis.mapper"/>
注意：此种方法要求mapper接口和mapper映射文件要名称相同，且放到同一个目录下；
映射文件
输入映射
在mybatis中，输入映射使用parameterType来进行映射
映射简单类型

映射pojo

映射包装pojo
映射文件

pojo包装类

mapper接口

映射mapper集合
同传递POJO对象一样，map的key相当于pojo的属性
映射文件
<!-- 传递hashmap综合查询用户信息 -->
<select id="findUserByHashmap" parameterType="hashmap" resultType="user">
select * from user where id=#{id} and username like '%${username}%'
</select>
输出映射
Mybatis在进行输出结果的映射时，有两种映射方式，一种 是使用resultType标签、一种是使用resultMap标签
resultType：需要满足查询的列名和映射对象属性名称保持一致即可。
resultMap：不需要查询的列名和映射对象的属性名称一致。但是需要定义一个resultMap标签来完成列名和属性名的映射关系。
动态sql
Mybatis提供了一些动态标签，可以让程序员再编写映射文件时，更加方便灵活、提高代码的可重用性
If、where标签：在综合查询时，查询条件由客户输入，不能固定，所以映射文件中的查询条件不能写死
Sql片段：Sql片段可以提高代码的可重用性。先定义后使用
Foreach标签：可以将集合参数传入到映射文件中，然后通过foreach标签对集合参数进行遍历映射
 
7.mybatis与hibernate的各自应用场景
Mybatis技术特点：
（1、通过直接编写SQL语句，可以直接对SQL进行性能的优化；
（2、学习门槛低，学习成本低。只要有SQL基础，就可以学习mybatis，而且很容易上手；
（3、由于直接编写SQL语句，所以灵活多变，代码维护性更好。
（4、不能支持数据库无关性，即数据库发生变更，要写多套代码进行支持，移植性不好。
Hibernate技术特点：
（1、标准的orm框架，程序员不需要编写SQL语句。
（2、具有良好的数据库无关性，即数据库发生变化的话，代码无需再次编写。
（3、学习门槛高，需要对数据关系模型有良好的基础，而且在设置OR映射的时候，需要考虑好性能和对象模型的权衡。
（4、程序员不能自主的去进行SQL性能优化。
Mybatis应用场景：
需求多变的互联网项目，例如电商项目。
Hibernate应用场景：
需求明确、业务固定的项目，例如OA项目、ERP项目等。



8.MyBatis框架的架构

下面作简要概述：
（1.SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句，此文件需要在SqlMapConfig.xml中加载。
（2.通过mybatis环境等配置信息构造SqlSessionFactory(即会话工厂)。
（3.由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行。
（4.mybatis底层自定义了Executor执行器接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器。
（5.MappedStatement也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个MappedStatement对象，sql的id即是MappedStatement的id。
（6.MappedStatement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过MappedStatement在执行sql前将输入的java对象映射至sql中，输入参数映射就是JDBC编程中对preparedStatement设置参数。
（7.MappedStatement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，Executor通过MappedStatement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于JDBC编程中对结果的解析处理过程。
（8.MyBatis 缓存机制

9.与Hibernate一样，MyBatis 同样提供了一级缓存和二级缓存的支持。
（1.一级缓存: 基于PerpetualCache 的 HashMap本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该Session中的所有 Cache 就将清空。
（2.二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义第三方存储源，如 Ehcache框架等。
总结对于缓存数据更新机制，当某一个作用域(一级缓存Session/二级缓存Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear。
