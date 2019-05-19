# Mybatis源码解析

## 查询测试Demo

```java
/**
 * 获取SqlSessionFactory对象
 *
 * @return
 * @throws IOException
 */
private SqlSessionFactory getSqlSessionFactory() throws IOException {
    return new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
}
@Test
public void testQuery() throws IOException {
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
        EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
        Employee empById = employeeMapper.getEmpById(2);
        System.out.println(empById);
        System.out.println(empById.getDept());
        sqlSession.commit();
    } catch (Exception e) {
    	e.printStackTrace();
    } finally {
    	sqlSession.close();
    }
}
```



## 获取SqlSessionFactory

1. 创建`XMLConfigBuilder`实例，解析mybatis全局配置文件及所有SQL映射文件

   调用构造器创建`XMLConfigBuilder`实例时，在构造器中调用`Configuration`的无参构造器创建了一个`Configuration`对象作为自己的属性

2. 解析mybatis全局配置文件

   解析settings、typeAliases、mappers、... 标签，并将信息封装到`XMLConfigBuilder`实例的`Configuration`对象属性中。

3. 解析SQL映射文件

   即第2步中的解析mappers标签的核心流程

   1. 创建`XMLMapperBuilde`r实例，解析SQL映射文件

      从mappers标签中解析出mapper映射文件的路径，创建流并创建`XMLMapperBuilder`实例

   2. 解析cache、cache-ref、select、update、delete、insert、... 标签

      针对select、update、delete、insert标签，解析标签中的每一个属性及标签体中的sql（包括解析动态sql），封装为一个`MappedStatement`对象，添加到`configuration`对象的`mappedStatements`（Map）中。

4. 创建`SqlSessionFactory`实现类对象并返回

   利用`XMLConfigBuilder`实例、`XMLMapperBuilder`实例解析完mybatis全局配置文件以及SQL映射文件之后，返回`XMLConfigBuilder`实例中的`Configuration`对象，利用返回的`Configuration`对象，作为`DefaultSqlSessionFactory`的构造器参数，创建`DefaultSqlSessionFactory`对象返回。

## 获取SqlSession

1. 调用`SqlSessionFactory`实例的`openSession`方法

   1. 通过`SqlSessionFactory`实例中的`Configuration`对象，获取当中的DataSource信息开启事务，创建并返回一个`Transaction`对象。

      这里开启事务是指逻辑上的开启事务，`Transaction`对象中的并没有数据库连接，仅仅是设置了`Transaction`对象的`autoCommit`属性值为false。

   2. 利用`Configuration`对象创建`Executor`

      根据`ExecutorType`创建对应的`Executor`，该`Executor`中初始化了一个本地缓存`PerpetualCache`，并封装了上一步中创建的事务对象。若还配置了二级缓存，则会将用`CachingExecutor`包装刚创建的`Executor`。

   3. 调用拦截器链的`pluginAll`方法包装`Executor`

      `executor = (Executor) interceptorChain.pluginAll(executor);`

2. 创建`DefaultSqlSession`对象并返回

   `DefaultSqlSession`对象封装了`Configuration`对象以及步骤1中创建的`Executor`。

## 获取Mapper

1. 调用`SqlSession`对象的`getMapper`方法，传入对应Mapper的Class对象。

2. 调用`Configuration`对象的`getMapper`方法。

3. 调用`MapperRegistry`对象的`getMapper`方法，利用动态代理创建了一个Mapper代理实例并返回。

   从`MapperRegistry`中获取了一个Mapper代理工厂`MapperProxyFactory`，通过代理工厂，利用动态代理技术创建了一个Mapper代理实例并返回。

```java
// 1 ---- TestCase
EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
// 2 ---- DefaultSqlSession
public <T> T getMapper(Class<T> type) {
    return configuration.<T>getMapper(type, this);
}
// 3 ---- Configuration
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}
// 4 ---- MapperRegistry
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    return mapperProxyFactory.newInstance(sqlSession);
}
// 5 ---- MapperRegistry
protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
```

## 执行查询

1. 把`Method`对象包装成`MapperMethod`对象

   上一步获取Mapper的时候，得到的是一个mapper代理对象，在调用代理对象方法的时候，会先把Method对象包装成`MapperMethod`对象，然后调用其execute方法。

2. 执行查询

   1. 判断要操作的类型，调用`SqlSession`对象的查询方法

   2. 获取当前操作对应的Sql信息

      这里先获取与当前操作对应的`MappedStatement`对象，该对象在获取`SqlSessionFactory`的时候，就已经通过解析对应的Mapper映射文件创建了出来，一个`MappedStatement`对象封装一个select|update|delete|insert语句信息。然后把查询参数与`MappedStatement`对象中Sql信息绑定到一起并返回`BoundSql`对象。

   3. 生成`CacheKey`对象

   4. 若配置了二级缓存，则拿`CacheKey`对象去二级缓存中查询，查询到则直接返回

   5. 若在二级缓存中没查到，再拿`CacheKey`对象从一级缓存中查找，查询到则直接返回

   6. 若在一级缓存中也没查到，则从数据库查询。查询到之后，先把结果放进一级缓存，然后在放进二级缓存，键都是`CacheKey`对象，最后返回。

