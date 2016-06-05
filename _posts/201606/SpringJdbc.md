---
title: SpringJdbc
date: 2016-06-05 13:49:10
tags:
- Spring
- 笔记
- java
categories:
- 笔记
---
## 基础
简单的JdbcTemplate创建：
1. 创建一个数据源
2. 生成一个JdbcTemplate实例
3. 执行sql语句

在Dao中的注入
1. 声明一个DAO
2. 注入JdbcTemplate 实例声明一个DAO

spring配置文件中配置DAO的一般步骤：
1. 定义Datasource
2. 定义JdbcTemplate
3. 声明一个抽象的<bean>，以便所有DAO复用配置JdbcTemplate属性配置
4. 配置具体的DAO

## 具体使用
使用update()修改数据，参数为类型参数为实体类的具体类型，但更好的做法是显示的指定每个占位符所对应的字段数据类型。在插入数据时，有时更喜欢使用数据库自动产生的主键值绑定到Statement或PreparedStatement。

可以使用batchTemplate()方法来一次性插入多条数据。spring在内部使用jdbc提供的批量更新API完成操作，如果底层的jdbc Driver不支持批量更新操作，Spring将采用逐条更新的方式模拟批量更新。当使用BatchPreparedStatementSetter回调接口作为参数进行批量参数的绑定工作，其会一次性的批量提交数据

查询数据：
1. 使用RowCallBackHandler处理结果集
2. 使用RowMapper<T>处理结果集

Spring宣称RoeCallBackHandler接口实现类是有状态的，而RowMapper<T>的实现类应该是无状态的。如果RowCallBackHandler实现了是有状态的，用户就不能再多个地方复用代码，只有无状态的实例才能才不同的地方复用。

jdbc查询返回一个ResultSet结果集时，jdbc并不会一次性将所有匹配的数据都加载到jvm中，而是只返回同一批次的数据，当使用ResultSet#next()游标滚动结果集超过数据范围时，jdbc再获取一批数据，避免了大量的数据开销。当处理大结果集时，如果使用RowMapper，因为是串行化处理，但是数据最后会汇聚为List<T>对象，会消耗大量内存。


查询单值数据：只适用于数据库中返回的时候仅有一个值的情况。提供了分别用于获取int、龙的单值以及其他类型的单值以Object类型返回

调用存储过程：
1. <T> T execute(String callString,CallableStatementCallback<T> action)
用户通过callString参数指定存储过程的SQL语句，第二个参数进行输入参数的绑定、输出参数注册以及返回数据处理等操作
2. <T> T execute(CallableStatementCreator csc,CallableStatementCallback<T> action)
CallableStatementCreator负责创建CallableStatement实例、绑定参数、注册输出参数，而CallableStatementCallback负责处理存储过程返回的结果

通过CallableStatementCreatorFactory创建CallableStatementCreator的顺序为：
1. 以一个SQL语句作为参数创建CallableStatementCreatorFactory工厂实例
2. 设置存储过程的入参(SqlParameter)、出参(SqlOutParameter)或者出入参(SqlInOutParameter)。注意添加的顺序必须和?占位符顺序一致
3. 通过 new CallableStatementCreator(Map<String,?> map)创建一个CallableStatementCreator实例。

Blob/Clob类型数据的操作：
1. 插入Lob类型数据
2. 以块数据方式读取Lob数据
3. 以流数据方式读取Lob数据

NamedParameterJdbcTemplate
1. 定义参数源
2. 定义命名参数
3. 使用模板方法
声明命名参数的格式是":paramName"，这是领域对象中命名的参数类型。如果数据表记录没有对应的领域对象，则直接使用MapSQLParameterSource来达到绑定参数的目的

## 以OO方式访问数据库
使用MappingSqlQuery查询数据库
在实际使用过程中，一般通定义内部类的方式使用MappingSqlQuery子类，指定查询语句并映射结果集。
步骤为：
1. 定义子类，在子类中声明SQL语句并定义行数据映射逻辑
2. 声明子类的变量并实例化该类
3. 使用MappingSqlQuery的方法执行数据查询

利用SqlUpdate更新数据
1. 扩展SqlUpdate定义新增对象的内部类
2. 定人插入记录的Sql的语句，该语句使用命名参数
3. 声明变量并实例化，然后执行调用
使用命名参数时，必须使用SQLParameter的方式定义参数

使用StoredProcedure执行存储过程

SqlFunction操作封装了一个SQL“函数”包装器，该包装器适用于查询并返回 
