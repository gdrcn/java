### SSSM三大框架整合
- 整合思路
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170821224348183-1245812487.png)

> ①、表现层，也就是 Controller,由 SpringMVC 来控制，而SpringMVC 是Spring 的一个模块，故不需要整合。

> ②、业务层，也就是 service，通常由 Spring 来管理 service 接口，我们会使用 xml 配置的方式来将 service 接口配置到 spring 配置文件中。而且事务控制一般也是在 service 层进行配置。

> ③、持久层，也就是 dao 层，而且包括实体类，由 MyBatis 来管理，通过 spring 来管理 mapper 接口，使用mapper的扫描器自动扫描mapper接口在spring中进行注册。

- 准备环境
1. 目录结构

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823090137043-1707039799.png)

2. 整合Dao层

①、在 db.properties 文件中，保存数据库连接的基本信息
```
#db.properties
dataSource=org.apache.commons.dbcp.BasicDataSource
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/ssm
username=root
password=root
```
3. mybatis全局配置文件 mybatis-configuration.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 全局 setting 配置，根据需要添加  -->
    <!--开启二级缓存  -->
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
     
    <!-- 配置别名 -->
    <typeAliases>
        <!-- 批量扫描别名 -->
        <package name="com.ys.po"/>
    </typeAliases>
     
    <!-- 配置mapper,由于使用 spring 和mybatis 的整合包进行 mapper 扫描，这里不需要配置了
        必须遵循：mapper.xml 和 mapper.java 文件同名且在同一个目录下
     -->
     <!-- <mappers>
     </mappers> -->
     
</configuration>
```
- 通过 mapper 接口来加载映射文件,必须满足下面四点
>　1、xxxMapper 接口必须要和 xxxMapper.xml 文件同名且在同一个包下，也就是说 UserMapper.xml 文件中的namespace是UserMapper接口的全类名

>　　2、xxxMapper接口中的方法名和 xxxMapper.xml 文件中定义的 id 一致

>　　3、xxxMapper接口输入参数类型要和 xxxMapper.xml 中定义的 parameterType 一致

>　　4、xxxMapper接口返回数据类型要和 xxxMapper.xml 中定义的 resultType 一致 

4. 配置 Spring 文件
- 我们需要配置数据源、SqlSessionFactory以及mapper扫描器，由于这是对 Dao 层的整合，后面还有对于 业务层，表现层等的整合，为了使条目更加清新，我们新建 config/spring 文件夹，这里将配置文件取名为 spring-dao.xml 放入其中


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">
 
    <!--第一步： 配置数据源 -->
    <!-- 加载db.properties文件中的内容,db.properties文件中的key名要有一定的特殊性 -->
    <context:property-placeholder location="classpath:db.properties" />
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="maxActive" value="30"></property>
        <property name="maxIdle" value="5"></property>
    </bean>
     
    <!-- 第二步：创建sqlSessionFactory。生产sqlSession  -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据库连接池 -->
        <property name="dataSource" ref="dataSource"></property>
        <!-- 加载mybatis全局配置文件,注意这个文件的目录 -->
        <property name="configLocation" value="classpath:mybatis/mybatis-configuration.xml"></property>
    </bean>
     
    <!-- 第三步：配置 mapper 扫描器
        * 接口类名和映射文件必须同名
        * 接口类和映射文件必须在同一个目录下
        * 映射文件namespace名字必须是接口的全类路径名
        * 接口的方法名必须和映射Statement的id一致
    -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 扫描的包路径，如果需要扫描多个包，中间使用逗号分隔 -->
        <property name="basePackage" value="com.ys.mapper"></property>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>
     
</beans>
```
5. 根据逆向工程生成 po 类以及 mapper 文件
6. 整合 service  

①、定义 service 接口
```
package com.ys.service.impl;
 
import com.ys.po.User;
 
public interface IUserService {
     
    //通过用户名和密码查询User
    public User selectUserByUsernameAndPassword(User user);
 
}
```
②、编写 service 实现类

```
package com.ys.service;
 
import org.springframework.beans.factory.annotation.Autowired;
 
import com.ys.mapper.UserMapper;
import com.ys.po.User;
import com.ys.service.impl.IUserService;
 
public class UserServiceImpl implements IUserService{
 
    @Autowired
    private UserMapper userMapper; //通过@Autowired向spring容器注入UserMapper
     
    //通过用户名和密码查询User
    @Override
    public User selectUserByUsernameAndPassword(User user) {
        User u = userMapper.selectUserByUsernameAndPassword(user);
        return u;
    }
 
}
```
通过@Autowired向spring容器注入UserMapper，它会通过spring配的扫描器扫描到，并将对象装载到spring容器中。

③、在spring容器中配置 Service 接口，这里我们使用 xml 的方式
   
   在 config/spring 目录下，新建 spring-service.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">
 
    <!--配置UserServiceImpl -->
    <bean id="userService" class="com.ys.service.UserServiceImpl"></bean>
     
</beans>
```
④、在spring容器中配置 事务处理  
在 config/spring 目录下，新建 spring-transaction.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">
 
    <!-- 事务管理器 -->
    <!-- 对mybatis操作数据事务控制，spring使用jdbc的事务控制类 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <!-- 数据源dataSource在spring-dao.xml中配置了 -->
         <property name="dataSource" ref="dataSource"/>
    </bean>
 
    <!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
         <tx:attributes>
             <tx:method name="save*" propagation="REQUIRED"/>
             <tx:method name="delete*" propagation="REQUIRED"/>
             <tx:method name="update*" propagation="REQUIRED"/>
             <tx:method name="insert*" propagation="REQUIRED"/>
             <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
             <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
             <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
         </tx:attributes>
    </tx:advice>
     
    <aop:config>
         <!-- com.ys.service.impl包里面的所有类，所有方法，任何参数 -->
         <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.ys.service.impl.*.*(..))"/>
    </aop:config>    
</beans>
```

7. 整合 SpringMVC
①、配置前端控制器    
在 web.xml 文件中添加如下代码
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://java.sun.com/xml/ns/javaee"
     xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>SpringMVC_01</display-name>
  <!-- 配置前端控制器DispatcherServlet -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--springmvc.xml 是自己创建的SpringMVC全局配置文件，用contextConfigLocation作为参数名来加载
        如果不配置 contextConfigLocation，那么默认加载的是/WEB-INF/servlet名称-servlet.xml，在这里也就是 springmvc-servlet.xml
      -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spirng/springmvc.xml</param-value>
    </init-param>
  </servlet>
 
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--第一种配置：*.do,还可以写*.action等等，表示以.do结尾的或者以.action结尾的URL都由前端控制器DispatcherServlet来解析
        第二种配置：/,所有访问的 URL 都由DispatcherServlet来解析，但是这里最好配置静态文件不由DispatcherServlet来解析
        错误配置：/*,注意这里是不能这样配置的，应为如果这样写，最后转发到 jsp 页面的时候，仍然会由DispatcherServlet进行解析，
                    而这时候会找不到对应的Handler，从而报错！！！
      -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

②、配置处理器映射器、处理器适配器、视图解析器  
在 config/spring 目录下新建 springmvc.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
 
    <!--使用mvc:annotation-driven可以代替上面的映射器和适配器
        这里面会默认加载很多参数绑定方法，比如json转换解析器就默认加载，所以优先使用下面的配置
      -->
    <mvc:annotation-driven></mvc:annotation-driven>
 
     
    <!--批量配置Handler,指定扫描的包全称  -->
    <context:component-scan base-package="com.ys.controller"></context:component-scan>
     
 
    <!--配置视图解析器  -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         
    </bean>
</beans>
```

③、编写 Handler，也就是 Controller

在 com.ys.controller 包下新建 UserController.java 文件
package com.ys.controller;
``` 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
 
import com.ys.po.User;
import com.ys.service.impl.IUserService;
 
@Controller
public class UserController {
    @Autowired
    public IUserService userService;
     
    @RequestMapping("/login")
    public ModelAndView login(User user){
        ModelAndView mv = new ModelAndView();
        User u = userService.selectUserByUsernameAndPassword(user);
        //根据用户名和密码查询user，如果存在，则跳转到 success.jsp 页面
        if(u != null){
            mv.addObject("username", u.getUsername());
            mv.addObject("user", u);
            mv.setViewName("view/success.jsp");
        }else{
            //如果不存在，则跳转到 login.jsp页面重新登录
            return new ModelAndView("redirect:/login.jsp");
        }
        return mv;
    }
 
}
```
④、加载 Spring 容器
我们在 classpath/spring 目录下新建了 spring-dao.xml,spring-service.xml,spring-transaction.xml 这些文件，里面有我们配置的 mapper，controller，service，那么如何将这些加载到 spring 容器中呢？

　　在 web.xml 文件中添加如下代码：
```
<!-- 加载spring容器 -->
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>classpath:spring/spring-*.xml</param-value>
</context-param>
<listener>
   <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
由于配置文件比较多，我们使用通配符加载的方式。注意：这段代码最好要加在前端控制器的前面。

　　至此 SSM 三大框架整合就完成了，接下来我们进行测试。
　　

### SpringMVC参数绑定
- 在 SpringMVC 中，提交请求的数据是通过方法形参来接收的。从客户端请求的 key/value 数据，经过参数绑定，将 key/value 数据绑定到 Controller 的形参上，然后在 Controller 就可以直接使用该形参。

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170824221830496-1908819758.png)


- 默认支持的类型
> SpringMVC 有支持的默认参数类型，我们直接在形参上给出这些默认类型的声明，就能直接使用了。如下：

>　　①、HttpServletRequest 对象

>　　②、HttpServletResponse 对象

>　　③、HttpSession 对象

>　　④、Model/ModelMap 对象　

Controller 代码： 

```
@RequestMapping("/defaultParameter")
    public ModelAndView defaultParameter(HttpServletRequest request,HttpServletResponse response,
                            HttpSession session,Model model,ModelMap modelMap) throws Exception{
        request.setAttribute("requestParameter", "request类型");
        response.getWriter().write("response");
        session.setAttribute("sessionParameter", "session类型");
        //ModelMap是Model接口的一个实现类，作用是将Model数据填充到request域
        //即使使用Model接口，其内部绑定还是由ModelMap来实现
        model.addAttribute("modelParameter", "model类型");
        modelMap.addAttribute("modelMapParameter", "modelMap类型");
         
        ModelAndView mv = new ModelAndView();
        mv.setViewName("view/success.jsp");
        return mv;
    }
```

```
<body>
    request:${requestParameter}
    session:${sessionParameter}
    model:${modelParameter}
    modelMap:${modelMapParameter}
</body>
```

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170824225355464-196324090.png)

### 基本数据类型参数绑定
一、byte，占用一个字节，取值范围为 -128-127，默认是“\u0000”，表示空

二、short，占用两个字节，取值范围为 -32768-32767

三、int，占用四个字节，-2147483648-2147483647

四、long，占用八个字节，对 long 型变量赋值时必须加上"L"或“l”,否则不认为是 long 型

五、float，占用四个字节，对 float 型进行赋值的时候必须加上“F”或“f”，如果不加，会产生编译错误，因为系统
自动将其定义为 double 型变量。double转换为float类型数据会损失精度。float a = 12.23产生编译错误的，float a = 12是正确的

六、double，占用八个字节，对 double 型变量赋值的时候最好加上“D”或“d”，但加不加不是硬性规定

七、char,占用两个字节，在定义字符型变量时，要用单引号括起来

八、boolean，只有两个值“true”和“false”，默认值为false，不能用0或非0来代替，这点和C语言不同

Jsp页面代码
```
<form action="basicData" method="post">
    <input name="username" value="10" type="text"/>
    <input type="submit" value="提交">
</form>
```
Controller 代码
```
@RequestMapping("/basicData")
    public void basicData(int username){
        System.out.println(username);//10
    }
```
结果是 打印出了表单里面的 value 的值。


- 注意：表单中input的name值和Controller的参数变量名保持一致，就能完成数据绑定。那么如果不一致呢？我们使用 @RequestParam 注解来完成，如下：
JSP页面代码不变，<input name="username">保持原样，Controller 代码如下
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170824232611824-803624054.png) 

- 问题：我们这里的参数是基本数据类型，如果从前台页面传递的值为 null 或者 “”的话，那么会出现数据转换的异常，就是必须保证表单传递过来的数据不能为null或”"，所以，在开发过程中，对可能为空的数据，最好将参数数据类型定义成包装类型，具体参见下面的例子。


#### 包装数据类型的绑定
- 包装类型如Integer、Long、Byte、Double、Float、Short，（String 类型在这也是适用的）这里我们以 Integer 为例

　　Controller 代码为：
　![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170824233243339-1897999963.png)
　

#### POJO（实体类）类型的绑定
User.java
```
package com.ys.po;
 
import java.util.Date;
 
public class User {
    private Integer id;
 
    private String username;
 
    private String sex;
 
    private Date birthday;
 
    public Integer getId() {
        return id;
    }
 
    public void setId(Integer id) {
        this.id = id;
    }
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username == null ? null : username.trim();
    }
 
    public String getSex() {
        return sex;
    }
 
    public void setSex(String sex) {
        this.sex = sex == null ? null : sex.trim();
    }
 
    public Date getBirthday() {
        return birthday;
    }
 
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```
JSP页面：注意输入框的 name 属性值和上面 POJO 实体类的属性保持一致即可映射成功
```
<form action="pojo" method="post">
        用户id:<input type="text" name="id" value="2"></br>
        用户名:<input type="text" name="username" value="Marry"></br>
        性别:<input type="text" name="sex" value="女"></br>
        出生日期:<input type="text" name="birthday" value="2017-08-25"></br>
        <input type="submit" value="提交">
    </form>
```
Controller ：
```
@RequestMapping("/pojo")
    public void pojo(User user){
        System.out.println(user);
    }
```
我们在上面代码打个断点，然后输入URL，进入到这个Controller中：
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170825235027636-310799695.png)

上面是报错了，User.java 的birthday 属性是 Date 类型的，而我们输入的是字符串类型，故绑定不了

 　　那么问题来了，Date 类型的数据绑定失败，如何解决这样的问题呢？这就是我们前面所说的需要自定义Date类型的转换器。
  
①、定义由String类型到 Date 类型的转换器
```
package com.ys.util;
 
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
 
import org.springframework.core.convert.converter.Converter;
 
//需要实现Converter接口，这里是将String类型转换成Date类型
public class DateConverter implements Converter<String, Date> {
 
    @Override
    public Date convert(String source) {
        //实现将字符串转成日期类型(格式是yyyy-MM-dd HH:mm:ss)
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            return dateFormat.parse(source);
        } catch (ParseException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //如果参数绑定失败返回null
        return null;
    }
 
}
```
②、在 springmvc.xml 文件中配置转换器
```
<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
     
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <!-- 自定义转换器的类名 -->
            <bean class="com.ys.util.DateConverter"></bean>
        </property>
    </bean>
```

输入 URL，再次查看Controller的形参
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170825234631371-2087963063.png)

#### 复合POJO（实体类）类型的绑定
这里我们增加一个实体类，ContactInfo.java
```
package com.ys.po;
 
public class ContactInfo {
    private Integer id;
 
    private String tel;
 
    private String address;
 
    public Integer getId() {
        return id;
    }
 
    public void setId(Integer id) {
        this.id = id;
    }
 
    public String getTel() {
        return tel;
    }
 
    public void setTel(String tel) {
        this.tel = tel == null ? null : tel.trim();
    }
 
    public String getAddress() {
        return address;
    }
 
    public void setAddress(String address) {
        this.address = address == null ? null : address.trim();
    }
}
```
然后在上面的User.java中增加一个属性 private ContactInfo contactInfo

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170827233150918-1977663722.png)

JSP 页面：注意属性name的命名，User.java 的复合属性名.字段名

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170827233935011-1374294292.png)

Controller![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170827234141152-1561771765.png)
User对象中有ContactInfo属性，但是，在表单代码中，需要使用“属性名(对象类型的属性).属性名”来命名input的name

#### 数组类型的绑定

- 需求：我们查询出所有User 的信息，并且在JSP页面遍历显示，这时候点击提交按钮，需要在 Controller 中获得页面中显示 User 类的 id 的所有值的数组集合。

JSP 页面：注意用户id的name值定义为 userId

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828001019668-1193860589.png)

Controller.java
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828001148636-76966636.png)

#### List类型的绑定

需求：批量修改 User 用户的信息

第一步：创建 UserVo.java，封装 List<User> 属性
```
package com.ys.po;
 
import java.util.List;
 
public class UserVo {
     
    private List<User> userList;
    public List<User> getUserList() {
        return userList;
    }
    public void setUserList(List<User> userList) {
        this.userList = userList;
    }
}
```
第二步：为了简化过程，我们直接从 Controller 中查询所有 User 信息，然后在页面显示
Controller
```
@RequestMapping("selectAllUserAndList")
    public ModelAndView selectAllUserAndList(){
        List<User> listUser = userService.selectAllUser();
        ModelAndView mv = new ModelAndView();
        mv.addObject("listUser", listUser);
        mv.setViewName("list.jsp");
        return mv;
    }
```

JSP 页面
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828004810386-206250894.png)

第三步：修改页面的值后，点击提交

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828004842824-310934982.png)
由于我们在 JSP 页面 input 输入框定义的name属性名是 userList[${status.index}].id 这种形式的，这里我们直接用 UserVo 就能获取页面批量提交的 User信息

#### Map类型的绑定

首先在 UserVo 里面增加一个属性 Map<String,User> userMap
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828010849640-1495160607.png)

第二步：JSP页面，注意看 <input >输入框 name 的属性值

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828011139983-385522269.png)

第三步：Controller 中获取页面的属性
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170828010826952-460408538.png)