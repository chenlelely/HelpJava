# Mybatis原理入门

GitHub：https://github.com/mybatis

web项目：https://github.com/mybatis/jpetstore-6

[MyBatis中文官网](http://www.mybatis.cn/)



## Mybatis-Hello程序

### Mybatis--Hello-easy

1、创建JavaBean实体类

2、**创建MyBatis全局配置文件**
MyBatis 的全局配置文件包含了影响 MyBatis 行为甚深的设置（ settings）和属性（ properties）信息、如**数据库连接池、事务管理器**信息等。指导着MyBatis进行工作。  

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
				<property name="username" value="root" />
				<property name="password" value="123456" />
			</dataSource>
		</environment>
	</environments>
	<!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
	<mappers>
		<mapper resource="EmployeeMapper.xml" />
	</mappers>
</configuration>
```

3、 **创建SQL映射文件**
映射文件的作用就相当于是定义Dao接口的实现类如何工作。这也是我们使用MyBatis时编写的最多的文件。  

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapper">
<!-- 
namespace:名称空间;
id：唯一标识;    （名称空间he唯一标识确定SQL语句）
resultType：返回值类型
#{id}：从传递过来的参数中取出id；public Employee getEmpById(Integer id);
 -->
	<select id="selectEmp" resultType="com.atguigu.mybatis.bean.Employee">
		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
	</select>
</mapper>
```



> ##### 测试  
>
> 1、根据全局配置文件， 利用SqlSessionFactoryBuilder创建SqlSessionFactory  
>
> ```java
> public SqlSessionFactory getSqlSessionFactory() throws IOException {
> 		String resource = "mybatis-config.xml";
> 		InputStream inputStream = Resources.getResourceAsStream(resource);
> 		return new SqlSessionFactoryBuilder().build(inputStream);
> 	}
> ```
>
> 2、使用SqlSessionFactory获取sqlSession对象。一个SqlSession对象代表和数据库的一次会话，不是线程安全的，不应该作为成员变量。
>
> ![image-20200526181647718](01Mybatis入门.assets/image-20200526181647718.png)
>
> 3、使用SqlSession根据方法id进行操作  
>
> ```java
> SqlSession openSession = sqlSessionFactory.openSession();
> try {
> //使用sql的唯一标志（名称空间.id）来告诉MyBatis执行哪个sql。sql都是保存在sql映射文件中的。
> Employee employee = openSession.selectOne("com.atguigu.mybatis.EmployeeMapper.selectEmp", 1);
> System.out.println(employee);
> } finally {
> openSession.close();
> }
> ```

### Mybatis-Hello-接口编程

1、创建JavaBean实体类

2、创建MyBatis全局配置文件

3、**创建一个Dao接口**

```java
public interface EmployeeMapper {
	public Employee getEmpById(Integer id);
}
```

4、**修改Mapper映射文件的名称空间和Id**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapper">
<!-- namespace:名称空间;指定为接口的全类名 
	id名字和方法名一致：public Employee getEmpById(Integer id);
--> 
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee">
		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
	</select>
</mapper>
```

> （1）Mapper接口方法名和mapper.xml中定义的每个sql的id相同；
>
> （2）Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同；
>
> （3）Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同；
>
> （4）Mapper.xml文件中的namespace即是mapper接口的类路径。

> ##### 测试
>
> 1、根据全局配置文件， 利用SqlSessionFactoryBuilder创建SqlSessionFactory  
>
> 2、使用SqlSessionFactory获取sqlSession对象。
>
> 3、获取接口的实现类对象：会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
>
> ```java
> SqlSession openSession = sqlSessionFactory.openSession();
> try {
> EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);//代理类
> Employee employee = mapper.getEmpById(1);
> System.out.println(mapper.getClass());//Proxy
> System.out.println(employee);
> } finally {
> openSession.close();
> }
> ```

---

> SqlSession 的实例不是线程安全的，因此是不能被共享的。
>
> SqlSession每次使用完成后需要正确关闭，这个关闭操作是必须的 SqlSession可以直接调用方法的id进行数据库操作，但是我们一般还是推荐使用SqlSession获取到Dao接口的代理类，执行代理对象的方法，可以更安全的进行类型检查操作  