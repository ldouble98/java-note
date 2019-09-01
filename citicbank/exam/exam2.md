# 第三四周复习知识点
参考 spring官网 https://spring.io/

## Spring 体系结构（了解）
- Core technologies: dependency injection, events, resources, i18n, validation, data binding, type conversion, SpEL, AOP.

- Testing: mock objects, TestContext framework, Spring MVC Test, WebTestClient.

- Data Access: transactions, DAO support, JDBC, ORM, Marshalling XML.

- Spring MVC and Spring WebFlux web frameworks.

- Integration: remoting, JMS, JCA, JMX, email, tasks, scheduling, cache.

- Languages: Kotlin, Groovy, dynamic languages.

##Spring 的IoC容器（了解）
spring提供了两种容器，分别是`BeanFactory`和`ApplicationContext`

### BeanFactory
BeanFactory接口有多个`实现类`，最常用的是`XMLBeanFactory`,根据XML配置文件来装配Bean

### ApplicationContext

`ApplicationContext` 是BeanFactory的`子接口`，在初始化时会进行自检，有利于检查所依赖的
属性是否注入。
## DI 依赖注入 （重点）
- @Autowired 注解方式
        
        @Autowired：自动装配
            - 默认byType的方式注入
            - 通过将@Autowired注解放在构造器上来完成构造器注入，默认构造器参数通过类型自动装配
            - 通过将@Autowired注解放在构造器上来完成字段注入
            - 通过将@Autowired注解放在方法上来完成方法参数注入
        @Resource
            - java java的注解，默认使用byName的方式，

    ```
    java 代码：
    @Autowired //构造器注入  
    private Test(String message) {  
        this.message = message;  
    } 
    // 属性注入
    @Autowired
    public String message;
    
    ``` 
- 构造器注入 (默认构造器参数、通过类型装配)
   
   使用`带参`的`构造方法` `constructor-arg` 
   在XML文件中同样不用<property>的形式，而是使用<constructor-arg>标签
   ref属性同样指向其它<bean>标签的name属性（参考java自学宝典544）
   ```java
  // java
  public class Person{
    private String name;
    private int age;
    public Person(){};
    public Person(String name,int age){
      super();
      this.name = name;
      this.age = age;
      }
  }
    ```
   ```xml
   <!--xml-->
   <!--无setter方法通过构造方式装配Person实例-->
      
    <bean name = "person" class = "****.Person">
          <constructor-arg index = "0" value="lxs"/>
          <constructor-arg index = "1" value="21"/>
    </bean>
    ```
    ```xml
    <!--通过setter装配Person实例-->
      
    <bean name = "person" class = "****.Person">
          <property  name="name" value="lxs"/>
          <property name="age" value="21"/>
    </bean>
    ```
   
- 静态工厂注入
   
        TODO
- 实例工厂注入

        TODO

## Spring Annotation (重点)
@Component 装配自己写的类

@Bean 装配外部写的类

@Autowired  Bean的自动装配

@Resource

@Service 标注一个业务逻辑组件类

@Controller 标注一个控制器组件类

@Repository ：标注一个DAO组件类

## Spring Bean 的作用域
 `Singleton` :单例模式，IoC容器中只会存在一个实例对象，每一个context中只有一次实例spring默认是单例的。
 `Prototype`:
 
 参考一二组第二次考试答案
 
 1. `singleton`：Spring IoC容器中只会存在`一个`共享`的Bean实例，无论有多少个Bean引用它，始终指向`同一对象`。Singleton作用域是Spring中的缺省作用域。
 
 2. `prototype`：每次通过Spring容器获取prototype定义的bean时，容器都将创建一个`新的Bean实例`，每个Bean实例都有`自己的属性和状态`，而singleton全局只有一个对象。
 
 3. `request`：在一次`Http请求`中，容器会返回该Bean的同一实例。而对`不同的Http请求`则会产生`新的Bean`，而且该bean仅在当前`Http Request`内有效。
 
 4. `session`：在一次`Http Session`中，容器会返回该Bean的同一实例。而对`不同的Session请求`则会创建`新的实例`，该bean实例仅在当前`Session`内有效。
 
 5. `global Session`：在一个`全局的Http Session`中，容器会返回该Bean的同一个实例，仅在使用`portlet context`时有效。

## Spring AOP （重点） 
参考 [AOP](../../java框架/spring/AOP.md)

## Spring 事务处理 （重点）

    TODO //待办
## SpringMVC 的流程（重点）
参考 [SpringMVC](../../java框架/spring/SpringMVC.md)

## springBean 的 生命周期（熟悉）
参考 [SpringMVC](../../java框架/spring/SpringBean生命周期.md)





   