# MyBatis

## 1 MyBatis的一级缓存和二级缓存

### 1.1 MyBatis的一级缓存

#### 1.1.1 什么是一级缓存

    `MyBatis`是的一级缓存默认是开启的，是`sqlSession`级别的，当这个`sqlSession`被关闭之后缓存就没有了。之所以需要一级缓存，是考虑到在一个会话中可能会反复执行一条相同的sql，这条sql返回的结果也是一直保持不变的，如果每次都查询数据库的话，会导致查询数据库的次数过多，导致数据库压力过大，通过引入缓存可以减轻数据库的压力。在查询一个sql的时候，`MyBatis`会先查询缓存中有没有，如果有就直接返回结果，如果没有再查询数据库。在一个查询中可以减轻查询数据库的压力。

#### 1.1.2 什么情况下能命中一级缓存

       `MyBatis`认为，对于两次查询，如果以下的条件都完全一样，那么就认为它们是完全相同的两次查询：

- 传入的`statementId`，`statementId`就是`mapper`接口中的

- 查询事要求的结果集中的结果范围（结果的范围通过rowBounds.offset和rowBounds.limit表示）

- 这次查询所产生的最终要传递给JDBC java.sql.Preparedstatement的SQL语句字符串（`boundSql.getSql()`）

- 传递给java.sql.Statement要设置的参数值

    现在解释一下上面的四个条件：

- 传入的`statementId`对于`MyBatis`而言，你要使用它，必须要一个`statementId`，它代表这你将执行什么样的`Sql`；
  
   `MyBatis`自身提供的分页的功能是通过`RowBounds`来实现的，它通过`rowBounds.offset`和`rowBounds.limit`来过滤查询出来的结果集，**这种分页功能是基于查询结果的再过滤,而不是进行数据库的物理分页**

- 由于`MyBatis`底层还是依赖于`JDBC`来实现的，那么对于两次完全一摸一样的查询，`MyBatis`要保证对于底层的JDBC而言，也是完全一致的查询才行。而对于`JDBC`而言，两次查询，只要传入给`JDBC`的SQL语句是完全一致的，传入的参数也是一致的，那么就认为两次查询是完全一致的。

- 上述的第三个条件正是要保证传递给`JDBC`的`SQL`语句完全一致；第四条则是保证传递给JDBC的参数也是完全一致；即3、4两条MyBatis最本质的要求就是调用JDBC的时候，传入的SQL语句要完全相同，传递给JDBC的参数值也要完全相同。

    综合上面来看，`CacheKey`由以下的条件决定：`statementId`+`rowBounds`+`传递给JDBC的SQL`+`传递给JDBC的参数值`。

#### 1.1.3 MyBatis一级缓存什么时候失效/MyBatis的一级缓存性能如何

    **MyBatis**对会话级别的一级缓存设计的比较简单，就简单的使用了`HashMap`来维护，并没有对`HashMap`的容量和大小进行限制。

> **上述可能存在的问题？**
> 
> 如果一致使用某一个`SqlSession`对象查询数据，这样会不会导致`HashMap`太大，而导致OOM呢？

    上述问题不是没有道理，但是一级缓存有着严格的限制，当发生下列行为的时候就会清空缓存：

- 一般而言`SqlSession`的生存时间很短。一般情况下使用一个`SqlSession`对象执行的操作不会太多，执行完就会消亡

- 对于某一个`SqlSession`对象而言，只要执行`update`操作（update、insert、delete），都会将这个`SqlSession`对象中对应的一级缓存清空掉，所以一般情况下不会出现缓存过大，影响JVM的情况

- 可以手动的释放`SqlSessiobn`对象中的缓存

- 一级缓存是一个粗力度的缓存，没有更新缓存和缓存过期的概念






