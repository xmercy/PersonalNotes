# Mybatis参数处理及取参数值

### 单个参数

- mybatis不做任何处理

- 取值方式：

  ​	#{参数名/任意名}

  ```xml
  <!-- Employee getEmpById(Integer id);  -->
  <select id="getEmpById" resultType="com.mxc.bean.Employee">
      select * from employee where id=#{id}
  </select>
  ```


### 多个参数

- mybatis会将多个参数自动封装为一个map

  ​	key：param1，...，paramN，也可以是0，...，N-1(即参数索引，N是参数的个数)

  ​	value：参数值

- 取值方式：

  ​	#{上述的key}

  ```xml
  <!-- Employee getEmpByIdAndLastName(Integer id, String lastName);  -->
  <select id="getEmpByIdAndLastName" resultType="com.mxc.bean.Employee">
      select * from employee where id=#{param1} and last_name=#{param2}
  </select>
  ```

- 为多个参数指定明确的key

  - `@Param`注解可以为参数指定一个明确的key，方便在sql中取参数值

  - 取值方式：

    ​	#{@Param注解指定的key}

  ```xml
  <!-- Employee getEmpByIdAndLastName(@Param("id")Integer id, @Param("lastName")String lastName);  -->
  <select id="getEmpByIdAndLastName" resultType="com.mxc.bean.Employee">
      select * from employee where id=#{id} and last_name=#{lastName}
  </select>
  ```

  > 参数多时会封装map，为了取参数值方便，使用@Param来指定封装时使用的key

### 参数是一个POJO

- 若多个参数刚好是一个POJO中的属性值，可以直接传入一个POJO

- 取值方式：

  ​	#{属性名}

  ```xml
  <!-- Employee getEmpByIdAndLastName(Employee emp);  -->
  <select id="getEmpByIdAndLastName" resultType="com.mxc.bean.Employee">
      select * from employee where id=#{id} and last_name=#{lastName}
  </select>
  ```

### 参数是一个Map

- 若多个参数不是某一个POJO的属性，可以封装为一个Map

- 取值方式：

  ​	#{Map的key}

  ```xml
  <!-- 
  	Employee getEmpByIdAndLastName(Map<String, Object> map); 
  	map：
  		Map<String, Object> map = new HashMap<>();
  		map.put("id", 1);
  		map.put("lastName", "mxc");
  -->
  <select id="getEmpByIdAndLastName" resultType="com.mxc.bean.Employee">
      select * from employee where id=#{id} and last_name=#{lastName}
  </select>
  ```

### 小试牛刀

```java
Employee getEmp(@Param("id")Integer id,String lastName);
// 取值：id=>#{id/param1}，lastName=>#{param2}

Employee getEmp(Integer id,@Param("e")Employee emp);
// 取值：id=>#{param1}，lastName => #{param2.lastName/e.lastName}
```

### `#{}`与`${}`的异同

- 同：

  ​	可以获取map中的值或者pojo对象属性的值

- 异：
  - `#{}`：是以预编译的形式，将参数设置到sql语句中，可以防止sql注入
  - `${}`：取出的值直接拼装在sql语句中，会有安全问题

- 使用场景

  大多情况下，取参数的值使用`#{}`。在原生JDBC不支持占位符的地方可以使用`${}`。如按年份分表查询：`select * from ${year}_salary;`










