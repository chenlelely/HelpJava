# 07Spring与JDBC

## Spring数据访问

Spring的目标之一就是允许我们在开发应用程序时， 能够遵循面向对象（OO） 原则中的“针对接口编程”。  

将数据访问的功能放到一个或多个专注于此项任务的组件中。 这样的组件通常称为**数据访问对象（data access object， DAO） 或Repository。**  

![image-20200509174854956](08Spring与JDBC.assets/image-20200509174854956.png)![image-20200509213736652](08Spring与JDBC.assets/image-20200509213736652.png)

### 数据访问模板化  

Spring将数据访问过程中固定的和可变的部分明确划分为两个不同的类： **模板（template） 和回调（callback）** 。 模板管理过程中固定的部分， 而回调处理自定义的数据访问代码。   

![image-20200509175137298](08Spring与JDBC.assets/image-20200509175137298.png)

## 配置数据源  

无论选择Spring的哪种数据访问方式， 你都需要配置一个数据源的引用。   

- 通过JDBC驱动程序定义的数据源；
- 通过JNDI查找的数据源；
- 连接池的数据源。  

----

### 使用JNDI数据源  

> 这种配置的好处在于数据源完全可以在应用程序之外进行管理， 这样应用程序只需在访问数据库的时候查找数据源就可以了。   

位于jee命名空间下的`<jee:jndilookup>`元素可以用于检索JNDI中的任何对象（包括数据源） 并将其作为Spring的bean。   

![image-20200509180605150](08Spring与JDBC.assets/image-20200509180605150.png)

jndi-name属性用于指定JNDI中资源的名称。 如果只设置了jndi-name属性， 那么就会根据指定的名称查找数据源。 但是， 如果应用程序运行在Java应用服务器中， 你需要将resource-ref属性设置为true， 这样给定的jndi-name将会自动添加“java:comp/env/”前缀。  

Java配置：

![image-20200509180717554](08Spring与JDBC.assets/image-20200509180717554.png)



----

### 使用数据源连接池  

XML配置：

![image-20200509175504530](08Spring与JDBC.assets/image-20200509175504530.png)

Java配置：

![image-20200509175523421](08Spring与JDBC.assets/image-20200509175523421.png)

前四个属性是配置BasicDataSource所必需的  ：

![image-20200509175634281](08Spring与JDBC.assets/image-20200509175634281.png)

![image-20200509175615977](08Spring与JDBC.assets/image-20200509175615977.png)

> 使用属性文件
>
> 通过`<context:property-placeholder>`引入属性文件，以`${xxx}`的方式引用属性。示例代码如下：
>
> ![image-20200509214337932](08Spring与JDBC.assets/image-20200509214337932.png)
>
> ![image-20200509214404714](08Spring与JDBC.assets/image-20200509214404714.png)

### 使用嵌入式的数据源  

对于开发和测试来讲， 嵌入式数据库都是很好的可选方案。 这是因为每次重启应用或运行测试的时候， 都能够重新填充测试数据  

Spring的jdbc命名空间能够简化嵌入式数据库的配置。  

![image-20200509175911202](08Spring与JDBC.assets/image-20200509175911202.png)

`<jdbc:script>`元素： 第一个引用了schema.sql， 它包含了在数据库中创建表的SQL； 第二个引用了test-data.sql， 用来将测试数据填充到数据库中  

Java 配置：

![image-20200509180100451](08Spring与JDBC.assets/image-20200509180100451.png)

### 使用profile选择数据源  

例如， 对于开发期来说， `<jdbc:embedded-database>`元素是很合适的， 而在QA环境中， 你可能希望使用DBCP的BasicDataSource， 在生产部署环境下， 可能需要使用`<jee:jndi-lookup>`。  

![image-20200509180230260](08Spring与JDBC.assets/image-20200509180230260.png)

-----

Spring XML代替Java配置  ：

![image-20200509180430754](08Spring与JDBC.assets/image-20200509180430754.png)

## 在Spring中使用JDBC  

使用JDBC所提供的直接操作数据库的API  :

![image-20200509180937657](08Spring与JDBC.assets/image-20200509180937657.png)

### 使用JDBC模板  

Spring为JDBC提供了三个模板类供选择：

- JdbcTemplate： 最基本的Spring JDBC模板， 这个模板支持简单的JDBC数据库访问功能以及基于索引参数的查询；

---

- **使用JdbcTemplate来插入数据**

为了让JdbcTemplate正常工作， 只需要为其设置DataSource就可以了  

![image-20200509181139396](08Spring与JDBC.assets/image-20200509181139396.png)

现在， 我们可以将jdbcTemplate装配到Repository中并使用它来访问数据库。   

![image-20200509181255122](08Spring与JDBC.assets/image-20200509181255122.png)

> 使用了@Inject注解， 因此在创建的时候， 会自动获得一个JdbcOperations对象。 JdbcOperations是一个接口， 定义了JdbcTemplate所实现的操作。 通过注入JdbcOperations， 而不是具体的JdbcTemplate， 能够保证JdbcSpitterRepository通过JdbcOperations接口达到与JdbcTemplate保持松耦合。  

基于JdbcTemplate的addSpitter()方法  :

![image-20200509181620319](08Spring与JDBC.assets/image-20200509181620319.png)

----

- **使用JdbcTemplate来读取数据**  

![image-20200509181744336](08Spring与JDBC.assets/image-20200509181744336.png)

> queryForObject()方法有三个参数：
>
> - String对象， 包含了要从数据库中查找数据的SQL；
> - RowMapper对象， 用来从ResultSet中提取数据并构建域对象（本例中为Spitter） ；  
> - 可变参数列表， 列出了要绑定到查询上的索引参数值。  
>
> 对于查询返回的每一行数据， JdbcTemplate将会调用RowMapper的mapRow()方法， 并传入一个ResultSet和包含行号的整数。 在SpitterRowMapper的mapRow()方法中， 我们创建了Spitter对象并将ResultSet中的值填充进去。  

----

- **在JdbcTemplate中使用Java 8的Lambda表达式  **

因为RowMapper接口只声明了addRow()这一个方法， 因此它完全符合函数式接口（functional interface） 的标准。  

![image-20200509182130825](08Spring与JDBC.assets/image-20200509182130825.png)

