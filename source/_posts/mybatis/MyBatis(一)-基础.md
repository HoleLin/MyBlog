---
title: MyBatis(一)-基础
date: 2021-09-11 13:59:17
cover: /img/cover/MyBatis.jpg
tags:
categories:
- MyBatis
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

#### JDBC操作

* 说明 JDBC 操作的核心步骤'

  * 注册数据库驱动类，指定数据库地址，其中包括 DB 的用户名、密码及其他连接信息；
  * 调用 DriverManager.getConnection() 方法创建 Connection 连接到数据库；
  * 调用 Connection 的 createStatement() 或 prepareStatement() 方法，创建 Statement 对象，此时会指定 SQL（或是 SQL 语句模板 + SQL 参数）；
* 通过 Statement 对象执行 SQL 语句，得到 ResultSet 对象，也就是查询结果集；
  * 遍历 ResultSet，从结果集中读取数据，并将每一行数据库记录转换成一个 JavaBean 对象；
* 关闭 ResultSet 结果集、Statement 对象及数据库 Connection，从而释放这些对象占用的底层资源。

#### MyBatis特性

* MyBatis中一个重要的功能就是可以帮助Java开发封装重复性的JDBC代码.
  * 通过Mapper映射配置文件以及相关注解,将Result结果映射为Java对象,在具体的映射规则中可以嵌套其他映射规则和必要的子查询.
* Mybatis相对于Hibernate和各类JPA实现框架更加灵活,更加轻量级,更加可控.
  * 可以在Mybatis的Mapper映射文件中,直接编写原生的SQL语句,应用底层数据库产品的方言,则可以直接优化SQL语句.
  * 还可以按照数据库的使用规则,让原生SQL语句选择我们期望的索引,从而保证服务的性能.
* MyBatis提供了强大的动态SQL功能来帮助开发者摆脱重复劳动.只需要在映射配置文件中编写动态SQL语句,MyBatis就可以根据执行时传入的实际参数拼凑出完整的,可执行的SQL语句.

#### Hibernate,Spring Data JPA,MyBaits对比

* 从**性能角度**,Hibernate,Spring Data JPA在对SQL语句的掌控,SQL手工调试,多表连接查询等方面,不及MyBatis直接使用原生SQL语句方便,高效.
* 从**可移植角度**,Hibernate屏蔽了底层数据库方言,Spring Data JPA屏蔽了ORM的差异,而MyBatis因为直接编写原生SQL,会与具体的数据完全绑定
* 从开发效率角度,Hibernate,Spring Data JPA处理中小型项目的效率会略高于MyBatis.

#### MyBatis使用步骤

* 定义操作接口以及XML

  ```java
  public interface CustomerMapper {
  
      // 根据用户Id查询Customer(不查询Address)
      Customer find(long id);
  
      // 根据用户Id查询Customer(同时查询Address)
      Customer findWithAddress(long id);
  
      // 根据orderId查询Customer
      Customer findByOrderId(long orderId);
  
      // 持久化Customer对象
      int save(Customer customer);
  
  }
  ```

* 无需真正实现`CustomerMapper`接口,而是在`resources/mapper`目录下配置相应的配置文件--`CustomerMapper.xml`,在该文件中需要执行的SQL语句以及查询结果集的映射规则.

  ```xml
  <mapper namespace="org.example.dao.CustomerMapper">
  
      <!-- 定义映射规则 -->
      <resultMap id="customerSimpleMap" type="Customer">
          <!--  主键映射 -->
          <id property="id" column="id"/>
          <!--  属性映射 -->
          <result property="name" column="name"/>
          <result property="phone" column="phone"/>
      </resultMap>
  
      <!-- 定义映射规则 -->
      <resultMap id="customerMap" type="Customer">
  
          <!--  主键映射 -->
          <id property="id" column="id"/>
          <!--  属性映射 -->
          <result property="name" column="name"/>
          <result property="phone" column="phone"/>
  
          <!-- 映射addresses集合，<collection>标签用于映射集合类的属性，实现一对多的关联关系 -->
          <collection property="addresses" javaType="list" ofType="Address">
              <id property="id" column="address_id"/>
              <result property="street" column="street"/>
              <result property="city" column="city"/>
              <result property="country" column="country"/>
          </collection>
      </resultMap>
      <!-- 定义t_order_item与OrderItem对象之间的映射关系-->
      <resultMap id="orderItemtMap" type="OrderItem">
          <id property="id" column="id"/>
          <result property="amount" column="amount"/>
          <result property="orderId" column="order_id"/>
          <!--映射OrderItem关联的Product对象，<association>标签用于实现一对一的关联关系-->
          <association property="product" javaType="Product">
              <id property="id" column="product_id"/>
              <result property="name" column="name"/>
              <result property="description" column="description"/>
              <result property="price" column="price"/>
          </association>
      </resultMap>
    
      <!-- 定义select语句，CustomerMapper接口中的find()方法会执行该SQL，
          查询结果通过customerSimpleMap这个映射生成Customer对象-->
      <select id="find" resultMap="customerSimpleMap">
          SELECT * FROM t_customer WHERE id = #{id:INTEGER}
      </select>
  
      <!-- 定义select语句，CustomerMapper接口中的findWithAddress()方法会执行该SQL，
          查询结果通过customerMap这个映射生成Customer对象-->
      <select id="findWithAddress" resultMap="customerMap">
          SELECT c.*,a.id as address_id, a.* FROM t_customer as c join t_address as a
          on c.id = a.customer_id
          WHERE c.id = #{id:INTEGER}
      </select>
  
      <!-- CustomerMapper接口中的findByOrderId()方法会执行该SQL，
          查询结果通过customerSimpleMap这个映射生成Customer对象-->
      <select id="findByOrderId" resultMap="customerSimpleMap">
          SELECT * FROM t_customer as c join t_order as t
          on c.id = t.customer_id
          WHERE t.customer_id = #{id:INTEGER}
      </select>
    
   		<!-- 定义select语句，OrderMapper接口中的findByCustomerId()方法会执行该SQL，
          查询结果通过orderMap这个映射生成Order对象。注意这里大于号、小于号在XML中的写法-->
      <select id="findByCustomerId" resultMap="orderMap">
          SELECT * FROM t_order WHERE customer_id = #{id}
          and create_date_time <![CDATA[ >= ]]> #{startTime}
          and  create_date_time <![CDATA[ <= ]]> #{endTime}
      </select>
  
      <!-- 定义insert语句，CustomerMapper接口中的save()方法会执行该SQL，
          数据库生成的自增id会自动填充到传入的Customer对象的id字段中-->
      <insert id="save" keyProperty="id" useGeneratedKeys="true">
        insert into t_customer (id, name, phone)
        values (#{id},#{name},#{phone})
      </insert>
  </mapper>
  
  ```

* Java操作

  ```java
  public class DaoUtils {
  
      private static SqlSessionFactory factory;
  
      static { // 在静态代码块中直接读取MyBatis的mybatis-config.xml配置文件
          String resource = "mybatis-config.xml";
          InputStream inputStream = null;
          try {
              inputStream = Resources.getResourceAsStream(resource);
          } catch (IOException e) {
              System.err.println("read mybatis-config.xml fail");
              e.printStackTrace();
              System.exit(1);
          }
          // 加载完mybatis-config.xml配置文件之后，会根据其中的配置信息创建SqlSessionFactory对象
          factory = new SqlSessionFactoryBuilder()
                  .build(inputStream);
      }
      
      public static <R> R execute(Function<SqlSession, R> function) {
          // 创建SqlSession
          SqlSession session = factory.openSession();
          try {
              R apply = function.apply(session);
              // 提交事务
              session.commit();
              return apply;
          } catch (Throwable t) {
              // 出现异常的时候，回滚事务
              session.rollback(); 
              System.out.println("execute error");
              throw t;
          } finally {
              // 关闭SqlSession
              session.close();
          }
      }
  }
  
  ```

* MyBatis配置文件 `mybatis-config.xml`,配置要连接的数据库地址,Mapper.xml等信息.

  ```xml
  <configuration>
      <properties> <!-- 定义属性值 -->
          <property name="username" value="root"/>
          <property name="id" value="xxx"/>
      </properties>
      <settings><!-- 全局配置信息 -->
          <setting name="cacheEnabled" value="true"/>
      </settings>
  
      <typeAliases>
          <!-- 配置别名信息，在映射配置文件中可以直接使用Customer这个别名
              代替org.example.domain.Customer这个类 -->
          <typeAlias type="org.example.domain.Customer" alias="Customer"/>
          <typeAlias type="org.example.domain.Address" alias="Address"/>
          <typeAlias type="org.example.domain.Order" alias="Order"/>
          <typeAlias type="org.example.domain.OrderItem" alias="OrderItem"/>
          <typeAlias type="org.example.domain.Product" alias="Product"/>
  
      </typeAliases>
      
      <environments default="development">
          <environment id="development">
              <!-- 配置事务管理器的类型 -->
              <transactionManager type="JDBC"/>
              <!-- 配置数据源的类型，以及数据库连接的相关信息 -->
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                  <property name="username" value="root"/>
                  <property name="password" value="xxx"/>
              </dataSource>
          </environment>
      </environments>
  
      <!-- 配置映射配置文件的位置 -->
      <mappers>
          <mapper resource="mapper/CustomerMapper.xml"/>
          <mapper resource="mapper/AddressMapper.xml"/>
          <mapper resource="mapper/OrderItemMapper.xml"/>
          <mapper resource="mapper/OrderMapper.xml"/>
          <mapper resource="mapper/ProductMapper.xml"/>
      </mappers>
  </configuration>

* Service层操作

  ```java
  public class ProductService {
      // 创建商品
      public long createProduct(Product product) {
          // 检查product中的各个字段是否合法
          Preconditions.checkArgument(product != null, "product is null");
          Preconditions.checkArgument(!Strings.isNullOrEmpty(product.getName()), "product name is empty");
          Preconditions.checkArgument(!Strings.isNullOrEmpty(product.getDescription()), "description name is empty");
          Preconditions.checkArgument(product.getPrice().compareTo(new BigDecimal(0)) > 0,
                  "price<=0 error");
          return DaoUtils.execute(sqlSession -> {
              // 通过ProductMapper中的save()方法完成持久化
              ProductMapper productMapper = sqlSession.getMapper(ProductMapper.class);
              return productMapper.save(product);
          });
      }
  }    
  ```

#### MyBatis架构

* MyBatis分为三层架构,分别为**基础支撑层,核心处理层**和**接口层**.

  <img src="https://www.holelin.cn/img/mybatis/MyBatis%E6%9E%B6%E6%9E%84%E5%9B%BE.png" alt="img" style="zoom:50%;" />

* 基础支撑层

  * 类型转换模块

    * `<typeAliase>`别名机制

    * 实现MyBatis中JDBC类型与Java类型之间相互转换.

      ```
      JDBC类 <-- SQL绑定实参 <--  类型转换 <-- SQL绑定实参 <-- Java类型
      JDBC类  --> 结果集映射  --> 类型转换  --> 结果集映射  --> Java类型
      ```

  * 日志模块

  * 反射工具模块

  * Binding模块

  * 数据源模块

  * 缓存模块

  * 解析器模块

  * 事务管理模块

* 核心处理层

  * 配置解析

    * MyBatis有三处可以添加配置信息的地方,分别是`mybatis-config.xml`配置文件,`Mapper.xml`配置文件以及`Mapper`接口中的注解细信息.在MyBatis初始化过程中,会加载这些配置信息,并解析之后得到的配置对象保存到`Configuration`对象中.

  * SQL解析与`scripting`模块

    * MyBatis的动态SQL功能,只需要通过MyBatis提供的标签即可根据实际的运行条件动态生成实际的SQL语句.`<where>`,`<if>`,`<set>`标签等.
    * MyBatis中`scripting`模块就是负责动态生成SQL的核心模块

  * SQL执行

    * 在MyBatis中,要执行一条SQL语句,会涉及非常多的组件,比较核心的有`Executor`,`StatementHandler`,`ParameterHandler`和`ResultSetHandler`

    * 其中,`Executor`会调用事务管理模块实现事务的相关控制,同时会通过缓存模块管理一级缓存和二级缓存.SQL语句的真正执行将会由`StatementHandler`实现.`StatementHandler`会依赖`ParameterHandler`进行SQL模板的实参绑定,然后由`java.sql.Statement`对象将SQL语句以及绑定好的实参传到数据库执行,从数据库中拿到`ResultSet`,最后,由`ResultSetHandler`将`Result`映射成Java对象返回给调用方.

      <img src="https://www.holelin.cn/img/mybatis/MyBatis%E6%89%A7%E8%A1%8C%E4%B8%80%E6%9D%A1SQL%E7%9A%84%E6%B5%81%E7%A8%8B.png" alt="img" style="zoom:50%;" />

  * 插件

* 接口层

  * 接口层是MyBatis暴露给调用的接口集合

