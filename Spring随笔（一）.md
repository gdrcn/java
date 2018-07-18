# Spring 

### 什么是 Spring ?
> Spring是一个开源框架，Spring是于2003 年兴起的一个轻量级的Java 开发框架，由Rod Johnson 在其著作Expert One-On-One J2EE Development and Design中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为 J2EE 应用程序开发提供集成的框架。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。

- 简单来说，Spring是一个分层的JavaSE/EE full-stack(一站式) 轻量级开源框架


### Spring 特点
- 方便解耦，简化开发
> 通过Spring提供的IoC容器，我们可以将对象之间的依赖关系交由Spring进行控制，避免硬编码所造成的过度程序耦合。有了Spring，用户不必再为单实例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用。
- AOP编程的支持
> 通过Spring提供的AOP功能，方便进行面向切面的编程，许多不容易用传统OOP实现的功能可以通过AOP轻松应付
- 声明式事务的支持
> 在Spring中，我们可以从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活地进行事务的管理，提高开发效率和质量
- 方便程序的测试
> 可以用非容器依赖的编程方式进行几乎所有的测试工作，在Spring里，测试不再是昂贵的操作，而是随手可做的事情。例如：Spring对Junit4支持，可以通过注解方便的测试Spring程序
- 方便集成各种优秀框架
> Spring不排斥各种优秀的开源框架，相反，Spring可以降低各种框架的使用难度，Spring提供了对各种优秀框架（如Struts,Hibernate、Hessian、Quartz）等的直接支持
- 降低Java EE API的使用难度
> Spring对很多难用的Java EE API（如JDBC，JavaMail，远程调用等）提供了一个薄薄的封装层，通过Spring的简易封装，这些Java EE API的使用难度大为降低
- Java 源码是经典学习范例
> Spring的源码设计精妙、结构清晰、匠心独运，处处体现着大师对Java设计模式灵活运用以及对Java技术的高深造诣。Spring框架源码无疑是Java技术的最佳实践范例。如果想在短时间内迅速提高自己的Java技术水平和应用开发水平，学习和研究Spring源码将会使你收到意想不到的效果

### 框架结构
1. 核心容器：核心容器提供 Spring 框架的基本功能(Spring Core)。核心容器的主要组件是BeanFactory，它是工厂模式的实现。BeanFactory使用控制反转（IOC）模式将应用程序的配置和依赖性规范与实际的应用程序代码分开
2. Spring 上下文：Spring上下文是一个配置文件，向Spring框架提供上下文信息。Spring上下文包括企业服务，例如JNDI、EJB、电子邮件、国际化、校验和调度功能。
3. Spring AOP：通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。
4. Spring DAO：JDBCDAO抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
5. Spring ORM：Spring 框架插入了若干个ORM框架，从而提供了 ORM 的对象关系工具，其中包括JDO、Hibernate和iBatisSQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构
6. Spring Web 模块：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
7. Spring MVC 框架：MVC框架是一个全功能的构建 Web应用程序的 MVC 实现。通过策略接口，MVC框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。模型由javabean构成，存放于Map；视图是一个接口，负责显示模型；控制器表示逻辑代码，是Controller的实现。Spring框架的功能可以用在任何J2EE服务器中，大多数功能也适用于不受管理的环境。Spring 的核心要点是：支持不绑定到特定 J2EE服务的可重用业务和数据访问对象。毫无疑问，这样的对象可以在不同J2EE 环境（Web 或EJB）、独立应用程序、测试环境之间重用。


### 框架特征
- 轻量——从大小与开销两方面而言Spring都是轻量的。完整的Spring框架可以在一个大小只有1MB多的JAR文件里发布。并且Spring所需的处理开销也是微不足道的。此外，Spring是非侵入式的：典型地，Spring应用中的对象不依赖于Spring的特定类
- 控制反转——Spring通过一种称作控制反转（IoC）的技术促进了低耦合。当应用了IoC，一个对象依赖的其它对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象。你可以认为IoC与JNDI相反——不是对象从容器中查找依赖，而是容器在对象初始化时不等对象请求就主动将依赖传递给它
- 面向切面——Spring提供了面向切面编程的丰富支持，允许通过分离应用的业务逻辑与系统级服务（例如审计（auditing）和事务（transaction）管理）进行内聚性的开发。应用对象只实现它们应该做的——完成业务逻辑——仅此而已。它们并不负责（甚至是意识）其它的系统级关注点，例如日志或事务支持。
- 容器——Spring包含并管理应用对象的配置和生命周期，在这个意义上它是一种容器，你可以配置你的每个bean如何被创建——基于一个可配置原型（prototype），你的bean可以创建一个单独的实例或者每次需要时都生成一个新的实例——以及它们是如何相互关联的。然而，Spring不应该被混同于传统的重量级的EJB容器，它们经常是庞大与笨重的，难以使用
- 框架——Spring可以将简单的组件配置、组合成为复杂的应用。在Spring中，应用对象被声明式地组合，典型地是在一个XML文件里。Spring也提供了很多基础功能（事务管理、持久化框架集成等等），将应用逻辑的开发留给了你。
- MVC——Spring的作用是整合，但不仅仅限于整合，Spring 框架可以被看做是一个企业解决方案级别的框架。客户端发送请求，服务器控制器（由DispatcherServlet实现的)完成请求的转发，控制器调用一个用于映射的类HandlerMapping，该类用于将请求映射到对应的处理器来处理请求。HandlerMapping 将请求映射到对应的处理器Controller（相当于Action）在Spring 当中如果写一些处理器组件，一般实现Controller 接口，在Controller 中就可以调用一些Service 或DAO 来进行数据操作 ModelAndView 用于存放从DAO 中取出的数据，还可以存放响应视图的一些数据。 如果想将处理结果返回给用户，那么在Spring 框架中还提供一个视图组件ViewResolver，该组件根据Controller 返回的标示，找到对应的视图，将响应response 返回给用户。

### spring容器的创建
- 第一种方法
```
    <!--
          创建对象的第一种方式：利用无参构造器
          id:唯一标识符
          class:类的全类名
    -->

    <bean id="hello" class="com.wushaoning.po.Hello" ></bean>
    <!-- 别名属性  name：和bean的 id 属性对应 -->
    <alias name="hello" alias="helloIoc"/>
```

```
public class Hello {

    public void hello(){
      System.out.println("hello");
    }
}
```

```
    @Test
    public void hello() {
        //1、启动 spring 容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2、从 spring 容器中取出数据
        Hello hello = (Hello) context.getBean("hello");
        //3、通过对象调用方法
        hello.hello();
        //利用配置文件 alias 别名属性创建对象
        Hello hello1 = (Hello) context.getBean("helloIoc");
        hello1.hello();
    }
```

- 第二种工厂方法
```
    <!--
        创建对象的第二种方式：利用静态工厂方法
        factory-method：静态工厂类的获取对象的静态方法
        class:静态工厂类的全类名
      -->
    <bean id="helloStaticFactory" factory-method="getInstances" class="com.wushaoning.po.HelloStaticFactory"></bean>
```

```
public class HelloStaticFactory {
    public HelloStaticFactory(){
        System.out.println("HelloStaticFactory constructor");
    }

    //静态工厂方法
    public static Hello getInstances(){
        return new Hello();
    }
}
```

```
    @Test
    public void createObjectStaticFactory(){
        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        Hello staticFactory =
                (Hello) context.getBean("helloStaticFactory");
        staticFactory.hello();
    }
```

- 第三种创建方法
```

    <!--
        创建对象的第三种方式：利用实例工厂方法
        factory-bean:指定当前Spring中包含工厂方法的beanID
        factory-method:工厂方法名称
      -->
    <bean id="instanceFactory" class="com.wushaoning.po.HelloInstanceFactory"></bean>
    <bean id="instance" factory-bean="instanceFactory" factory-method="getInstance"></bean>
```
```
    /**
     * Spring 容器利用实例工厂方法创建对象
     */
    @Test
    public void createObjectInstanceFactory(){
        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        Hello staticFactory =
                (Hello) context.getBean("instance");
        
        staticFactory.hello();
    }
```

### Spring创建对象的时机
1. 默认情况下，启动 spring 容器便创建对象（遇到bean便创建对象）
2. 在spring的配置文件bean中有一个属性lazy-init="default/true/false"

 　　　　①、如果lazy-init为"default/false"在启动spring容器时创建对象（默认情况）

　　　　 ②、如果lazy-init为"true",在context.getBean时才要创建对象
　　　　 
> 在第一种情况下可以在启动spring容器的时候，检查spring容器配置文件的正确性，如果再结合tomcat,如果spring容器不能正常启动，整个tomcat就不能正常启动。但是这样的缺点是把一些bean过早的放在了内存中，如果有数据，则对内存来是一个消耗。

>　反过来，在第二种情况下，可以减少内存的消耗，但是不容易发现错误

#### spring的bean种的scope
- "singleton/prototype/request//session/global session"

一. 默认的scope的值是singleton，即产生的对象是单例的
```
//applicationContext.xml 文件中配置：
<bean id="helloIoc" scope="singleton" class="com.ys.ioc.HelloIoc" ></bean>
```
```
//spring 容器默认产生对象是单例的 scope="singleton"
    @Test
    public void test_scope_single_CreateObject(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        HelloIoc hello1 = (HelloIoc) context.getBean("helloIoc");
        HelloIoc hello2 = (HelloIoc) context.getBean("helloIoc");
        System.out.println(hello1.equals(hello2)); //true
    }
```
二. scope=“prototype”。 多例模式， 并且spring容器启动的时候并不会创建对象，而是在得到 bean 的时候才会创建对象

```
applicationContext.xml 文件中配置：

<bean id="helloIoc" scope="prototype" class="com.ys.ioc.HelloIoc" ></bean>
```

```
//spring 容器默认产生对象是单例的 scope="prototype"
    @Test
    public void test_scope_prototype_CreateObject(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        HelloIoc hello1 = (HelloIoc) context.getBean("helloIoc");
        HelloIoc hello2 = (HelloIoc) context.getBean("helloIoc");
        System.out.println(hello1.equals(hello2)); //false
    }
```

- 总结：在单例模式下，启动 spring 容器，便会创建对象；在多例模式下，启动容器并不会创建对象，获得 bean 的时候才会创建对象


### spring的生命周期
```
package com.ys.ioc;
/**
 * Spring 容器的生命周期
 * @author hadoop
 *
 */
public class SpringLifeCycle {
    public SpringLifeCycle(){
        System.out.println("SpringLifeCycle");
    }
    //定义初始化方法
    public void init(){
        System.out.println("init...");
    }
    //定义销毁方法
    public void destroy(){
        System.out.println("destroy...");
    }
     
    public void sayHello(){
        System.out.println("say Hello...");
    }
}
```
```
<!-- 生命周期 -->
    <bean id="springLifeCycle" init-method="init" destroy-method="destroy" class="com.ys.ioc.SpringLifeCycle"></bean>
```
```
//spring 容器的初始化和销毁
    @Test
    public void testSpringLifeCycle(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringLifeCycle hello = (SpringLifeCycle) context.getBean("springLifeCycle");
         
        hello.sayHello();
         
        //销毁spring容器
        ClassPathXmlApplicationContext classContext = (ClassPathXmlApplicationContext) context;
        classContext.close();
    }
```
> 控制台打印
```
SpringLifeCycle
init...
sat Hello...
....
....
.
....
destroy...
```


- 分析
1. spring容器创建对象
2. 执行init方法
3. 调用自己的方法
4. 当spring容器关闭的时候执行destroy方法

- 注意：当scope为"prototype"时，调用 close（） 方法时是不会调用 destroy 方法的
- 

## 依赖注入
- spring动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的。

> 简单来说什么是依赖注入，就是给属性赋值（包括基本数据类型和引用数据类型）

#### 利用set方法给属性赋值
```
package com.ys.di;
 
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
 
public class Person {
    private Long pid;
    private String pname;
    private Student students;
    private List lists;
    private Set sets;
    private Map maps;
    private Properties properties;
     
    public Long getPid() {
        return pid;
    }
    public void setPid(Long pid) {
        this.pid = pid;
    }
    public String getPname() {
        return pname;
    }
    public void setPname(String pname) {
        this.pname = pname;
    }
    public Student getStudents() {
        return students;
    }
    public void setStudents(Student students) {
        this.students = students;
    }
    public List getLists() {
        return lists;
    }
    public void setLists(List lists) {
        this.lists = lists;
    }
    public Set getSets() {
        return sets;
    }
    public void setSets(Set sets) {
        this.sets = sets;
    }
    public Map getMaps() {
        return maps;
    }
    public void setMaps(Map maps) {
        this.maps = maps;
    }
    public Properties getProperties() {
        return properties;
    }
    public void setProperties(Properties properties) {
        this.properties = properties;
    }
     
}
```

```
<!--
    property是用来描述一个类的属性
    基本类型的封装类、String等需要值的类型用value赋值
    引用类型用ref赋值
-->
<bean id="person" class="com.ys.di.Person">
    <property name="pid" value="1"></property>
    <property name="pname" value="vae"></property>
    <property name="students">
        <ref bean="student"/>
    </property>
     
    <property name="lists">
        <list>
            <value>1</value>
            <ref bean="student"/>
            <value>vae</value>
        </list>
    </property>
     
    <property name="sets">
        <set>
            <value>1</value>
            <ref bean="student"/>
            <value>vae</value>
        </set>
    </property>
     
    <property name="maps">
        <map>
            <entry key="m1" value="1"></entry>
            <entry key="m2" >
                <ref bean="student"/>
            </entry>
        </map>
    </property>   
     
    <property name="properties">
        <props>
            <prop key="p1">p1</prop>
            <prop key="p2">p2</prop>
        </props>
    </property>  
     
</bean>
 
 
<bean id="student" class="com.ys.di.Student"></bean>
```
```
//利用 set 方法给对象赋值
    @Test
    public void testSet(){
        //1、启动 spring 容器
        //2、从 spring 容器中取出数据
        //3、通过对象调用方法
        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person) context.getBean("person");
        System.out.println(person.getPname());//vae
    }
```

#### 利用构造函数给属性赋值
- 第一步：在实体类Person.java中添加两个构造方法：有参和无参
```
//默认构造函数
    public Person(){}
    //带参构造函数
    public Person(Long pid,Student students){
        this.pid = pid;
        this.students = students;
    }
```
- 第二步：在 applicationContext.xml 中进行赋值
```
<!-- 根据构造函数赋值 -->
    <!--
        index  代表参数的位置  从0开始计算
        type   指的是参数的类型,在有多个构造函数时，可以用type来区分，要是能确定是那个构造函数，可以不用写type
        value  给基本类型赋值
        ref    给引用类型赋值
      -->
    <bean id="person_con" class="com.ys.di.Person">
        <constructor-arg index="0" type="java.lang.Long" value="1">
        </constructor-arg>       
        <constructor-arg index="1" type="com.ys.di.Student" ref="student_con"></constructor-arg>
    </bean>
    <bean id="student_con" class="com.ys.di.Student"></bean>
```

```
//利用 构造函数 给对象赋值
    @Test
    public void testConstrutor(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person) context.getBean("person_con");
        System.out.println(person.getPid());//1
    }
```
　　1. 如果spring的配置文件中的bean中没有<constructor-arg>该元素，则调用默认的构造函数

　　2. 如果spring的配置文件中的bean中有<constructor-arg>该元素，则该元素确定唯一的构造函数

### Annotation注解
第一步：在 applicationContext.xml 中引入命名空间
第二步：在 applicationContext.xml 文件中引入注解扫描器
```
<context: component-scan base-package="com.ys.annotation"></context:component-scan>

```
第三步：在person类添加注解@Compnent
```
@Component
public class Person {
    
}
```
第四步：测试
```
@Test
    public void testAnnotation(){
        //1、启动 spring 容器
        //2、从 spring 容器中取出数据
        //3、通过对象调用方法
        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person) context.getBean("person");
        System.out.println(person.getPname());
    }
```

#### 注解@Resource
- @Resource 注解，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Resource 的使用来消除 set ，get方法。首先创建一个 学生类 Student.java

```
public class Student{
    public void descStudent(){
        System.out.println("descStudent...");
    }
}
```
然后在Person类中添加一个属性Student
```
public class Person{
    private Student student;
    
    public void showStudent(){
        this.studemt.descStudent();
    }
}

```

- 不使用注解
```
<property name="students">
    <ref bean="student"/>
</property>
<bean id="student" class="com.ys.annotation_di.Student"></bean>
```

- 使用注解
```
@Component("personDI")
public class Person{
    @Resource(name="student")
    private Student student;
    
    public void showStudent(){
        this.studemt.descStudent();
    }
}


@Component("student")
public class Student{
    
}
```

- 注解名称要相同
@Resource注解以后，判断该注解name的属性是否为""(name没有写)

①、如果没有写name属性，则会让属性的名称的值和spring配置文件bean中ID的值做匹配（如果没有进行配置，也和注解@Component进行匹配），如果匹配成功则赋值，如果匹配不成功，则会按照spring配置文件class类型进行匹配，如果匹配不成功，则报错

②、如果有name属性，则会按照name属性的值和spring的bean中ID进行匹配，匹配成功，则赋值，不成功则报错


### 注解 @Autowired
- 功能和注解 @Resource 一样，可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。只不过注解@Resource 是按照名称来进行装配，而@Autowired 则是按照类型来进行装配。


- 第一步： 创建接口 PersonDao
```
package com.ys.autowired;
 
public interface PersonDao {
     
    public void savePerson();
 
}
```

- 第二步：创建一个接口实现类 PersonDaoImplOne
```
package com.ys.autowired;
 
import org.springframework.stereotype.Component;
 
@Component("personDaoImplOne")
public class PersonDaoImplOne implements PersonDao{
 
    @Override
    public void savePerson() {
        System.out.println("save Person One");
         
    }
 
}
```

- 第三步：创建PersonService。

```
package com.ys.autowired;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
@Service("personService")
public class PersonService{
    @Autowired
    private PersonDao personDao;
     
    public void savePerson() {
        this.personDao.savePerson();
    }
 
}
```
- 注意：这里我们在 private PesronDao personDao 上面添加了注解 @Autowired，它首先会根据类型去匹配，PersonDao 是一个接口，它的实现类是 PesronDaoImpOne，那么这里的意思就是：PersonDao personDao = new PersonDaoImpOne();

- 那若有多个实现类怎么解决

1. 第一种方法：更改名称
```
@Service(”personService“)
public class PersonService{
    @Autowired
    //private PersonDao personDao;
    private PersonDao personDaoImplOne;
    //与实现接口名称相同
}
```
2. 第二种方法:@Autowired和@Qualifier(”名称“)相同
```
@Service("personService")
public class PersonService{
    @Autowired
    @Qualifier("personDaoImplOne")
    private PersonDao personDao;
}
```

> 在使用@Autowired时，首先在容器中查询对应类型的bean

　　　　如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据

　　　　如果查询的结果不止一个，那么@Autowired会根据名称来查找。

　　　　如果查询的结果为空，那么会抛出异常。解决方法时，使用required=false
