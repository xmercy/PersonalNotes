# Mybatis缓存

Mybatis有两级缓存，分别是一级缓存和二级缓存。

## 一级缓存（本地缓存）

- 一级缓存是SqlSession级别的缓存，默认是开启的。
- 一级缓存使用的是Cache接口的实现：`PerpetualCache`，底层使用的是Map
- 同一次与数据库会话期间，从数据库查询到的数据会保存到一级缓存，在这个会话期间，再次或多次获取该数据，先从一级缓存中拿。

- 一级缓存失效情景：
  - SqlSession不同
  - SqlSession相同，查询条件不同。(当前一级缓存中还没有这个数据)
  - SqlSession相同，查询条件相同，但两次查询之间执行了增删改操作(这次增删改可能对当前数据有影响)
  - SqlSession相同，查询条件相同，但手动清除了一级缓存

## 二级缓存（全局缓存）

- 二级缓存是namespace级别的缓存
- 默认情况下，mybatis二级缓存使用的也是Cache接口的实现：`PerpetualCache`
- 数据会在执行提交或关闭SqlSession之前，保存到二级缓存中。新的会话查询该数据的时候会先从二级缓存中查找。

- 使用二级缓存：

  1. mybatis配置文件中开启全局缓存

     `<setting name="cacheEnabled" value="true"/>`

  2. 需要使用二级缓存的mapper.xml中配置二级缓存开启，加上`<cache></cache>`即可。cache标签属性说明：

     ```xml
     <cache eviction="FIFO" flushInterval="60000" readOnly="false" size="1024"></cache>
     <!--  
     	eviction:缓存的回收策略：
             • LRU – 最近最少使用的：移除最长时间不被使用的对象。
             • FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
             • SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
             • WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
             • 默认的是 LRU。
     	flushInterval：缓存刷新间隔。缓存多长时间刷新一次，默认无刷新间隔。单位：毫秒
     	readOnly:默认false
     		true：只读；mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
     		false：非只读：mybatis觉得获取的数据可能会被修改。利用序列化&反序列化的技术克隆一份新的数据给用户。安全，速度慢。
     	size：缓存存放多少元素；默认1024
     	type：指定自定义缓存的全类名；自定义缓存需要自行实现Cache接口。
      -->
     ```

  3. 相关的POJO需要实现`java.io.Serializable`接口。因为默认情况下`cache`标签的`readOnly`属性值为`false`，这将导致从二级缓存中获取数据的时候需要使用反序列化技术克隆出一份相同的数据给用户。

- 使用第三方缓存作为二级缓存（ehcache）

  1. 加入mybatis整合ehcache依赖

     ```xml
     <dependency>
         <groupId>org.mybatis.caches</groupId>
         <artifactId>mybatis-ehcache</artifactId>
         <version>1.1.0</version>
     </dependency>
     ```

     mybatis-ehcache依赖于ehcache，maven会自动把ehcache引入项目

  2. 加入ehcache配置文件（ehcache包中有模板配置文件）

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
      <!-- 磁盘保存路径 -->
      <diskStore path="E:\DevData\ehcache" />
      
      <defaultCache 
        maxElementsInMemory="10000" 
        maxElementsOnDisk="10000000"
        eternal="false" 
        overflowToDisk="true" 
        timeToIdleSeconds="120"
        timeToLiveSeconds="120" 
        diskExpiryThreadIntervalSeconds="120"
        memoryStoreEvictionPolicy="LRU">
      </defaultCache>
     </ehcache>
      
     <!-- 
         属性说明：
             diskStore：指定数据在磁盘中的存储位置。
             defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
      
         以下属性是必须的：
             maxElementsInMemory - 在内存中缓存的element的最大数目 
             maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
             eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
             overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
      
         以下属性是可选的：
             timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
             timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
             diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
             diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
             diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
             memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
      -->
     ```

  3. mapper.xml中配置使用第三方缓存

     `<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>`

## 查询及缓存数据的流程

1. 先从二级缓存中查找，若有则直接返回。
2. 若二级缓存中没有，则在一级缓存中查找，若有则直接返回。
3. 若一级缓存中也没有，则从数据库查询数据。
4. 从数据库返回的数据在返回给用户之前，先放进一级缓存中。
5. 在SqlSession执行提交或者关闭的之前，会把数据保存到二级缓存中。