### 通过mapper接口加载映射文件
- 通过 mapper 接口加载映射文件，这对于后面 ssm三大框架 的整合是非常重要的。那么什么是通过 mapper 接口加载映射文件呢？
- 我们首先看以前的做法，在全局配置文件 mybatis-configuration.xml 通过 <mappers> 标签来加载映射文件，那么如果我们项目足够大，有很多映射文件呢，难道我们每一个映射文件都这样加载吗，这样肯定是不行的，那么我们就需要使用 mapper 接口来加载映射文件

以前的做法：
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807212857174-537878858.png)
改进做法：使用 mapper 接口来加载映射文件

1. 定义userMapper接口
```
package com.ys.mapper;
 
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据 id 查询 user 表数据
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    public void deleteUserById(int id) throws Exception;
}
```

2. 在全局配置文件 mybatis-configuration.xml 文件中加载 UserMapper 接口（单个加载映射文件）
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807213755002-286929257.png)

3. 编写UserMapper.xml 文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.mapper.UserMapper">
 
     
    <!-- 根据 id 查询 user 表中的数据
       id:唯一标识符，此文件中的id值不能重复
       resultType:返回值类型，一条数据库记录也就对应实体类的一个对象
       parameterType:参数类型，也就是查询条件的类型
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        <!-- 这里和普通的sql 查询语句差不多，后面的 #{id}表示占位符，里面不一定要写id,写啥都可以，但是不要空着 -->
        select * from user where id = #{id1}
    </select>
     
     
     
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
     
     
     
    <!-- 根据 id 删除 user 表的数据 -->
    <delete id="deleteUserById" parameterType="int">
        delete from user where id=#{id}
    </delete>
     
</mapper>
```

4.测试
```
//根据id查询user表数据
    @Test
    public void testSelectUserById() throws Exception{
        //获取mapper接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        User user = userMapper.selectUserById(1);
        System.out.println(user);
        session.close();
    }
```

5. 批量加载映射文件
```
<mappers>
       <!--批量加载mapper
          指定 mapper 接口的包名，mybatis自动扫描包下的mapper接口进行加载
         -->
       <package name="com.ys.mapper"/>
</mappers>
```

6. 注意
　1、UserMapper 接口必须要和 UserMapper.xml 文件同名且在同一个包下，也就是说 UserMapper.xml 文件中的namespace是UserMapper接口的全类名
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807214006877-162394845.png)

　2、UserMapper接口中的方法名和 UserMapper.xml 文件中定义的 id 一致

 

　　3、UserMapper接口输入参数类型要和 UserMapper.xml 中定义的 parameterType 一致

 

　　4、UserMapper接口返回数据类型要和 UserMapper.xml 中定义的 resultType 一致
　　
　　
### 一对一，一对多，多对多
[博客](https://www.cnblogs.com/ysocean/p/7309308.html)


### 懒加载

- 需求：查询订单信息，有时候需要关联查出用户信息。
第一种方法：我们直接关联查询出所有订单和用户的信息
```
select * from orders o ,user u where o.user_id = u.id;
```
　　分析：

　　①、这里我们一次查询出所有的信息，需要什么信息的时候直接从查询的结果中筛选。但是如果订单和用户表都比较大的时候，这种关联查询肯定比较耗时。

　　②、我们的需求是有时候需要关联查询用户信息，这里不是一定需要用户信息的。即有时候不需要查询用户信息，我们也查了，程序进行了多余的耗时操作。


第二种方法：分步查询，首先查询出所有的订单信息，然后如果需要用户的信息，我们在根据查询的订单信息去关联用户信息

```
//查询所有的订单信息，包括用户id
select * from orders;
//如果需要用户信息，我们在根据上一步查询的用户id查询用户信息
select * from user where id=user_id
```

　①、这里两步都是单表查询，执行效率比关联查询要高很多

　　②、分为两步，如果我们不需要关联用户信息，那么我们就不必执行第二步，程序没有进行多余的操作。
　　
　　
#### 什么是懒加载？
- 通俗的讲就是按需加载，我们需要什么的时候再去进行什么操作。而且先从单表查询，需要时再从关联表去关联查询，能大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。

　　在mybatis中，resultMap可以实现高级映射(使用association、collection实现一对一及一对多映射)，association、collection具备延迟加载功能。
　　
- 实例
①、创建实体类

　　　　User.java
```
package com.ys.lazyload;
 
import java.util.List;
 
public class User {
    //用户ID
    private int id;
    //用户姓名
    private String username;
    //用户性别
    private String sex;
    //一个用户能创建多个订单，用户和订单构成一对多的关系
    public List<Orders> orders;
     
    public List<Orders> getOrders() {
        return orders;
    }
    public void setOrders(List<Orders> orders) {
        this.orders = orders;
    }
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
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + "]";
    }
}
```
Orders.java
```
package com.ys.lazyload;
 
public class Orders {
    //订单ID
    private int id;
    //用户ID
    private int userId;
    //订单数量
    private String number;
    //和用户表构成一对一的关系，即一个订单只能由一个用户创建
    private User user;
     
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public int getUserId() {
        return userId;
    }
 
    public void setUserId(int userId) {
        this.userId = userId;
    }
 
    public String getNumber() {
        return number;
    }
 
    public void setNumber(String number) {
        this.number = number;
    }
 
    public User getUser() {
        return user;
    }
 
    public void setUser(User user) {
        this.user = user;
    }
 
    @Override
    public String toString() {
        return "Orders [id=" + id + ", userId=" + userId + ", number=" + number
                + ", user=" + user + "]";
    }
 
}
```
②、创建 OrderMapper 接口和 OrderMapper.xml 文件
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809230815136-193517654.png)
由于我们采用 Mapper 代理加载 xxxMapper.xml 文件，这里我们重复一下 Mapper 代理所需的条件，接口和xml文件必须满足以下几个条件：

　　1、接口必须要和 xml 文件同名且在同一个包下，也就是说 xml 文件中的namespace是接口的全类名　　

　　2、接口中的方法名和xml 文件中定义的 id 一致

　　3、接口输入参数类型要和xml 中定义的 parameterType 一致

　　4、接口返回数据类型要和xml 中定义的 resultType 一致
　　
- OrderMapper 接口
```
package com.ys.lazyload;
 
import java.util.List;
 
import com.ys.lazyload.Orders;
import com.ys.lazyload.User;
 
public interface OrdersMapper {
    /**
     * select * from order //得到user_id
     * select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
     * @param userID
     * @return
     */
    //得到订单信息(包含user_id)
    public List<Orders> getOrderByOrderId();
    //根据用户ID查询用户信息
    public User getUserByUserId(int userID);
 
}
```
OrderMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.lazyload.OrdersMapper">
    <!--
         延迟加载：
         select user_id from order WHERE id=1;//得到user_id
         select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
         property:别名(属性名)    column：列名 -->
    <select id="getOrderByOrderId" resultMap="getOrderMap">
        select * from orders
    </select>
    <resultMap type="com.ys.lazyload.Orders" id="getOrderMap">
        <id column="id" property="id"/>
        <result column="number" property="number"/>
        <!-- select:指定延迟加载需要执行的statement的id（根据user_id查询的statement）
                    如果不在本文件中，需要加上namespace
             property:resultMap中type指定类中的属性名
             column:和select查询关联的字段user_id
         -->
        <association property="user" javaType="com.ys.lazyload.User"  column="user_id" select="getUserByUserId">
         
        </association>
    </resultMap>
    <select id="getUserByUserId" resultType="com.ys.lazyload.User">
        select * from user where id=#{id}
    </select>
     
</mapper>
```

③、向 mybatis-configuration.xml 配置文件中注册 OrderMapper.xml 文件
![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809231708324-1791713117.png)

④、开启懒加载配置
```
<!-- 开启懒加载配置 -->
<settings>
    <!-- 全局性设置懒加载。如果设为‘false'，则所有相关联的都会被初始化加载。 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 当设置为‘true'的时候，懒加载的对象可能被任何懒属性全部加载。否则，每个属性都按需加载。 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

　⑤、测试
　```
　@Test
public void testLazy(){
    String statement = "com.ys.lazyload.OrdersMapper.getOrderByOrderId";
    //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
    OrdersMapper orderMapper = session.getMapper(OrdersMapper.class);
    List<Orders> orders = orderMapper.getOrderByOrderId();//第一步
    for(Orders order : orders){
        System.out.println(order.getUser());//第二步
    }
    session.close();
}
　```
　
### 一级缓存、二级缓存
mybatis 为我们提供了一级缓存和二级缓存，可以通过下图来理解

![image](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810222807042-122444009.png)

①、一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

　　②、二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

