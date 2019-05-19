# Mybatis的resultType与resultMap

### resultType

指定一个JavaBean的全类名，mybatis会把查询到的结果自动封装为指定的JavaBean

```xml
<!-- Employee getEmpById(Integer id); -->
<select id="getEmpById" resultType="com.mxc.entity.Employee">
	select id, last_name, gender, email from emp where id = #{id}
</select>
```



### resultMap

`resultMap`用于自定义JavaBean的封装规则

```xml
<!-- 
 type：指定自定义规则最终封装的目标Java类型
 id:指定id，便于被其他select中的resultMap使用
   -->
<resultMap type="com.mxc.bean.Employee" id="rmDemo">
    <!-- 指定主键列的封装规则
      id：定义主键，底层会有优化；
      column：指定表的哪一列
      property：指定对应的JavaBean的属性
    -->
    <id column="id" property="id"/>
    <!-- 定义普通列封装规则 -->
    <result column="last_name" property="lastName"/>
    <result column="email" property="email"/>
    <result column="gender" property="gender"/>
</resultMap>

<!-- resultMap:自定义结果集映射规则；  -->
<!-- public Employee getEmpById(Integer id); -->
<select id="getEmpById"  resultMap="rmDemo">
    select * from employee where id=#{id}
</select>
```

### 两个典型的resultMap使用场景

#### JavaBean

- `Employee.java`

  ```java
  package com.mxc.entity;
  
  public class Employee {
  
      private Integer id;
      private String lastName;
      private String gender;
      private String email;
      private Department dept;
  
      public Employee() {
      }
  
      public Employee(Integer id, String lastName, String gender, String email) {
          this.id = id;
          this.lastName = lastName;
          this.gender = gender;
          this.email = email;
      }
  
      // 省略getter和setter...
  
      @Override
      public String toString() {
          return "Employee{" +
                  "id=" + id +
                  ", lastName='" + lastName + '\'' +
                  ", gender='" + gender + '\'' +
                  ", email='" + email + '\'' +
                  '}';
      }
  }
  ```

- `Department.java`

  ```java
  package com.mxc.entity;
  
  import java.util.List;
  
  public class Department {
      private Integer id;
      private String deptName;
      private List<Employee> emps;
  
      public Department() {
      }
  
      public Department(Integer id, String deptName) {
          this.id = id;
          this.deptName = deptName;
      }
  
      // 省略getter和setter...
  
      @Override
      public String toString() {
          return "Department{" +
                  "id=" + id +
                  ", deptName='" + deptName + '\'' +
                  '}';
      }
  }
  ```

#### 查询员工同时查询员工对应的部门

##### 代码

- `EmployeeMapper.java`

  ```java
  Employee getEmpByIdAssociate(Integer id);
  ```

- `EmployeeMapper.xml`

  ```xml
  <resultMap id="empMap" type="com.mxc.entity.Employee">
  	<id column="id" property="id"></id>
  	<result column="last_name" property="lastName"></result>
  	<result column="gender" property="gender"></result>
  	<result column="email" property="email"></result>
  	<association property="dept" column="d_id"
  		select="com.mxc.dao.DeptMapper.getDeptById"></association>
  </resultMap>
  <select id="getEmpByIdAssociate" resultMap="empMap">
  	select * from emp where id = #{id}
  </select>
  ```

- `DeptMapper.java`

  ```java
  Department getDeptById(Integer id);
  ```

- `DeptMapper.xml`

  ```xml
  <select id="getDeptById" resultType="com.mxc.entity.Department">
  	select id, dept_name from dept where id = #{id}
  </select>
  ```

##### 执行及结果

```java
/**
 * 获取SqlSessionFactory对象
 * @return
 * @throws IOException
 */
private SqlSessionFactory getSqlSessionFactory() throws IOException {
    return new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
}

@Test
public void testAssociate() throws IOException {
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    SqlSession sqlSession = sqlSessionFactory.openSession();

    try {
        EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
        Employee empByIdAssociate = employeeMapper.getEmpByIdAssociate(3);
        System.out.println(empByIdAssociate.getLastName());
        System.out.println(empByIdAssociate.getDept().getDeptName());
        sqlSession.commit();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        sqlSession.close();
    }
}
```

控制台打印：

```
13:47:57.277 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - ==>  Preparing: select * from emp where id = ? 
13:47:57.406 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - ==> Parameters: 3(Integer)
13:47:57.458 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - ====>  Preparing: select id, dept_name from dept where id = ? 
13:47:57.459 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - ====> Parameters: 1(Integer)
13:47:57.463 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - <====      Total: 1
13:47:57.467 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - <==      Total: 1
夏侯瑾轩
研发部
```

##### 开启延迟加载

- 延迟加载：在需要使用部门属性的时候才发SQL去数据库查询对应的信息

- 开启延迟加载：mybatis配置文件中加入如下配置

  ```xml
  <settings>
      <!-- 其他配置... -->
      <!-- lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。默认为false -->
      <setting name="lazyLoadingEnabled" value="true"/>
      <!-- aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载。大于3.4.1版本以后的mybatis默认为false，之前默认为true -->
      <setting name="aggressiveLazyLoading" value="false"/>
  </settings>
  ```

- 执行结果：

  ```
  13:58:25.545 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - ==>  Preparing: select * from emp where id = ? 
  13:58:25.636 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - ==> Parameters: 3(Integer)
  13:58:25.745 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpByIdAssociate - <==      Total: 1
  夏侯瑾轩
  13:58:25.750 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - ==>  Preparing: select id, dept_name from dept where id = ? 
  13:58:25.750 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - ==> Parameters: 1(Integer)
  13:58:25.754 [main] DEBUG com.mxc.dao.DeptMapper.getDeptById - <==      Total: 1
  研发部
  ```

#### 查部门的同时查该部门下所有的员工

##### 代码

- `DeptMapper.java`

  ```java
  Department getDeptByIdCollection(Integer id);
  ```

- ``DeptMapper.xml`

  ```xml
  <resultMap id="deptMap" type="com.mxc.entity.Department">
          <id column="id" property="id"></id>
          <result column="dept_name" property="deptName"></result>
          <collection property="emps" ofType="com.mxc.entity.Employee"
                      column="id" select="com.mxc.dao.EmployeeMapper.getEmpByDeptId">		   </collection>
  </resultMap>
  <select id="getDeptByIdCollection" resultMap="deptMap">
      select id, dept_name from dept where id = #{id}
  </select>
  ```

- `EmployeeMapper.java`

  ```java
  List<Employee> getEmpsByDeptId(Integer deptId);
  ```

- `EmployeeMapper.xml`

  ```xml
  <select id="getEmpsByDeptId" resultType="com.mxc.entity.Employee">
      select * from emp where d_id = #{deptId}
  </select>
  ```


##### 执行及结果（延迟加载）

```java
/**
 * 获取SqlSessionFactory对象
 * @return
 * @throws IOException
 */
private SqlSessionFactory getSqlSessionFactory() throws IOException {
    return new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
}

@Test
public void testCollection() throws IOException {
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    SqlSession sqlSession = sqlSessionFactory.openSession();

    try {
        DeptMapper deptMapper = sqlSession.getMapper(DeptMapper.class);
        Department deptByIdCollection = deptMapper.getDeptByIdCollection(1);
        System.out.println(deptByIdCollection.getDeptName());
        List<Employee> emps = deptByIdCollection.getEmps();
        for (Employee emp : emps) {
            System.out.println("emp = " + emp);
        }
        sqlSession.commit();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        sqlSession.close();
    }
}
```

控制台输出：

```
15:17:59.458 [main] DEBUG com.mxc.dao.DeptMapper.getDeptByIdCollection - ==>  Preparing: select id, dept_name from dept where id = ? 
15:17:59.564 [main] DEBUG com.mxc.dao.DeptMapper.getDeptByIdCollection - ==> Parameters: 1(Integer)
15:17:59.698 [main] DEBUG com.mxc.dao.DeptMapper.getDeptByIdCollection - <==      Total: 1
研发部
15:17:59.703 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpsByDeptId - ==>  Preparing: select * from emp where d_id = ? 
15:17:59.704 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpsByDeptId - ==> Parameters: 1(Integer)
15:17:59.709 [main] DEBUG com.mxc.dao.EmployeeMapper.getEmpsByDeptId - <==      Total: 2
emp = Employee{id=3, lastName='夏侯瑾轩', gender='1', email='xhjx@outlook.com'}
emp = Employee{id=4, lastName='瑕', gender='0', email='xia@outlook.com'}
```



