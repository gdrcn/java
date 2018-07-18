### AOP是什么
- AOP（Aspect Oriented Programming），通常称为面向切面编程。它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

传统实现方法：（对User实现增加和删除）
1. 创建UserService接口
2. UserService的实现类
3. 创建事务类MyTransaction
4. 创建代理类ProyUser
```
@Test 
public void testOne(){
    MyTransaction transaction = new MyTransaction();
    UserService userService = new UserService();
    //产生静态代理对象
    ProxyUser proxy = new ProxyUser(userService, transaction);
    proxy.addUser(null);
    proxy.deleteUser(0);
}
```

#### 解决方法
- 动态代理就不要自己手动生成代理类了，我们去掉 ProxyUser.java 类，增加一个 ObjectInterceptor.java 类
```
package com.ys.aop.two;
 
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
 
import com.ys.aop.one.MyTransaction;
 
public class ObjectInterceptor implements InvocationHandler{
    //目标类
    private Object target;
    //切面类（这里指事务类）
    private MyTransaction transaction;
     
    //通过构造器赋值
    public ObjectInterceptor(Object target,MyTransaction transaction){
        this.target = target;
        this.transaction = transaction;
    }
     
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        //开启事务
        this.transaction.before();
        //调用目标类方法
        method.invoke(this.target, args);
        //提交事务
        this.transaction.after();
        return null;
    }
     
}
```
测试
```
@Test
    public void testOne(){
        //目标类
        Object target = new UserServiceImpl();
        //事务类
        MyTransaction transaction = new MyTransaction();
        ObjectInterceptor proxyObject = new ObjectInterceptor(target, transaction);
        /**
         * 三个参数的含义：
         * 1、目标类的类加载器
         * 2、目标类所有实现的接口
         * 3、拦截器
         */
        UserService userService = (UserService) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), proxyObject);
        userService.addUser(null);
    }
```

### AOP
　　1. target：目标类，需要被代理的类。例如：UserService

　　2. Joinpoint(连接点):所谓连接点是指那些可能被拦截到的方法。例如：所有的方法
 
　　3. PointCut 切入点：已经被增强的连接点。例如：addUser()

　　4. advice 通知/增强，增强代码。例如：after、before

　　5.  Weaving(织入):是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.

　　6. proxy 代理类：通知+切入点

　　7. Aspect(切面): 是切入点pointcut和通知advice的结合
　　
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170906221200444-1388566125.png)

- Spring按照通知Advice在目标类方法的连接点位置，可以分为5类

- [x] 前置通知 org.springframework.aop.MethodBeforeAdvice
- [ ] 在目标方法执行前实施增强，比如上面例子的 before()方法
- [x] 后置通知 org.springframework.aop.AfterReturningAdvice
- [ ] 在目标方法执行后实施增强，比如上面例子的 after()方法
- [x] 环绕通知 org.aopalliance.intercept.MethodInterceptor
- [ ] 在目标方法执行前后实施增强
- [x] 异常抛出通知 org.springframework.aop.ThrowsAdvice
- [ ] 在方法抛出异常后实施增强
- [x] 引介通知 org.springframework.aop.IntroductionInterceptor
- [ ] 　　　　　　在目标类中添加一些新的方法和属性

#### 使用AOP解决上面的需求
1. 在Spring的配置文件applicationContext.xml进行如下配置

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--1、 创建目标类 -->
    <bean id="userService" class="com.ys.aop.UserServiceImpl"></bean>  
    <!--2、创建切面类（通知）  -->
    <bean id="transaction" class="com.ys.aop.one.MyTransaction"></bean>
     
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
        <!-- 切入点表达式 -->
        <aop:pointcut expression="execution(* com.ys.aop.*.*(..))" id="myPointCut"/>
        <aop:aspect ref="transaction">
            <!-- 配置前置通知，注意 method 的值要和 对应切面的类方法名称相同 -->
            <aop:before method="before" pointcut-ref="myPointCut"></aop:before>
            <aop:after-returning method="after" pointcut-ref="myPointCut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

2. 测试
```
@Test
    public void testAop(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService useService = (UserService) context.getBean("userService");
        useService.addUser(null);
    }
```

```
execution（modifiers-pattern? ref-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?）
　　　　　　　　　　   类修饰符           返回值           方法所在的包                  方法名                     方法抛出的异常
```

  
   ++springAOP 的具体加载步骤：++

　　1、当 spring 容器启动的时候，加载了 spring 的配置文件

　　2、为配置文件中的所有 bean 创建对象

　　3、spring 容器会解析 aop:config 的配置

   　　　　1、解析切入点表达式，用切入点表达式和纳入 spring 容器中的 bean 做匹配

       　　　　如果匹配成功，则会为该 bean 创建代理对象，代理对象的方法=目标方法+通知

       　　　　如果匹配不成功，不会创建代理对象

　　4、在客户端利用 context.getBean() 获取对象时，如果该对象有代理对象，则返回代理对象；如果没有，则返回目标对象

　　　　说明：如果目标类没有实现接口，则 spring 容器会采用 cglib 的方式产生代理对象，如果实现了接口，则会采用 jdk 的方式
　　　　

#### AspectJ实现AOP

- 什么是AspectJ?
> AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法，也可以说 AspectJ 是一个基于 Java 语言的 AOP 框架。通常我们在使用 Spring AOP 的时候，都会导入 AspectJ 的相关 jar 包

- 切入表达式
```
<!-- 切入点表达式 -->
<aop:pointcut expression="execution(* com.ys.aop.*.*(..))" id="myPointCut"/>
```

![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170911231547860-1268274701.png)

- 注意：如果切入点表达式有多个不同目录呢？ 可以通过 || 来表示或的关系。
```
<aop:pointcut expression="execution(* com.ys.*Service1.*(..)) ||
                          execution(* com.ys.*Service2.*(..))" id="myPointCut"/>
```

> 1、execution:匹配方法的执行（常用）execution(public *.*(..))
  
  
> 2.within:匹配包或子包中的方法(了解)within(com.ys.aop..*)  

> 3.this:匹配实现接口的代理对象中的方法(了解)this(com.ys.aop.user.UserDAO)  

> 4.target:匹配实现接口的目标对象中的方法(了解)target(com.ys.aop.user.UserDAO)  

> 5.args:匹配参数格式符合标准的方法(了解)args(int,int)
  
> 6.bean(id)  对指定的bean所有的方法(了解)
    bean('userServiceId')

- Aspect 通知类型
```
    before:前置通知(应用：各种校验)
    在方法执行前执行，如果通知抛出异常，阻止方法运行
afterReturning:后置通知(应用：常规数据处理)
    方法正常返回后执行，如果方法中抛出异常，通知无法执行
    必须在方法执行后才执行，所以可以获得方法的返回值。
around:环绕通知(应用：十分强大，可以做任何事情)
    方法执行前后分别执行，可以阻止方法的执行
    必须手动执行目标方法
afterThrowing:抛出异常通知(应用：包装异常信息)
    方法抛出异常后执行，如果方法没有抛出异常，无法执行
after:最终通知(应用：清理现场)
    方法执行完毕后执行，无论方法中是否出现异常
```

#### AOP实例
① 创建接口
```
package com.ys.aop;
 
public interface UserService {
    //添加 user
    public void addUser();
    //删除 user
    public void deleteUser();
}
```
② 创建实现类
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

🌂 创建切面类（包含各种通知）
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
     
    public void myAfter(){
        System.out.println("最终通知");
    }
 
}
```

④. 创建spring配置文件applicationContext.xml

　　我们首先测试前置通知、后置通知、最终通知
```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--1、 创建目标类 -->
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
🌫 测试
```
@Test
    public void testAop(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService useService = (UserService) context.getBean("userService");
        useService.addUser();
    }
```

![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170912075824688-1165099409.png)

- 注意，后置通知的返回值为 null，是因为我们的目标方法 addUser() 没有返回值。如果有返回值，这里就是addUser() 的返回值


### 测试异常通知

1. 目标接口保持不变，目标类我们手动引入异常：
```
public void addUser() {
        int i = 1/0;//显然这里会抛出除数不能为 0
        System.out.println("增加 User");
    }
```
2. 接着配置切面：MyAspect.java
```
public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
        System.out.println("抛出异常通知 ： " + e.getMessage());
    }
```
3. 接着在 applicationContext.xml 中配置如下
```
<!-- 3.4 抛出异常
                <aop:after-throwing method="" pointcut-ref="" throwing=""/>
                    throwing ：通知方法的第二个参数名称
                通知方法格式：public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
                    参数1：连接点描述对象
                    参数2：获得异常信息，类型Throwable ，参数名由throwing="e" 配置
        -->
        <aop:after-throwing method="myAfterThrowing" pointcut-ref="myPointCut" throwing="e"/>
```

4. 　测试：
```
@Test
    public void testAop(){
        String str = "com/ys/execption/applicationContext.xml";
        ApplicationContext context = new ClassPathXmlApplicationContext(str);
        UserService useService = (UserService) context.getBean("userService");
        useService.addUser();
    }
```
![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170912081334516-37060139.png)

#### 环绕通知
1. 目标接口和目标类保持不变，切面MyAspect 修改如下
```
public class MyAspect {
     
    public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("前置通知");
        //手动执行目标方法
        Object obj = joinPoint.proceed();
         
        System.out.println("后置通知");
        return obj;
    }
 
}
```

2. applicationContext.xml 配置如下：
```
<!-- 环绕通知
                <aop:around method="" pointcut-ref=""/>
                通知方法格式：public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable{
                    返回值类型：Object
                    方法名：任意
                    参数：org.aspectj.lang.ProceedingJoinPoint
                    抛出异常
                执行目标方法：Object obj = joinPoint.proceed();
        -->
        <aop:around method="myAround" pointcut-ref="myPointCut"/>
```

3. 测试
```
@Test
    public void testAop(){
        String str = "com/ys/around/applicationContext.xml";
        ApplicationContext context = new ClassPathXmlApplicationContext(str);
        UserService useService = (UserService) context.getBean("userService");
        useService.addUser();
    }
```

![image](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170912082012516-659289478.png)

