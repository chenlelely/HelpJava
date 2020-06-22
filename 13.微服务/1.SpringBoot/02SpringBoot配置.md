# 配置文件

## 1. 配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的：

- `application.properties`
- `application.yml  `

配置文件放在`src/main/resources`目录或者类路径`/config`下  

全局配置文件的可以对一些默认配置值进行修改  

## 2. YAML 语法

> .yml是YAML（ YAML Ain't Markup Language）语言的文件，以数据为中心，比json、 xml等更适合做配置文件  

https://yaml.org/

### 1. 基本语法

- `k:(空格)v`：表示一对键值对（空格必须有）；

- 以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的。
  - (缩进时不允许使用Tab键，只允许使用空格；缩进的空格数目不重要，只要相同层级的元素左侧对齐即可  )

- 大小写敏感  

### 2. 三种数据结构  

#### 字面量

`k: v`：字面直接来写；

- 字符串默认不用加上单引号或者双引号；

- `""`：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
  - `name:   "zhangsan \n lisi"`：输出：`zhangsan 换行  lisi`
- `''`：单引号；会转义特殊字符，特殊字符最终只是一个**普通的字符串数据**
  - `name:   ‘zhangsan \n lisi’`：输出：`zhangsan \n  lisi`

#### 对象（Map）

`k: v`：在下一行来写对象的属性和值的关系；注意缩进

```yaml
friend:
        lastName: zhangsan
        age: 21
```

行内写法：

````yaml
friend: {lastName: zhangsan,age: 21}
````

#### 数组/list/set

用`- `值表示数组中的一个元素，一组连词线`-`开头的行，构成一个数组，` []`为行内写法  

````yaml
pets:
    - cat
    - dog
    - pig
friends: 
    - lastName: zhangsan
      age: 21
    - lastName: lisi
      age: 22
````

```yaml
pets: [cat,dog,pig]
friends: [{lastName: zhangsan,age: 21},{lastName: lisi,age: 22}]
```

## 3. 配置文件属性注入

### @ConfigurationProperties获取值

与@Bean结合为属性赋值

与@PropertySource（只能用于properties文件）结合读取指定文件  

> - @Component注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。
>
> - @Bean注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。
>
> @Component和@Bean都是用来注册Bean并装配到Spring容器中，但是Bean比Component的自定义性更强。可以实现一些Component实现不了的自定义加载类，比如添加第三方类库中的bean组件时候只能使用@Bean。

- Java Bean

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @NotNull
    private String name;
    private Date birth;
    private String email;
    private List<Pet> pets;
    private Map<String,Object> friends;
    ....
}
```

- 配置文件：application.yml

```yaml
person:
  name: tom
  birth: 2000/12/12
  email: 133@qq.com
  pets:
    - name: catee
      birth: 2020/1/3
    - name: dogg
      birth: 2020/2/2
  friends: {boy: zhangsan, girl: eve}
```

> @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
>
> prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
>
> 只有这个组件是容器中的组件(@Component)，才能容器提供的@ConfigurationProperties功能；
>
> 我们可以在pom文件中导入配置文件处理器，以后编写配置就有提示了：
>
> ```xml
> <!--导入配置文件处理器，配置文件进行绑定就会有提示-->
> 		<dependency>
> 			<groupId>org.springframework.boot</groupId>
> 			<artifactId>spring-boot-configuration-processor</artifactId>
> 			<optional>true</optional>
> 		</dependency>
> ```

### @Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

### 配置文件注入值数据校验

使用`@Validated`注解

### @PropertySource&@ImportResource&@Bean

:one:@**PropertySource**：加载指定位置的配置文件；

`@PropertySource(value = {"classpath:person.properties"})`:

:two:@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；想让Spring的配置文件生效，加载进来；@**ImportResource**标注在一个配置类上

`@ImportResource(locations = {"classpath:beans.xml"})`

:three:@**Bean​**

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1. 配置类用**@Configuration**注解标识（对应Spring配置文件）
2. 使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring的xml配置文件
 * 在配置文件中用<bean><bean/>标签添加组件
 */
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

## 4.配置文件占位符

### 1. 随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

### 2. 占位符获取之前配置的值，如果没有可以是用:指定默认值

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```

## 5. Profile

### 1. 多Profile文件

我们在主配置文件编写的时候，文件名可以是   ·`application-{profile}.properties/yml`

- `application-dev.properties`
- `application-prod.properties`

默认使用`application.properties`的配置；

### 2. yml支持多文档块方式

```yml
server:
  port: 8081
spring:
  profiles:
    active: prod
---
server:
  port: 8083
spring:
  profiles: dev
---
server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```

### 3. 激活指定profile

1. 在配置文件中指定  `spring.profiles.active=dev`

2. 命令行：
   1. `java -jar spring-boot-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；`
   2. 可以直接在测试的时候，配置传入命令行参数

3. 虚拟机参数；

​		`-Dspring.profiles.active=dev`

## 6. 配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

- `file:./config/`
- `file:./`
- `classpath:/config/`
- `classpath:/`

优先级由高到底，高优先级的配置会覆盖低优先级的配置；SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

==我们还可以通过`spring.config.location`来改变默认的配置文件位置==

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

`java -jar spring-boot-01-SNAPSHOT.jar --spring.config.location=G:/application.properties`

### 外部配置加载顺序

**==SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置==**

:star:**1.命令行参数**

所有的配置都可以在命令行上进行指定

`java -jar spring-boot-02-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc`

多个配置用空格分开； `--配置项=值`

:v:2.来自java:comp/env的JNDI属性

:v:3.Java系统属性（System.getProperties()）

:v:4.操作系统环境变量

:v:5.RandomValuePropertySource配置的random.*属性值

==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==

:star:**6.jar包外部的`application-{profile}.properties`或`application.yml`(带spring.profile)配置文件**

:star:**7.jar包内部的`application-{profile}.properties`或`application.yml`(带spring.profile)配置文件**

==**再来加载不带profile**==

:star:**8.jar包外部的`application.properties`或`application.yml`(不带spring.profile)配置文件**

:star:**9.jar包内部的`application.properties`或`application.yml`(不带spring.profile)配置文件**

:v:10.@Configuration注解类上的`@PropertySource`

:v:11.通过`SpringApplication.setDefaultProperties`指定的默认属性

所有支持的配置加载来源；

[参考官方文档](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/htmlsingle/#boot-features-external-config)

## 7. 自动配置原理:star::heart:

[配置文件中可以配置的属性参考](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/htmlsingle/#common-application-properties)

### 1. 自动配置原理

SpringBoot启动的时候加载主配置类，开启了自动配置功能 ==@EnableAutoConfiguration==

:v:**@EnableAutoConfiguration 作用**：

 -  利用`EnableAutoConfigurationImportSelector`给容器中导入一些组件

-  可以查看`selectImports()`方法的内容；

-  `List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);`获取候选的配置

- `SpringFactoriesLoader.loadFactoryNames()`扫描所有jar包类路径下 ` META-INF/spring.factories`把扫描到的这些文件的内容包装成`properties`对象，从properties中获取到`EnableAutoConfiguration.class`类（类名）对应的值，然后把他们添加在容器中

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
.......
```

> 每一个这样的  `xxxAutoConfiguration`类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

:star:以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；

````java
@Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
 //将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中(配置文件属性注入)
@EnableConfigurationProperties(HttpEncodingProperties.class) 
//Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；
//判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication 
//判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnClass(CharacterEncodingFilter.class)  
//判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) 

public class HttpEncodingAutoConfiguration {
  	//他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;
   //只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
````

> 根据当前不同的条件判断，决定这个配置类是否生效？
>
> 一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；
>
> 所有在配置文件中能配置的属性都是在`xxxxProperties`类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类
>
> ```java
> @ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
> public class HttpEncodingProperties {
>   ....
> }
> ```

---

:star::star::star:**精髓：**

1. SpringBoot启动会加载大量的自动配置类
2. 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；
3. 我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）
4. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

- `xxxxAutoConfigurartion`：自动配置类；

- `xxxxProperties`:封装配置文件中相关属性；
- `yml/properties`文件中能配置的值就来源于[属性配置类]  

---

### 2. @Conditional细节

@Conditional派生注解（Spring注解版原生的@Conditional作用）

> 作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效；**

我们怎么知道哪些自动配置类生效；

**==我们可以通过启用  debug=true属性；来让控制台打印自动配置报告==**，这样我们就可以很方便的知道哪些自动配置类生效；

```java
=========================
AUTO-CONFIGURATION REPORT
=========================

Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
            
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        
```



