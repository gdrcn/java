#### 什么是SpringMVC？
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170817204804350-377625461.png)

- 我们可以看到，在 Spring 的基本架构中，红色圈起来的 Spring Web MVC ，也就是本系列的主角 SpringMVC，它是属于Spring基本架构里面的一个组成部分，属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面，所以我们在后期和 Spring 进行整合的时候，几乎不需要别的什么配置。
- 配置前端过滤器
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
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 对应上一步创建全局配置文件的文件名以及目录 -->
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>
 
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
</web-app>
```

- 编写处理器Handler
```
package com.ys.controller;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
 
public class HelloController implements Controller{
 
    @Override
    public ModelAndView handleRequest(HttpServletRequest request,
            HttpServletResponse response) throws Exception {
        ModelAndView modelView = new ModelAndView();
        //类似于 request.setAttribute()
        modelView.addObject("name","张三");
        modelView.setViewName("/WEB-INF/view/index.jsp");
        return modelView;
    }
 
}
```

- 再springmvc.xml文件种配置Handler处理器映射器，以及视图解析器
```
<!-- 配置Handler -->   
<bean name="/hello.do" class="com.ys.controller.HelloController" />
   <!-- 配置处理器映射器
    将bean的name作为url进行查找，需要在配置Handler时指定bean name（就是url）-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
   <!-- 配置处理器适配器，所有适配器都得实现 HandlerAdapter接口 -->
   <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
 
<!-- 配置视图解析器
    进行jsp解析，默认使用jstl标签，classpath下得有jstl的包-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" />
```

### SpringMVC详细介绍
- Spring Web MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发，Spring Web MVC也是要简化我们日常Web开发的
- Spring Web MVC也是服务到工作者模式的实现，但进行可优化。前端控制器是DispatcherServlet；应用控制器其实拆为处理器映射器(Handler Mapping)进行处理器管理和视图解析器(View Resolver)进行视图管理；页面控制器/动作/处理器为Controller接口（仅包含ModelAndView handleRequest(request, response) 方法）的实现（也可以是任何的POJO类）；支持本地化（Locale）解析、主题（Theme）解析及文件上传等；提供了非常灵活的数据验证、格式化和数据绑定机制；提供了强大的约定大于配置（惯例优先原则）的契约式编程支持。

#### SpringMVC处理请求的流程

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170816195217834-1322829917.png)

第一步：用户发送请求到前端控制器（DispatcherServlet）。

第二步：前端控制器请求 HandlerMapping 查找 Handler，可以根据 xml 配置、注解进行查找。

第三步： 处理器映射器 HandlerMapping 向前端控制器返回 Handler

第四步：前端控制器调用处理器适配器去执行 Handler

第五步：处理器适配器执行 Handler

第六步：Handler 执行完成后给适配器返回 ModelAndView

第七步：处理器适配器向前端控制器返回 ModelAndView

　　　　ModelAndView 是SpringMVC 框架的一个底层对象，包括 Model 和 View

第八步：前端控制器请求试图解析器去进行视图解析

　　　　根据逻辑视图名来解析真正的视图。

第九步：试图解析器向前端控制器返回 view

第十步：前端控制器进行视图渲染

　　　　就是将模型数据（在 ModelAndView 对象中）填充到 request 域

第十一步：前端控制器向用户响应结果

- 下面我们对上面出现的一些组件进行解释
　　1、前端控制器DispatcherServlet（不需要程序员开发）。 
　　　　作用：接收请求，响应结果，相当于转发器，中央处理器。有了DispatcherServlet减少了其它组件之间的耦合度。
　　2、处理器映射器HandlerMapping（不需要程序员开发）。
　　　　作用：根据请求的url查找Handler。
　　3、处理器适配器HandlerAdapter（不需要程序员开发）。
　　　　作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler。
　　4、处理器Handler（需要程序员开发）。
　　　　注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
　　5、视图解析器ViewResolver（不需要程序员开发）。
　　　　作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
　　6、视图View（需要程序员开发jsp）。
　　　　注意：View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf…）
　　ps:不需要程序员开发的，需要程序员自己做一下配置即可。

#### 配置前端控制器
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
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--第一种配置：*.do,还可以写*.action等等，表示以.do结尾的或者以.action结尾的URL都由前端控制器DispatcherServlet来解析
    	第二种配置：/,所有访问的 URL 都由DispatcherServlet来解析，但是这里最好配置静态文件不由DispatcherServlet来解析
    	错误配置：/*,注意这里是不能这样配置的，应为如果这样写，最后转发到 jsp 页面的时候，仍然会由DispatcherServlet进行解析，
    				而这时候会找不到对应的Handler，从而报错！！！
      -->
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
</web-app>
```

- 配置处理器适配器
第一种配置：编写 Handler 时必须要实现 Controller
```
<!-- 配置处理器适配器，所有适配器都得实现 HandlerAdapter接口 -->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
```
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170820153439396-594465168.png)
第二种配置：编写 Handler 时必须要实现 HttpRequestHandler
```
<!-- 配置处理器适配器第二种方法，所有适配器都得实现 HandlerAdapter接口 ，这样配置所有Handler都得实现 HttpRequestHandler接口-->
<bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter" /> 
```
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170820154638537-1136538291.png)

- 编写 Handler

第一种：实现Controller 接口
```
package com.ys.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class HelloController implements Controller{

	@Override
	public ModelAndView handleRequest(HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		ModelAndView modelView = new ModelAndView();
		//类似于 request.setAttribute()
		modelView.addObject("name","张三");
		modelView.setViewName("/WEB-INF/view/index.jsp");
		return modelView;
	}

}
```
第二种：实现 HttpRequestHandler 接口
```
package com.ys.controller;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.HttpRequestHandler;

public class HelloController2 implements HttpRequestHandler{

	@Override
	public void handleRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		request.setAttribute("name", "张三");
		request.getRequestDispatcher("/WEB-INF/view/index.jsp").forward(request, response);
	}

}
```

- 配置处理器映射器
第一种方法
```
<!-- 配置Handler -->    
<bean name="/hello.do" class="com.ys.controller.HelloController2" />

<!-- 配置处理器映射器
	将bean的name作为url进行查找，需要在配置Handler时指定bean name（就是url）-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
　　这样配置的话，那么请求的 URL，必须为 http://localhost:8080/项目名/hello.do
```
第二种方法
```
<!-- 配置Handler -->    
<bean id="hello1" class="com.ys.controller.HelloController" />
<bean id="hello2" class="com.ys.controller.HelloController" />
<!-- 第二种方法：简单URL配置处理器映射器 -->
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings">
		<props>
			<prop key="/hello1.do">hello1</prop>
			<prop key="/hello2.do">hello2</prop>
		</props>
	</property>
</bean>
```

- 配置视图解析器
第一种配置:
```
<!-- 配置视图解析器 
  	进行jsp解析，默认使用jstl标签，classpath下得有jstl的包-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" />
　如果这样配，那么在 Handler 中返回的必须是路径+jsp页面名称+".jsp"
```
第二种配置
```
<!--配置视图解析器  -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 返回视图页面的前缀 -->
		<property name="prefix" value="/WEB-INF/view"></property>
		<!-- 返回页面的后缀 -->
		<property name="suffix" value=".jsp"></property>
	</bean>
```

- 配置视图解析器
第一种配置
```
进行jsp解析，默认使用jstl标签，classpath下得有jstl的包-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" />
　如果这样配，那么在 Handler 中返回的必须是路径+jsp页面名称+".jsp"
```
第二种配置
```
<!--配置视图解析器  -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 返回视图页面的前缀 -->
		<property name="prefix" value="/WEB-INF/view"></property>
		<!-- 返回页面的后缀 -->
		<property name="suffix" value=".jsp"></property>
	</bean>
　　如果这样配，那么在 Handler 中只需要返回在 view 文件夹下的jsp 页面名就可以了。
```

## 基于注解的入门实例

- 在web.xml文件种配置前端处理器
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
        <param-value>classpath:springmvc.xml</param-value>
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

- 在 springmvc.xml文件中配置处理器映射器，处理器适配器，视图解析器
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
    <!--注解处理器映射器  -->   
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>
     
    <!--注解处理器适配器  -->   
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"></bean>  
 
    <!--使用mvc:annotation-driven可以代替上面的映射器和适配器
        这里面会默认加载很多参数绑定方法，比如json转换解析器就默认加载，所以优先使用下面的配置
      -->
    <!-- <mvc:annotation-driven></mvc:annotation-driven> -->
 
 
    <!--单个配置Handler  -->
    <!-- <bean class="com.ys.controller.HelloController"></bean> -->
     
    <!--批量配置Handler,指定扫描的包全称  -->
    <context:component-scan base-package="com.ys.controller"></context:component-scan>
     
 
    <!--配置视图解析器  -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 返回视图页面的前缀 -->
        <property name="prefix" value="/WEB-INF/view/"></property>
        <!-- 返回页面的后缀 -->
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

- 编写Handler
```
package com.ys.controller;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
 
//使用@Controller注解表示这个类是一个Handler
@Controller
public class HelloController {
 
    //@RequestMapping注解括号里面的表示访问的URL
    @RequestMapping("hello")
    public ModelAndView hello(){
        ModelAndView modelView = new ModelAndView();
        //类似于 request.setAttribute()
        modelView.addObject("name","张三");
        //配置返回的视图名，由于我们在springmvc.xml中配置了前缀和后缀，这里直接写视图名就好
        modelView.setViewName("index");
        //modelView.setViewName("/WEB-INF/view/index.jsp");
        return modelView;
    }
 
}
```
- 编写 视图 index.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
hello:${name}
</body>
</html>
```