## 1、JDBC

JDBC是Java提供访问数据库的同一接口，每个厂商针对这套接口具有不同的实现。

```java
/**
 * jdbc基本试验
 */
public class JdbcTest {
    
        public static void main(String[] args) throws Exception {
            //1.导入驱动  mysql-connect包
            //2.注册驱动，驱动实际上为厂商对接口的实现jar包
            Class.forName("com.mysql.jdbc.Driver");
            //3.获取连接对象，通过 DriverManager.getConnection() 方法建立一个连接
            Connection conn = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/mytest",
                    "root",
                    "root");
            //4.编写sql
            String sql = "select * from person";
            //5.获取发送平台
            /*
             JDBC 的 Statement，CallableStatement 和 PreparedStatement 接口定义的方法和属性，
             可以让你发送 SQL 命令或 PL/SQL 命令到数据库，并从你的数据库接收数据。
            */
            Statement statement = conn.createStatement();
            //sql已经预编译好，可以通过set方法进行参数注入，然后正常执行
            //PreparedStatement preparedStatement = conn.prepareStatement(sql);
            //preparedStatement.setInt(1);
            //preparedStatement.setString("111");

            //6.执行，resultSet将每一行数据作为一个数据集，每一列数据再作为单独的数据。
            //next()：将光标下移一行，光标从0开始
            ResultSet resultSet = statement.executeQuery(sql);
            resultSet.next();
            String string = resultSet.getString(2);
            System.out.println(string);
            //资源关闭
            statement.close();
            conn.close();
        }
}

```

**statement的类别：**

|     **接口**      |                         **推荐使用**                         |
| :---------------: | :----------------------------------------------------------: |
|     Statement     | 可以正常访问数据库，适用于运行静态 SQL 语句。 Statement 接口不接受参数。 |
| PreparedStatement | 计划多次使用 SQL 语句， PreparedStatement 接口运行时接受输入的参数。 |
| CallableStatement | 适用于当你要访问数据库存储过程的时候， CallableStatement 接口运行时也接受输入的参数。 |

## 2、Mybatis

mybatis将编写sql的框架代码分离，交由程序员操作。

![1](https://github.com/oldgarlic/Blog/blob/master/mybatis/images/1.png)

### 1、流程

```xml
<!--mybatis-config.xml定义数据源，以及接口映射关系-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 根标签 -->
<configuration>
    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="test">
        <!-- id：唯一标识 -->
        <environment id="test">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC" />
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mytest" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    <!--映射关系-->
    <mappers>
        <mapper resource="myMapper.xml"></mapper>
    </mappers>
</configuration>


<!--myMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper:根标签，namespace：命名空间，随便写，一般保证命名空间唯一 -->
<mapper namespace="MyMapper">
    <!-- statement，内容：sql语句。id：唯一标识，随便写，在同一个命名空间下保持唯一
       resultType：sql语句查询结果集的封装类型,tb_user即为数据库中的表
     -->
    <select id="selectUser" resultType="com.lll.mybatisplusdemo.pojo.Person">
      select * from person1 where id = #{id}
   </select>
</mapper>


```

**Test操作流程：**

```java
public class MybatisTest {
    public static void main(String[] args) throws Exception {
        //1.读取xml文件
        String resource = "mybatis-config.xml";
        InputStream in = Resources.getResourceAsStream(resource);
        //2.构建sqlSession
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3.发送sql并处理结果
        Person selectUser = sqlSession.selectOne("selectUser", 1);
        System.out.println(selectUser);
    }
}

```

**接口式编程：**

```xml
<!--mybatix-config.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 根标签 -->
<configuration>
    <!-- 环境，可以配置多个，default：指定采用哪个环境 -->
    <environments default="test">
        <!-- id：唯一标识 -->
        <environment id="test">
            <!-- 事务管理器，JDBC类型的事务管理器 -->
            <transactionManager type="JDBC" />
            <!-- 数据源，池类型的数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mytest" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="myMapper.xml"></mapper>
    </mappers>
</configuration>


<!--myMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper:根标签，namespace：命名空间，随便写，接口式编程下需要和接口的路径对应 -->
<mapper namespace="com.lll.mybatisplusdemo.mapper.PersonMapper">
    <!-- statement，内容：sql语句。id：唯一标识，随便写，在同一个命名空间下保持唯一，接口式编程下需要和方法名称对应。resultType：sql语句查询结果集的封装类型,tb_user即为数据库中的表
     -->
    <select id="selectOne" resultType="com.lll.mybatisplusdemo.pojo.Person">
      select * from person1 where id = #{id}
   </select>
</mapper>

```

**接口和操作：**

```java
public interface PersonMapper {
    Person selectOne(int id);
}

public class MybatisTest {
    public static void main(String[] args) throws Exception {
        //1.读取xml文件
        String resource = "mybatis-config.xml";
        InputStream in = Resources.getResourceAsStream(resource);
        //2.构建sqlSession
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3.获取mapper，发送sql并处理结果
        PersonMapper mapper = sqlSession.getMapper(PersonMapper.class);
        Person person = mapper.selectOne(3);
        System.out.println(person);
    }
}
```

**接口式编程流程：**

	1. mybatis-config.xml 全局配置文件，主要配置数据源等全局信息。
 	2. 编写Mapper和XXXMapper.xml文件，名称空间和接口路径，sql Id和方法的对应关系。
 	3. 将XXXMapper.xml与mybatis.config.xml关联起来。
 	4. 通过读取全局配置文件获取sqlSessionFactory，通过openSession()获得sqlSession。
 	5. 根据接口.class获取可执行mapper。
 	6. mapper执行方法并处理结果。
 	7. 接口式编程有安全地类型检查，解耦能力。

## 3、全局配置文件

### 1、properties

```xml
    <!--properties可以引入外路径下的资源，resource:引入项目路径下，url:引入网络/自盘路径下    -->
    <properties resource="" url="">
    <!--可以在properties中字型进行配置-->
        <property name="" value=""/>
    </properties>
```

### 2、setting

```xml
    <!--mybatis在运行过程中，是否开启某些设置，例如是否开启缓存，驼峰命名法等-->
    <settings>
        <!-- name:设置项，value:true/false,默认为false-->
        <setting name="" value=""/>
    </settings>
```

### 3、typeAliases

```xml
    <!--别名处理器，可以为某个java类取别名    -->
    <typeAliases>
        <!--type:java类的全路径，别名默认为类名小写
            alias:指定别名
            package:批量取别名，指定包路径，给包及包路径下的类取别名，默认的类名小写
            批量情况下,如果类名重复，可以用@Alias来指定别名
         -->
        <typeAlias type="" alias=""></typeAlias>
        <package name=""/>
    </typeAliases>
```

### 4、typeHandlers

```xml
   <!--类型处理器：数据类型处理到数据库可兼容的类型-->
    <typeHandlers></typeHandlers>
```

### 5、plugins

先过

### 6、environments

```xml
    <!--具体环境配置信息，default：指定某个环境，值为环境id    -->
    <environments default="">
        <!--id：环境id-->
        <environment id="" >
            <!--事务管理器
                type：事务管理器的类型，JDBC/MANAGED
            -->
            <transactionManager type=""></transactionManager>
            <!--数据源
                type：数据源类型，UNPOOLED|POOLED|JNDI
                自定义数据源
                property：数据源的具体信息
            -->
            <dataSource type="">
                <property name="" value=""/>
            </dataSource>
        </environment>
    </environments>
```

### 7、mappers

```xml
    <mappers>
        <!--mapper：注册一个sql映射
            url：引用项目路径下的文件
            resource：引用网络/磁盘路径下的文件
            class：引用接口，使用该方法必须满足如下条件：
                    1)有注解文件，xml文件需要和接口在同一个路径下，必须同名
                    2)没有注解文件，接口下使用注解方式
        -->
        <mapper class="" url="" resource=""></mapper>
        <!--package：批量注册，name：包名，情况和class的情况相同-->
        <package name=""/>
    </mappers>
```

## 4、Mapper映射文件

mybatis允许增删改直接定义以下类型返回值：Integer，void，long，boolean

### 1、insert,delete,update

```xml
  <!--insert语句配置：
        useGeneratedKeys：仅对insert有用，这 会 告 诉 MyBatis 使 用 JDBC 的
        getGeneratedKeys 方法来取出由数据（比如：像 MySQL 和 SQL
        Server 这样的数据库管理系统的自动递增字段）内部生成的主键。
        默认值： false。
        keyProperty：仅对insert有用，标记一个属性， MyBatis 会通过getGeneratedKeys
        或者通过 insert 语句的 selectKey 子元素设置它的值。默认：不设置。
        databaseId：数据库厂商的选择
        -->
    <insert id="s" parameterType="" useGeneratedKeys="true" keyProperty="id" databaseId="">
    </insert>
```

### 2、select

```xml
    <!--select语句配置
        id：sql标识符
        parameterType:参数数据类型
        resultType:返回值数据类型
        resultMap：引用外部resultMap，也就是结果集封装
        flushCache：该语句执行，会清空缓存
        useCache：该语句是否缓存
        timeout：过期时间
        fetchSize:驱动程序每次返回结果集的行数
        statementType:STATEMENT,PREPARED 或 CALLABLE 的一种。这会让 MyBatis
        使用选择使用 Statement， PreparedStatement 或 CallableStatement。
        默认值： PREPARED。
        resultSetType:STATEMENT,PREPARED 或 CALLABLE 的一种。这会让 MyBatis
        使用选择使用 Statement， PreparedStatement 或 CallableStatement。
        默认值： PREPARED。(驱动程序自行处理)
    -->
    <select id=""  parameterType="" resultType="" resultMap="" flushCache="true" useCache="true" timeout=""
        statementType=""  resultSetType=""   fetchSize="">
    </select>
	<!--返回集合:resultType写集合中类的类型就可以
		返回map:resultType写map
		@mapKey(""),注解在接口的方法上，指定map的id
	-->


```

### 3、参数处理

```xml
   <!--mybatis在做多参数处理时，会把多参数封装到一个map中，
	   key:param1,param2...，value:实际参数1,参数2
	   在接口处可使用@param("id")来替换map中的key-->
	<select id="">
    	select * from person where id = #{id} and name = #{name}
	</select>

	#{}:预编译
	${}:取出的值直接注入sql中
```

**POJO：**

如果多参数都为POJO中的成员变量，可以直接传入POJO。

**Map：**

如果多个参数不也业务模型中的数据，可以传入map。

### 4、ResultMap

```xml
    <!--ResultType:mybatis内部的自动封装-->
    <!--ResultMap:外部自己手动指定
        当查询的对象里有对象的时候，可以进行级联查询/association
        级联查询：使用成员对象.属性进行赋值
    -->

    <!--解决列名不匹配的方式
       id：唯一id
       type：自定义的java类型，一般为某个java类
    -->
    <resultMap id="" type="">
        <!--id：指定主键列封装规则
            column：指定那一列
            property：指对应javabean的属性
        -->
        <id column="" property="" ></id>
        <!--result：定义普通封装规则-->
        <result column="" property=""></result>
        <!--association:可以指定联合的javabean对象
            property:那个属性对应着这个
            javaType：指定这个属性对象的类型
            select:指定该属性由select标记的方法查出
            column:那一列作为参数值传入select指定的方法
        -->
        <association property="" javaType="" select="" column="" >

        </association>
        <!--collection:定义关联集合属性的封装规则
            ofType:集合中数据类型
            fetchType：延迟加载
        -->
        <collection property="" ofType="" fetchType="">

        </collection>

        <!--discriminator：mybatis可以根据鉴别器根据某列的值来改变结果的封装
            column:指定判断的列
            javaType:判断列对应的java类型
        -->
        <discriminator javaType="" column="">
            <!--value：期望值
                resultType：预封装的javabean
            -->
            <case value="" resultType="">
            <!--该情况下的封装操作-->
            </case>
        </discriminator>
    </resultMap>
```

## 5、动态sql

```xml
    <select id="">
    select * from person where id=#{id}
        <!--if：判断语句
            test：判断语句，某参数怎样怎样
            表达式为OGNL表达式
        -->
        <if test="name!=null">
            and name =#{name}
        </if>
        <!--choose判断语句

        -->
        <choose>
            <when test="age!=null">
                and age =#{age}
            </when>
            <otherwise>

            </otherwise>
        </choose>

        select * from person
        <!--where：就是条件大集合-->
        <where>
            <!--写上判断语句，if/choose-->
        </where>

        <!--trim:可以设置sql语句的前缀,后缀
            suffix：后缀
            prefix：前缀
        -->
        <trim prefix="" prefixOverrides="" suffix="" suffixOverrides="">

        </trim>

        <!--set：用于更新数据库资源-->
        <set>
            <!--写上判断语句-->
        </set>

        <!--foreach：集合遍历，通常构建在in集合里面
            collection:集合名称
            index：集合下标,遍历map时，index为key，item为value
            item：集合单个实例名称
            separator：每个元素之间的分隔符
            #{}：取出遍历值
            open：给遍历出的结果前添加一个字符
            close：给遍历出的结果后添加一个字符

        -->
        <foreach collection="" index="" item="" separator="" open="(" close=")">

        </foreach>
    </select>


    <!--sql：可以做sql抽取，在执行sql内使用include来进行sql引用-->
    <sql id="">
        
    </sql>
    <include refid="sqlId"></include>
```

## 6、缓存

mybatis有两级缓存：

- 一级缓存默认开启(本地缓存)，sqlSession缓存，一直开启，每个sqlSession有自己独立的一级缓存

  - 查取数据库的相同数据，会把查询到的数据缓存到本地缓存中

  - 以后如果需要相同的数据，直接从该缓存中读取

- 二级缓存默认关闭(全局缓存)，namespace级别缓存，每个namespace对应一个二级缓存

  - 查取的数据会放到

```xml
    <!--全局配置文件-->
	<!--mybatis在运行过程中，是否开启某些设置，例如是否开启缓存，驼峰命名法等    -->
    <settings>
        <!-- name:设置项，value:true/false,默认为true-->
        <setting name="cacheEnabled" value="true"/>
    </settings>
	<!--mapper配置文件-->
	<!--cache：开启二级缓存
        eviction：可用的收回策略有：
         LRU – 最近最少使用的：移除最长时间不被使用的对象。
         FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
         SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
         WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
        默认的是 LRU。
        flushInterval：刷新间隔）可以被设置为任意的正整数，
        readOnly：只读
        size：引用数目，默认1024
		type:自定义缓存，type 属性指定的类必须实现org.mybatis.cache.Cache 接口。
    -->
    <cache  eviction="" flushInterval="" readOnly="" size="" type=""></cache>
```

## 7、逆向工程

略过

## 8、MyBatis-运行原理

1. 获取sqlSessionFactory对象
2. 获取sqlSession对象
3. 获取执行mapper对象
4. 执行mapper

**Jdk动态代理：**

```java
public interface MyInf {
    void helloWorld();
}

class MyInfImpl implements MyInf{

    @Override
    public void helloWorld() {
        System.out.println("Hello World!!!");
    }
}
class MyInvocationHandler implements InvocationHandler {

    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    public MyInvocationHandler() {
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法执行前");
        //需要被代理实例和参数
        method.invoke(target,args);
        System.out.println("方法执行后");
        return null;
    }
}

class MyTest{
    public static void main(String[] args) {
        // newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
        /**ClassLoader: 被代理实现类的加载器
         * interfaces： 被代理实现类的接口
         * InvocationHandler：InvocationHandler接口的实现类
         */
        MyInf myInf = (MyInf)Proxy.newProxyInstance(MyInfImpl.class.getClassLoader(), MyInfImpl.class.getInterfaces(), new MyInvocationHandler(new MyInfImpl()));
        myInf.helloWorld();
    }
}
```



### 1、sqlSessionFactory

创建解析器解析配置文件并封装为configuration对象，mapper配置文件中的每一条增删改查封装mapperStatement，**调用构造new DefaultSqlSessionFactory(config);返回**。

![1581349223342](D:\计算机笔记\框架\MyBatis\images\6.png)

### 2.sqlSession

![1581350855352](D:\计算机笔记\框架\MyBatis\images\7.png)

### 3、mapper对象

根据configuration中的mapperRegistry属性，利用jdk动态代理创建mapper

![1581351851140](D:\计算机笔记\框架\MyBatis\images\8.png)

### 4、执行mapper方法



![1581426435417](C:\Users\蒜先生\AppData\Roaming\Typora\typora-user-images\1581426435417.png)

![1581426541491](C:\Users\蒜先生\AppData\Roaming\Typora\typora-user-images\1581426541491.png)

![1581426943420](C:\Users\蒜先生\AppData\Roaming\Typora\typora-user-images\1581426943420.png)

