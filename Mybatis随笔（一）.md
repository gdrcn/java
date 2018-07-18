### 什么是MyBatis?
- MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。

- iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

- MyBatis 是支持普通SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Ordinary Java Objects，普通的 Java对象）映射成数据库中的记录。


### 为什么会有MyBatis?
- 对比JDBC

Person表
```
public class Person {
    private Long pid;
    private String pname;
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
}
```

JDBC查询
```
package com.ys.dao;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
 
import javax.swing.DebugGraphics;
 
import com.ys.bean.Person;
 
public class CRUDDao {
    //MySQL数据库驱动
    public static String driverClass = "com.mysql.jdbc.Driver";
    //MySQL用户名
    public static String userName = "root";
    //MySQL密码
    public static String passWord = "root";
    //MySQL URL
    public static String url = "jdbc:mysql://localhost:3306/test";
    //定义数据库连接
    public static Connection conn = null;
    //定义声明数据库语句,使用 预编译声明 PreparedStatement提高数据库执行性能
    public static PreparedStatement ps = null;
    //定义返回结果集
    public static ResultSet rs = null;
    /**
     * 查询 person 表信息
     * @return：返回 person 的 list 集合
     */
    public static List<Person> readPerson(){
        List<Person> list = new ArrayList<>();
        try {
            //加载数据库驱动
            Class.forName(driverClass);
            //获取数据库连接
            conn = DriverManager.getConnection(url, userName, passWord);
            //定义 sql 语句,?表示占位符
            String sql = "select * from person where pname=?";
            //获取预编译处理的statement
            ps = conn.prepareStatement(sql);
            //设置sql语句中的参数，第一个为sql语句中的参数的?(从1开始)，第二个为设置的参数值
            ps.setString(1, "qzy");
            //向数据库发出 sql 语句查询，并返回结果集
            rs = ps.executeQuery();
            while (rs.next()) {
                Person p = new Person();
                p.setPid(rs.getLong(1));
                p.setPname(rs.getString(2));
                list.add(p);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //关闭数据库连接
            if(rs!=null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(ps!=null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
         
        return list;
    }
 
    public static void main(String[] args) {
        System.out.println(CRUDDao.readPerson());
    }
}
```


- 分析
　　通过上面的例子我们可以分析如下几点：

　　①、问题一：数据库连接，使用时就创建，使用完毕就关闭，这样会对数据库进行频繁的获取连接和关闭连接，造成数据库资源浪费，影响数据库性能。

　　　　设想解决：使用数据库连接池管理数据库连接

　　②、问题二：将 sql 语句硬编码到程序中，如果sql语句修改了，那么需要重新编译 Java 代码，不利于系统维护

　　　　设想解决：将 sql 语句配置到 xml 文件中，即使 sql 语句变化了，我们也不需要对 Java 代码进行修改，重新编译

　　③、问题三：在 PreparedStatement 中设置参数，对占位符设置值都是硬编码在Java代码中，不利于系统维护

　　　　设想解决：将 sql 语句以及占位符和参数都配置到 xml 文件中

　　④、问题四：从 resultset 中遍历结果集时，对表的字段存在硬编码，不利于系统维护

　　　　设想解决：将查询的结果集自动映射为 Java 对象

　　⑤、问题五：重复性代码特别多，频繁的 try-catch

　　　　设想解决：将其整合到一个 try-catch 代码块中

　　⑥、问题六：缓存做的很差，如果存在数据量很大的情况下，这种方式性能特别低

　　　　设想解决：集成缓存框架去操作数据库

　　⑦、问题七：sql 的移植性不好，如果换个数据库，那么sql 语句可能要重写

　　　　设想解决：在 JDBC 和 数据库之间插入第三方框架，用第三方去生成 sql 语句，屏蔽数据库的差异
　　　　
　　　　
### 入门实例

1. 创建MySQL数据库：mybatisDemo和表：user
这里我们就不写脚本创建了，创建完成后，再向其中插入几条数据即可。

　　user 表字段如下：
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803001845334-1050272765.png)
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803082356912-1932891072.png)

2. 建立一个Java工程，并导入相应的jar包，具体目录如下
注意：log4j和Junit不是必须的，但是我们为了查看日志以及便于测试，加入了这两个jar包

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002013365-1628654827.png)

3. 在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
<!-- 注意：environments标签，当mybatis和spring整合之后，这个标签是不用配置的 -->
 
<!-- 可以配置多个运行环境，但是每个 SqlSessionFactory 实例只能选择一个运行环境  
  一、development:开发模式
   二、work：工作模式-->
 <environments default="development">
 <!--id属性必须和上面的default一样  -->
    <environment id="development">
    <!--事务管理器
        一、JDBC：这个配置直接简单使用了 JDBC 的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围
        二、MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接。而它会让容器来管理事务的整个生命周期
            比如 spring 或 JEE 应用服务器的上下文，默认情况下，它会关闭连接。然而一些容器并不希望这样，
            因此如果你需要从连接中停止它，就可以将 closeConnection 属性设置为 false，比如：
            <transactionManager type="MANAGED">
                <property name="closeConnection" value="false"/>
            </transactionManager>
      -->
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatisdemo"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
  </environments>
   
</configuration>
```

4. 定义表所对应的实体类

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002247834-447237894.png)

```
package com.ys.po;
 
import java.util.Date;
 
public class User {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public Date getBirthday() {
        return birthday;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
    public String getAddress() {
        return address;
    }
    public void setAddress(String address) {
        this.address = address;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + ", birthday=" + birthday + ", address=" + address + "]";
    }
}
```

5. 定义操作 user 表的sql映射文件userMapper.xml
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002356303-1648947603.png)

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.po.userMapper">
 
    <!-- 根据 id 查询 user 表中的数据
       id:唯一标识符，此文件中的id值不能重复
       resultType:返回值类型，一条数据库记录也就对应实体类的一个对象
       parameterType:参数类型，也就是查询条件的类型
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        <!-- 这里和普通的sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写id,写啥都可以，但是不要空着，如果有多个参数则必须写pojo类里面的属性 -->
        select * from user where id = #{id}
    </select>
   
     
    <!-- 查询 user 表的所有数据
        注意：因为是查询所有数据，所以返回的应该是一个集合,这个集合里面每个元素都是User类型
     -->
    <select id="selectUserAll" resultType="com.ys.po.User">
        select * from user
    </select>
     
    <!-- 模糊查询：根据 user 表的username字段
            下面两种写法都可以，但是要注意
            1、${value}里面必须要写value，不然会报错
            2、${}表示拼接 sql 字符串，将接收到的参数不加任何修饰拼接在sql语句中
            3、使用${}会造成 sql 注入
     -->
    <select id="selectLikeUserName" resultType="com.ys.po.User" parameterType="String">
        select * from user where username like '%${value}%'
        <!-- select * from user where username like #{username} -->
    </select>
     
    <!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
     
    <!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user set username=#{username} where id=#{id}
    </update>
     
    <!-- 根据 id 删除 user 表的数据 -->
    <delete id="deleteUserById" parameterType="int">
        delete from user where id=#{id}
    </delete>
</mapper>
```

6. 向 mybatis-configuration.xml 配置文件中注册 userMapper.xml 文件
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002534725-1062751486.png)

```
<mappers>
       <!-- 注册userMapper.xml文件，
       userMapper.xml位于com.ys.mapper这个包下，所以resource写成com/ys/mapper/userMapper.xml-->
       <mapper resource="com/ys/mapper/userMapper.xml"/>
</mappers>
```

7. 创建测试类
```
package com.ys.test;
 
import java.io.InputStream;
import java.util.List;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.po.User;
 
public class CRUDTest {
    //定义 SqlSession
    SqlSession session =null;
     
    @Before
    public void init(){
        //定义mybatis全局配置文件
        String resource = "mybatis-configuration.xml";
        //加载 mybatis 全局配置文件
        InputStream inputStream = CRUDTest.class.getClassLoader()
                                    .getResourceAsStream(resource);
        //构建sqlSession的工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //根据 sqlSessionFactory 产生 session
        session = sessionFactory.openSession();
    }
     
    //根据id查询user表数据
    @Test
    public void testSelectUserById(){
        /*这个字符串由 userMapper.xml 文件中 两个部分构成
            <mapper namespace="com.ys.po.userMapper"> 的 namespace 的值
            <select id="selectUserById" > id 值*/
        String statement = "com.ys.po.userMapper.selectUserById";
        User user = session.selectOne(statement, 1);
        System.out.println(user);
        session.close();
    }
     
    //查询所有user表所有数据
    @Test
    public void testSelectUserAll(){
        String statement = "com.ys.po.userMapper.selectUserAll";
        List<User> listUser = session.selectList(statement);
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
    }
     
    //模糊查询：根据 user 表的username字段
    @Test
    public void testSelectLikeUserName(){
        String statement = "com.ys.po.userMapper.selectLikeUserName";
        List<User> listUser = session.selectList(statement, "%t%");
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
         
    }
    //向 user 表中插入一条数据
    @Test
    public void testInsertUser(){
        String statement = "com.ys.po.userMapper.insertUser";
        User user = new User();
        user.setUsername("Bob");
        user.setSex("女");
        session.insert(statement, user);
        //提交插入的数据
        session.commit();
        session.close();
    }
     
    //根据 id 更新 user 表的数据
    @Test
    public void testUpdateUserById(){
        String statement = "com.ys.po.userMapper.updateUserById";
        //如果设置的 id不存在，那么数据库没有数据更改
        User user = new User();
        user.setId(4);
        user.setUsername("jim");
        session.update(statement, user);
        session.commit();
        session.close();
    }
     
 
    //根据 id 删除 user 表的数据
    @Test
    public void testDeleteUserById(){
        String statement = "com.ys.po.userMapper.deleteUserById";
        session.delete(statement,4);
        session.commit();
        session.close();
    }
}
```

- 补充：如何得到插入数据之后的主键值？
第一种：数据库设置主键自增机制

　　　　userMapper.xml 文件中定义：

```
<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select LAST_INSERT_ID()：查询上一次执行insert 操作返回的主键id值，只适用于自增主键
             resultType:指定 select LAST_INSERT_ID() 的结果类型
             order:AFTER，相对于 select LAST_INSERT_ID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user(username,sex,birthday,address)
            value(#{username},#{sex},#{birthday},#{address})
    </insert>
```

　测试：
　```
　//向 user 表中插入一条数据并获取主键值
    @Test
    public void testInsertUser(){
        String statement = "com.ys.po.userMapper.insertUser";
        User user = new User();
        user.setUsername("Bob");
        user.setSex("女");
        session.insert(statement, user);
        //提交插入的数据
        session.commit();
        //打印主键值
        System.out.println(user.getId());
        session.close();
    }
　```
　
　第二种：非自增主键机制
　```
　<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
        流程是：首先通过 select UUID()得到主键值，然后设置到 user 对象的id中，在进行 insert 操作
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select UUID()：得到主键的id值，注意这里是字符串
             resultType:指定 select UUID() 的结果类型
             order:BEFORE，相对于 select UUID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="String" order="BEFORE">
            select UUID()
        </selectKey>
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
　```
　
- 总结：  

　　①、parameterType:指定输入参数的类型

　　②、resultType:指定输出结果的类型，在select中如果查询结果是集合，那么也表示集合中每个元素的类型

　　③、#{}:表示占位符，用来接收输入参数，类型可以是简单类型，pojo,HashMap等等

　　　　如果接收简单类型，#{}可以写成 value 或者其他名称

　　　　如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值

　　④、${}:表示一个拼接符，会引起 sql 注入，不建议使用　　

　　　　用来接收输入参数，类型可以是简单类型，pojo,HashMap等等

　　　　如果接收简单类型，${}里面只能是 value

　　　　如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值