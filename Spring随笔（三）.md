## AOP注解
#### xml实现AOP
1. 接口 UserService
```
package com.ys.aop;
 
public interface UserService {
    //添加 user
    public void addUser();
    //删除 user
    public void deleteUser();
}
```
2. 实现类UserServiceImpl
```
package com.ys.aop;
 
public class UserServiceImpl implements UserService{
    @Override
    public void addUser() {
        System.out.println("增加 User");
    }
    @Override
    public void deleteUser() {
        System.out.println("删除 User");
    }
}
```
3. 切面类，也就是通知类MyAspect
```
package com.ys.aop;
 
import org.aspectj.lang.JoinPoint;
 
 
public class MyAspect {
    /**
     * JoinPoint 能获取目标方法的一些基本信息
     * @param joinPoint
     */
    public void myBefore(JoinPoint joinPoint){
        System.out.println("前置通知 ： " + joinPoint.getSignature().getName());
    }
     
    public void myAfterReturning(JoinPoint joinPoint,Object ret){
        System.out.println("后置通知 ： " + joinPoint.getSignature().getName() + " , -->" + ret);
    }
     
    public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
        System.out.println("抛出异常通知 ： " + e.getMessage());
    }
     
    public void myAfter(){
        System.out.println("最终通知");
    }
 
}
```

4. AOP配置文件 applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">   
    <!--1、创建目标类 -->
    <bean id="userService" class="com.ys.aop.UserServiceImpl"></bean>  
    <!--2、创建切面类（通知）  -->
    <bean id="myAspect" class="com.ys.aop.MyAspect"></bean>
     
    <!--3、aop编程 
        3.1 导入命名空间
        3.2 使用 <aop:config>进行配置
                proxy-target-class="true" 声明时使用cglib代理
                如果不声明，Spring 会自动选择cglib代理还是JDK动态代理
            <aop:pointcut> 切入点 ，从目标对象获得具体方法
            <aop:advisor> 特殊的切面，只有一个通知 和 一个切入点
                advice-ref 通知引用
                pointcut-ref 切入点引用
        3.3 切入点表达式
            execution(* com.ys.aop.*.*(..))
            选择方法         返回值任意   包             类名任意   方法名任意   参数任意
     
    -->
    <aop:config>
        <aop:aspect ref="myAspect">
        <!-- 切入点表达式 -->
        <aop:pointcut expression="execution(* com.ys.aop.*.*(..))" id="myPointCut"/>
        <!-- 3.1 前置通知
                <aop:before method="" pointcut="" pointcut-ref=""/>
                    method : 通知，及方法名
                    pointcut :切入点表达式，此表达式只能当前通知使用。
                    pointcut-ref ： 切入点引用，可以与其他通知共享切入点。
                通知方法格式：public void myBefore(JoinPoint joinPoint){
                    参数1：org.aspectj.lang.JoinPoint  用于描述连接点（目标方法），获得目标方法名等
        -->
        <aop:before method="myBefore" pointcut-ref="myPointCut"/>
         
         
        <!-- 3.2后置通知  ,目标方法后执行，获得返回值
                <aop:after-returning method="" pointcut-ref="" returning=""/>
                    returning 通知方法第二个参数的名称
                通知方法格式：public void myAfterReturning(JoinPoint joinPoint,Object ret){
                    参数1：连接点描述
                    参数2：类型Object，参数名 returning="ret" 配置的
        -->
        <aop:after-returning method="myAfterReturning" pointcut-ref="myPointCut" returning="ret" />
             
        <!-- 3.3 最终通知 -->        
        <aop:after method="myAfter" pointcut-ref="myPointCut"/>  
             
        </aop:aspect>
    </aop:config>
</beans>
```
5. 测试
```
@Test
public void testAop(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserService useService = (UserService) context.getBean("userService");
    useService.addUser();
    useService.deleteUser();
}
```
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916114837610-277393711.png)

#### 基于注解实现
1. 注解配置bean
```
<!--1、创建目标类 -->
<bean id="userService" class="com.ys.aop.UserServiceImpl"></bean>  
<!--2、创建切面类（通知）  -->
<bean id="myAspect" class="com.ys.aop.MyAspect"></bean>
```
2. 注解配置
```
目标类：
@Service("userService")
public class UserServiceImpl implements UserService{
    
}

切面类：
@Component("myAspect")
public class MyAspect{
    
}
```
3. 配置扫描注解识别
```
<!-- 配置扫描注解类
        base-package:表示含有注解类的包名。
        如果扫描多个包，则下面的代码书写多行，改变 base-package 里面的内容即可！
    -->
    <context:component-scan base-package="com.ys.aop"></context:component-scan>
```
4. 注解配置 AOP
一. 我们用xml配置如下
```
<aop:comfig>
<aop:aspect ref="myAspect">
```
- 这是告诉Spring哪个是切面类，配置注解如下
```
@Component("myAspect")
@Aspect
public class MyAspect{
    
}
```
二. 如何让 Spring 认识我们所配置的 AOP 注解呢？光有前面的类注解扫描是不够的，这里我们要额外配置 AOP 注解识别。
```
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
三. 注解配置前置通知
- 我们先看 xml 配置前置通知如下：
```
<!-- 切入点表达式 -->
        <aop:pointcut expression="execution(* com.ys.aop.*.*(..))" id="myPointCut"/>
        <!-- 3.1 前置通知
                <aop:before method="" pointcut="" pointcut-ref=""/>
                    method : 通知，及方法名
                    pointcut :切入点表达式，此表达式只能当前通知使用。
                    pointcut-ref ： 切入点引用，可以与其他通知共享切入点。
                通知方法格式：public void myBefore(JoinPoint joinPoint){
                    参数1：org.aspectj.lang.JoinPoint  用于描述连接点（目标方法），获得目标方法名等
        -->
        <aop:before method="myBefore" pointcut-ref="myPointCut"/>
```
- 那么注解的方式如下：
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916130707688-151711705.png)

四. 注解配置后置通知
- xml配置后置通知
```
<!-- 3.2后置通知  ,目标方法后执行，获得返回值
                <aop:after-returning method="" pointcut-ref="" returning=""/>
                    returning 通知方法第二个参数的名称
                通知方法格式：public void myAfterReturning(JoinPoint joinPoint,Object ret){
                    参数1：连接点描述
                    参数2：类型Object，参数名 returning="ret" 配置的
        -->
        <aop:after-returning method="myAfterReturning" pointcut-ref="myPointCut" returning="ret" />
```
- 　注意看，后置通知有个 returning="ret" 配置，这是用来获得目标方法的返回值的。
- 注解配置如下：
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916131117375-1818967074.png)

5. 测试
```
@Test
    public void testAopAnnotation(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext_Annotation.xml");
        UserService useService = (UserService) context.getBean("userService");
        useService.addUser();
        useService.deleteUser();
    }
```
6. 打印
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916131332141-1932607836.png)


## 注解改进

- 　我们可以看前置通知和后置通知的注解配置
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916131527922-706373475.png)

- 注意看红色框住的部分，很显然这里是重复的，而且如果我们有多个通知方法，那就得在每个方法名都写上该注解，而且如果包名够复杂，也很容易写错。那么怎么办呢？

- 解决办法就是声明公共切入点：
1. 在 切面类 MyAspect.java 中新增一个切入点方法 myPointCut()，然后在这个方法上添加 @Pointcut 注解

![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916131909516-1956602330.png)

2. 那么前置通知和后置通知，我们可以进行如下改写配置

![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170916132025407-707464879.png)

## 事务介绍
- 事务（Transaction），一般是指要做的或所做的事情。在计算机术语中是指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)。

- 事务的四个特性（ACID）
1. 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
2. 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
3. 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
4. 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中


#### 事务管理器
①、TransactionStatus getTransaction(TransactionDefinition definition) ，事务管理器 通过TransactionDefinition，获得“事务状态”，从而管理事务。  
②、void commit(TransactionStatus status)  根据状态提交  
③、void rollback(TransactionStatus status) 根据状态回滚

#### 基本事务属性的定义

- 那么什么是事务属性呢？事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面，如图所示

![image](https://images2017.cnblogs.com/blog/1120165/201710/1120165-20171003121421083-444531549.png)

一. 传播行为：当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。

> ①、PROPAGATION_REQUIRED ：required , 必须。默认值，A如果有事务，B将使用该事务；如果A没有事务，B将创建一个新的事务  

> ②、PROPAGATION_SUPPORTS：supports ，支持。A如果有事务，B将使用该事务；如果A没有事务，B将以非事务执行。

> ③、PROPAGATION_MANDATORY：mandatory ，强制。A如果有事务，B将使用该事务；如果A没有事务，B将抛异常。

> ④、PROPAGATION_REQUIRES_NEW ：requires_new，必须新的。如果A有事务，将A的事务挂起，B创建一个新的事务；如果A没有事务，B创建一个新的事务。

> ⑤、PROPAGATION_NOT_SUPPORTED ：not_supported ,不支持。如果A有事务，将A的事务挂起，B将以非事务执行；如果A没有事务，B将以非事务执行。  

> ⑥、PROPAGATION_NEVER ：never，从不。如果A有事务，B将抛异常；如果A没有事务，B将以非事务执行。

> ⑦、PROPAGATION_NESTED ：nested ，嵌套。A和B底层采用保存点机制，形成嵌套事务


二. 隔离级别：定义了一个事务可能受其他并发事务影响的程度。

- 并发事务引起的问题：
> 在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务。并发虽然是必须的，但可能会导致以下的问题。

> ①、脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。

>  ②、不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。  

> ③、幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

- 注意：不可重复读重点是修改，而幻读重点是新增或删除。

> 在 Spring 事务管理中，为我们定义了如下的隔离级别：


> ①、ISOLATION_DEFAULT：使用后端数据库默认的隔离级别

> ②、ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读

> ③、ISOLATION_READ_COMMITTED：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

> ④、ISOLATION_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生

> ⑤、ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的

三. 只读
> 这是事务的第三个特性，是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施

四. 事务超时
> 为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

五. 回滚规则
> 事务五边形的最后一个方面是一组规则，这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的） 。但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

### Spring编程式事务和声明实事务

- 编程式事务处理：所谓编程式事务指的是通过编码方式实现事务，允许用户在代码中精确定义事务的边界。即类似于JDBC编程实现事务管理。管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
- 声明式事务处理：管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。


#### 编程式事务
第一步： 编写 Dao 层　AccountDao 接口：
```
package com.ys.dao;
 
public interface AccountDao {
    /**
     * 汇款
     * @param outer 汇款人
     * @param money 汇款金额
     */
    public void out(String outer,int money);
     
    /**
     * 收款
     * @param inner 收款人
     * @param money 收款金额
     */
    public void in(String inner,int money);
 
}
```
AccountDaoImpl 接口实现类

```
 
import org.springframework.jdbc.core.support.JdbcDaoSupport;
 
import com.ys.dao.AccountDao;
 
public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao {
 
    /**
     * 根据用户名减少账户金额
     */
    @Override
    public void out(String outer, int money) {
        this.getJdbcTemplate().update("update account set money = money - ? where username = ?",money,outer);
    }
 
    /**
     * 根据用户名增加账户金额
     */
    @Override
    public void in(String inner, int money) {
        this.getJdbcTemplate().update("update account set money = money + ? where username = ?",money,inner);
    }
 
}
```
第二步： 实现 Service 层　AccountService 接口
```
package com.ys.service;
 
public interface AccountService {
     
    /**
     * 转账
     * @param outer 汇款人
     * @param inner 收款人
     * @param money 交易金额
     */
    public void transfer(String outer,String inner,int money);
 
}
```
```
package com.ys.service.impl;
 
import com.ys.dao.AccountDao;
import com.ys.service.AccountService;
 
public class AccountServiceImpl implements AccountService{
 
    private AccountDao accountDao;
     
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
    @Override
    public void transfer(String outer, String inner, int money) {
        accountDao.out(outer, money);
        accountDao.in(inner, money);
    }
 
}
```
第三部：Spring 全局配置文件 applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">   
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
     
    <bean id="accountDao" class="com.ys.dao.impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
     
    <bean id="accountService" class="com.ys.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>
</beans>
```

第五步
```
public class TransactionTest {
     
    @Test
    public void testNoTransaction(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        AccountService account = (AccountService) context.getBean("accountService");
        //Tom 向 Marry 转账1000
        account.transfer("Tom", "Marry", 1000);
    }
 
}
```

#### 
> 上面转账的两步操作中间发生了异常，但是第一步依然在数据库中进行了增加操作。实际应用中不会允许这样的情况发生，所以我们这里用事务来进行管理。

>　　Dao 层不变，我们在 Service 层注入 TransactionTemplate 模板，因为是用模板来管理事务，所以模板需要注入事务管理器  DataSourceTransactionManager 。而事务管理器说到底还是用底层的JDBC在管理，所以我们需要在事务管理器中注入 DataSource。这几个步骤分别如下：

```
package com.ys.service.impl;
 
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;
 
import com.ys.dao.AccountDao;
import com.ys.service.AccountService;
 
public class AccountServiceImpl implements AccountService{
 
    private AccountDao accountDao;
    private TransactionTemplate transactionTemplate;
     
    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
    @Override
    public void transfer(final String outer,final String inner,final int money) {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus arg0) {
                accountDao.out(outer, money);
                //int i = 1/0;
                accountDao.in(inner, money);
            }
        });
    }
 
}
```
spring全局配置文件applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">   
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
     
    <bean id="accountDao" class="com.ys.dao.impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
     
    <bean id="accountService" class="com.ys.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
        <property name="transactionTemplate" ref="transactionTemplate"></property>
    </bean>
     
    <!-- 创建模板 -->
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="txManager"></property>
    </bean>
     
    <!-- 配置事务管理器 ,管理器需要事务，事务从Connection获得，连接从连接池DataSource获得 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>
```

```
//定义一个某个框架平台的TransactionManager，如JDBC、Hibernate
        DataSourceTransactionManager dataSourceTransactionManager =
                new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(this.getJdbcTemplate().getDataSource()); // 设置数据源
     
        DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); // 定义事务属性
        transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); // 设置传播行为属性
        TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef); // 获得事务状态
        try {
            // 数据库操作
            accountDao.out(outer, money);
            int i = 1/0;
            accountDao.in(inner, money);
             
            dataSourceTransactionManager.commit(status);// 提交
        } catch (Exception e) {
            dataSourceTransactionManager.rollback(status);// 回滚
        }
```

- 声明式事务处理(基于xml)

> Dao 层和 Service   层与我们最先开始的不用事务实现转账保持不变。主要是  applicationContext.xml 文件变化了。我们在 applicationContext.xml 文件中配置 aop 自动生成代理，进行事务管理：

　　①、配置管理器

　　②、配置事务详情

　　③、配置 aop
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">
                             
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
     
    <bean id="accountDao" class="com.ys.dao.impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
     
    <bean id="accountService" class="com.ys.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>
     
    <!-- 1 事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
     
    <!-- 2 事务详情（事务通知）  ， 在aop筛选基础上，比如对ABC三个确定使用什么样的事务。例如：AC读写、B只读 等
        <tx:attributes> 用于配置事务详情（属性属性）
            <tx:method name=""/> 详情具体配置
                propagation 传播行为 ， REQUIRED：必须；REQUIRES_NEW:必须是新的
                isolation 隔离级别
    -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="transfer" propagation="REQUIRED" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>
     
    <!-- 3 AOP编程，利用切入点表达式从目标类方法中 确定增强的连接器，从而获得切入点 -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.ys.service..*.*(..))"/>
    </aop:config>
</beans>
```
#### 声明式事务处理（基于AOP注解）
分为如下两步：

　　①、在applicationContext.xml 配置事务管理器，将并事务管理器交予spring

　　②、在目标类或目标方法添加注解即可 @Transactional

　　首先在 applicationContext.xml 文件中配置如下：
　　
```
<!-- 1 事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 2 将管理器交予spring
        * transaction-manager 配置事务管理器
        * proxy-target-class
            true ： 底层强制使用cglib 代理
    -->
    <tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>
```
其次在目标类或者方法添加注解@Transactional。如果在类上添加，则说明类中的所有方法都添加事务，如果在方法上添加，则只有该方法具有事务。
```
package com.ys.service.impl;
 
import javax.annotation.Resource;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
 
import com.ys.dao.AccountDao;
import com.ys.service.AccountService;
 
@Transactional(propagation=Propagation.REQUIRED , isolation = Isolation.DEFAULT)
@Service("accountService")
public class AccountServiceImpl implements AccountService{
    @Resource(name="accountDao")
    private AccountDao accountDao;
     
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
    @Override
    public void transfer(String outer, String inner, int money) {
        accountDao.out(outer, money);
        //int i = 1/0;
        accountDao.in(inner, money);
    }
 
}
```