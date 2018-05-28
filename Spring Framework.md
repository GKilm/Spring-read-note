# Spring Framework

## Spring

Beans	container	metadata	resources



A Spring IoC container manages one or more beans. These beans are created with metadata.

Beans are the backbone of application

## Container

作用：instantiate, configure, and assemble beans；manage beans

主要实现接口：BeanFactory ;context.ApplicationContext ;

实现类：(ClassPathXmlApplicationContext ；FileSystemXmlApplicationContext ;)<--

使用方法：

1.

```java
// create and configure beans
//我的理解是创建容器并由容器创建相应metadata里的bean instance。
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
// retrieve(获取) configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);
// use configured instance
List<String> userList = service.getUsernameList();
```

2.

```java
//等价于new ClassPathXmlApplicationContext();
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

## Bean

作用： 构成应用的主干

构建：the bean definition

实例化：

1. 使用构造器；

   ```xml
   <bean id="exampleBean" class="examples.ExampleBean"/>
   <bean name="anotherExample" class="examples.ExampleBeanTwo"/>
   ```

   

2. 静态工厂方法

   ```xml
   <bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
   ```

   ```java
   public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}
    public static ClientService createInstance() {
    return clientService;
    }
   }
   ```

3. 实例化工厂方法

   ```xml
   <!-- the factory bean, which contains a method called createInstance() -->
   <bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
   </bean>
   <!-- the bean to be created via the factory bean -->
   <bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
   
   <bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
   ```

   ```java
   public class DefaultServiceLocator {
    private static ClientService clientService = new ClientServiceImpl();
    private static AccountService accountService = new AccountServiceImpl();
    public ClientService createClientServiceInstance() {
    return clientService;
    }
    public AccountService createAccountServiceInstance() {
    return accountService;
    }
   }
   ```


The bean definition transfer to “BeanDefinition.class” in the Container.

### The bean definition

| Property                 | Explained in…                                 |
| ------------------------ | --------------------------------------------- |
| class                    | the section called “Instantiating beans”      |
| name                     | the section called “Naming beans”             |
| scope                    | the section “Bean scopes”                     |
| constructor arguments    | the section called “Dependency Injection”     |
| properties               | the section called “Dependency Injection”     |
| autowiring mode          | the section called “Autowiring collaborators” |
| lazy-initialization mode | the section called “Lazy-initialized beans”   |
| initialization method    | the section called “Initialization callbacks” |
| destruction method       | the section called “Destruction callbacks”    |

注：

class：inner class 

```java
<bean class="com.example.Foo$Bar"></bean>
```

name：

如果类没被命名，将采用class的类名，并将首字母小写。(如果第二个字母大写时，保持class名)

不提供名称的动机与使用 inner beans和autowiring collaborators有关；ref时必须给定name。

给定别名：

```xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```



### BeanDefinition

contain metadata:

1. 可实现类
2. Bean活动配置元素：声明bean在容器中如何活动(scope，lifecycle ，callbacks 等)
3. collaborators or dependencies
4. 其它的配置设置

### Scope



## metadata

作用：为Container实例化Bean提供information

组成：Bean‘s construction ，lifecycle，dependencies

来源形式：XML-based ；Annotation-based；Java-based

### XML-based 

文件导入：

```xml
<beans>
  <!-- using "../" not recommended
 	   >the runtime resolution process chooses the "nearest" classpath
	   <import> is provided by the beans
   -->
 <import resource="services.xml"/>
 <import resource="resources/messageSource.xml"/>
 <import resource="/resources/themeSource.xml"/>
 <bean id="bean1" class="..."/>
</beans>
```

拓展：.groovy等价于.xml

```groovy
beans {
 	dataSource(BasicDataSource) {
	driverClassName = "org.hsqldb.jdbcDriver"
 	url = "jdbc:hsqldb:mem:grailsDB"
 	username = "sa"
 	password = ""
 	settings = [mynew:"setting"]
 	}
 	sessionFactory(SessionFactory) {
 	dataSource = dataSource
 	}
 	myService(MyService) {
 		nestedBean = { AnotherBean bean ->
 			dataSource = dataSource
 		}
 	}
}
```



### Annotation-based 

### Java-based 





## Dependency

Container管理beans之间的依赖；beans合作以达到同一个目标；如果把Bean类比为节点，Dependency类比为线。

```java
public class SimpleMovieLister {
 // the SimpleMovieLister has a dependency on a MovieFinder
 private MovieFinder movieFinder;
 // a constructor so that the Spring container can inject a MovieFinder
 public SimpleMovieLister(MovieFinder movieFinder) {
 this.movieFinder = movieFinder;
 }
 // business logic that actually uses the injected MovieFinder is omitted...
}

```

### DI形式

容器ApplicationContext使用PropertyEditor改变BeanDefinition使得两种DI方式同时工作；强制依赖使用Constructor，可选依赖使用Setter。

1. Constructor-based DI

   specify argument：no；type；index；name

   ```xml
   <beans>
    <bean id="foo" class="x.y.Foo">
    <constructor-arg ref="bar"/>
    <constructor-arg ref="baz"/>		<!--no specify-->
    </bean>
   </beans>
   ```

   ```xml
   <bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>		<!--type specify-->
    <constructor-arg type="java.lang.String" value="42"/>
   </bean>
   ```

   ```xml
   <bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>		<!--index specify-->
    <constructor-arg index="1" value="42"/>
   </bean>
   ```

   ```xml
   <!-- 避免使用，脱离框架可能无法工作（需要compiled with the debug flag）或者使用 @ConstructorProperties（{"*"，"*"}）-->
   <bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>	<!--name specify-->
    <constructor-arg name="ultimateAnswer" value="42"/>
   </bean>
   ```

   

2. Setter-based DI

   ```java
   public class SimpleMovieLister {
    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;
    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
    }
    // business logic that actually uses the injected MovieFinder is omitted...
   }
   ```

### Dependency resolution process

DRP可以理解为how to complete dependency



## Resources

从资源加载的角度看Spring Framework。



