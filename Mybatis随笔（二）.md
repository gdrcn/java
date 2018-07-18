### 基于注解
1. 创建MySQL数据库：mybatisDemo和表：user
2. 建立一个Java工程，并导入相应的jar包，具体目录如下
3. 在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml
4. 定义表所对应的实体类
5. 定义操作 user 表的注解接口 UserMapper.java
```
package com.ys.annocation;
 
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据 id 查询 user 表数据
    @Select("select * from user where id = #{id}")
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    @Insert("insert into user(username,sex,birthday,address) value(#{username},#{sex},#{birthday},#{address})")
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    @Update("update user set username=#{username},sex=#{sex} where id=#{id}")
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    @Delete("delete from user where id=#{id}")
    public void deleteUserById(int id) throws Exception;
     
}
```

6. 向 mybatis-configuration.xml 配置文件中注册 UserMapper.java 文件

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803231541162-1969485958.png)

```
<mappers>
       <mapper class="com.ys.annocation.UserMapper"/>
</mappers>
```
7. 创建测试类
```
package com.ys.test;
 
import java.io.InputStream;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.annocation.UserMapper;
import com.ys.po.User;
 
public class UserAnnocationTest {
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
     
    //注解的增删改查方法测试
    @Test
    public void testAnncationCRUD() throws Exception{
        //根据session获取 UserMapper接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //调用selectUserById()方法
        User user = userMapper.selectUserById(1);
        System.out.println(user);
         
        //调用  insertUser() 方法
        User user1 = new User();
        user1.setUsername("aliks");
        user1.setSex("不详");
        userMapper.insertUser(user1);
         
        //调用 updateUserById() 方法
        User user2 = new User();
        user2.setId(6);
        user2.setUsername("lbj");
        userMapper.updateUserById(user2);
         
        //调用 () 方法
        userMapper.deleteUserById(6);
         
        session.commit();
        session.close();
    }
}
```

- 注解配置就不需要userMapper.xml文件


####  properties以及别名意义
- 介绍了mybatis的增删改查入门实例，我们发现在 mybatis-configuration.xml 的配置文件中，对数据库的配置都是硬编码在这个xml文件
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804220404022-1871062647.png)
1. 我们将 数据库的配置语句写在 db.properties 文件中
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm
jdbc.username=root
jdbc.password=root
```
2. mybatis-configuration.xml中加载db.properties文件并读取
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!-- 加载数据库属性文件 -->
<properties resource="db.properties">
</properties>
 <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```
##### mybatis 的别名配置
- 在 userMapper.xml 文件中，我们可以看到resultType 和 parameterType 需要指定，这这个值往往都是全路径，不方便开发，那么我们就可以对这些属性进行一些别名设置
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230103850-1701803236.png)

1. mybatis 默认支持的别名
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230414350-900097506.png)

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230557694-737797674.png)


#### 自定义别名

一. 定义单个别名
首先在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下

```
<!-- 定义别名 -->
<typeAliases>
    <typeAlias type="com.ys.po.User" alias="user"/>
</typeAliases>
```
第二步通过 user 引用
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804231055990-1789427240.png)

二. 批量定义别名
在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下
```
<!-- 定义别名 -->
<typeAliases>
    <!-- mybatis自动扫描包中的po类，自动定义别名，别名是类名(首字母大写或小写都可以,一般用小写) -->
    <package name="com.ys.po"/>
</typeAliases>

```
引用的时候类名的首字母大小写都可以


### 动态SQL
- mybatis对一张表进行的CRUD操作，但是我们发现写的 SQL 语句都比较简单，如果有比较复杂的业务，我们需要写复杂的 SQL 语句，往往需要拼接，而拼接 SQL ，稍微不注意，由于引号，空格等缺失可能都会导致错误
- 使用 mybatis 动态SQL，通过 if, choose, when, otherwise, trim, where, set, foreach等标签，可组合成非常灵活的SQL语句，从而在提高 SQL 语句的准确性的同时，也大大提高了开发人员的效率。


#### 动态SQL:if 语句

根据 username 和 sex 来查询数据。如果username为空，那么将只根据sex来查询；反之只根据username来查询

　　首先不使用 动态SQL 来书写
```
<select id="selectUserByUsernameAndSex"
        resultType="user" parameterType="com.ys.po.User">
    <!-- 这里和普通的sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写id,
            写啥都可以，但是不要空着，如果有多个参数则必须写pojo类里面的属性 -->
    select * from user where username=#{username} and sex=#{sex}
</select>
```
上面的查询语句，我们可以发现，如果 #{username} 为空，那么查询结果也是空，如何解决这个问题呢？使用 if 来判断
```
上面的查询语句，我们可以发现，如果 #{username} 为空，那么查询结果也是空，如何解决这个问题呢？使用 if 来判断
```

- 这样写我们可以看到，如果 sex 等于 null，那么查询语句为 select * from user where username=#{username},但是如果usename 为空呢？那么查询语句为 select * from user where and sex=#{sex}，这是错误的 SQL 语句，如何解决呢？请看下面的 where 语句

#### 动态SQL:if+where 语句

```
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <where>
        <if test="username != null">
           username=#{username}
        </if>
         
        <if test="username != null">
           and sex=#{sex}
        </if>
    </where>
</select>
```

这个“where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。

#### 动态SQL:if+set 语句
同理，上面的对于查询 SQL 语句包含 where 关键字，如果在进行更新操作的时候，含有 set 关键词，我们怎么处理呢？
```
<!-- 根据 id 更新 user 表的数据 -->
<update id="updateUserById" parameterType="com.ys.po.User">
    update user u
        <set>
            <if test="username != null and username != ''">
                u.username = #{username},
            </if>
            <if test="sex != null and sex != ''">
                u.sex = #{sex}
            </if>
        </set>
     
     where id=#{id}
</update>
```
这样写，如果第一个条件 username 为空，那么 sql 语句为：update user u set u.sex=? where id=?

　　　　　　如果第一个条件不为空，那么 sql 语句为：update user u set u.username = ? ,u.sex = ? where id=?

#### 动态SQL:choose(when,otherwise) 语句

- 有时候，我们不想用到所有的查询条件，只想选择其中的一个，查询条件有一个满足即可，使用 choose 标签可以解决此类问题，类似于 Java 的 switch 语句`
```
<select id="selectUserByChoose" resultType="com.ys.po.User" parameterType="com.ys.po.User">
      select * from user
      <where>
          <choose>
              <when test="id !='' and id != null">
                  id=#{id}
              </when>
              <when test="username !='' and username != null">
                  and username=#{username}
              </when>
              <otherwise>
                  and sex=#{sex}
              </otherwise>
          </choose>
      </where>
  </select>
```
　　也就是说，这里我们有三个条件，id,username,sex，只能选择一个作为查询条件

　　　　如果 id 不为空，那么查询语句为：select * from user where  id=?

　　　　如果 id 为空，那么看username 是否为空，如果不为空，那么语句为 select * from user where  username=?;

　　　　　　　　　　如果 username 为空，那么查询语句为 select * from user where sex=?

#### 动态SQL:trim 语句
trim标记是一个格式化的标记，可以完成set或者是where标记的功能

　　①、用 trim 改写上面第二点的 if+where 语句
```
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
        select * from user
        <!-- <where>
            <if test="username != null">
               username=#{username}
            </if>
             
            <if test="username != null">
               and sex=#{sex}
            </if>
        </where>  -->
        <trim prefix="where" prefixOverrides="and | or">
            <if test="username != null">
               and username=#{username}
            </if>
            <if test="sex != null">
               and sex=#{sex}
            </if>
        </trim>
    </select>
```
prefix：前缀　　　　　　

　　prefixoverride：去掉第一个and或者是or
②、用 trim 改写上面第三点的 if+set 语句
```
<!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user u
            <!-- <set>
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex}
                </if>
            </set> -->
            <trim prefix="set" suffixOverrides=",">
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex},
                </if>
            </trim>
         
         where id=#{id}
    </update>
```
suffix：后缀　　

　　suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）

#### 动态SQL: SQL 片段
有时候可能某个 sql 语句我们用的特别多，为了增加代码的重用性，简化代码，我们需要将这些代码抽取出来，然后使用时直接调用。

　　比如：假如我们需要经常根据用户名和性别来进行联合查询，那么我们就把这个代码抽取出来，如下：
　　
```
<!-- 定义 sql 片段 -->
<sql id="selectUserByUserNameAndSexSQL">
    <if test="username != null and username != ''">
        AND username = #{username}
    </if>
    <if test="sex != null and sex != ''">
        AND sex = #{sex}
    </if>
</sql>
```

引用 sql 片段

```
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <trim prefix="where" prefixOverrides="and | or">
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
        <include refid="selectUserByUserNameAndSexSQL"></include>
        <!-- 在这里还可以引用其他的 sql 片段 -->
    </trim>
</select>
```
　注意：①、最好基于 单表来定义 sql 片段，提高片段的可重用性

　　　　　②、在 sql 片段中不要包括 where 


#### 动态SQL: foreach 语句

需求：我们需要查询 user 表中 id 分别为1,2,3的用户

　　sql语句：select * from user where id=1 or id=2 or id=3

　　　　　　 select * from user where id in (1,2,3)
　　　　　　 
①、建立一个 UserVo 类，里面封装一个 List<Integer> ids 的属性
```
package com.ys.vo;
 
import java.util.List;
 
public class UserVo {
    //封装多个用户的id
    private List<Integer> ids;
 
    public List<Integer> getIds() {
        return ids;
    }
 
    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
 
}　　
```
②、我们用 foreach 来改写 select * from user where id=1 or id=2 or id=3
```
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
    select * from user
    <where>
        <!--
            collection:指定输入对象中的集合属性
            item:每次遍历生成的对象
            open:开始遍历时的拼接字符串
            close:结束时拼接的字符串
            separator:遍历对象之间需要拼接的字符串
            select * from user where 1=1 and (id=1 or id=2 or id=3)
          -->
        <foreach collection="ids" item="id" open="and (" close=")" separator="or">
            id=#{id}
        </foreach>
    </where>
</select>
```
测试
```
//根据id集合查询user表数据
@Test
public void testSelectUserByListId(){
    String statement = "com.ys.po.userMapper.selectUserByListId";
    UserVo uv = new UserVo();
    List<Integer> ids = new ArrayList<>();
    ids.add(1);
    ids.add(2);
    ids.add(3);
    uv.setIds(ids);
    List<User> listUser = session.selectList(statement, uv);
    for(User u : listUser){
        System.out.println(u);
    }
    session.close();
}
```

③、我们用 foreach 来改写 select * from user where id in (1,2,3)
```
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
        select * from user
        <where>
            <!--
                collection:指定输入对象中的集合属性
                item:每次遍历生成的对象
                open:开始遍历时的拼接字符串
                close:结束时拼接的字符串
                separator:遍历对象之间需要拼接的字符串
                select * from user where 1=1 and id in (1,2,3)
              -->
            <foreach collection="ids" item="id" open="and id in (" close=") " separator=",">
                #{id}
            </foreach>
        </where>
    </select>
```
- 其实动态 sql 语句的编写往往就是一个拼接的问题，为了保证拼接准确，我们最好首先要写原生的 sql 语句出来，然后在通过 mybatis 动态sql 对照着改，防止出错