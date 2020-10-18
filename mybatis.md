# MyBatis

> 小插曲：
>
> log4j配置
>
> ```
> log4j.rootLogger=DEBUG,Console
> log4j.appender.Console=org.apache.log4j.ConsoleAppender
> log4j.appender.Console.layout=org.apache.log4j.PatternLayout
> log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
> log4j.logger.org.apache=INFO
> ```

## 背景

编写sql -> 预编译 -> 设置参数 -> 执行sql -> 封装结果

sql语句编写在java代码存在的问题：硬编码、高耦合

Hibernate：全自动全映射ORM（Object Relation Mapping）框架，它将上述过程全自动处理作为黑箱操作，HQL定制sql语句，旨在消除sql

希望sql语句由开发人员编写，而mybatis将sql与java编码分离，放入配置文件中，后续过程自动完成

## HELLOWORD

**step1**

测试类根据xml配置文件（全局）创建一个SqlSessionFactory对象，以及配置一些运行环境信息

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

**step2**

新建一个xml，作为sql映射文件，配置每一个sql以及sql的封装规则

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="EmpMapper">
    <!--数据字段名若改动，则可以不需要更改Java代码，只需取一个java代码中的变量名作为别名即可-->
    <select id="selectEmp" resultType="demo1.Emp">
        select id,first_name Name,gender from mybatis where id = #{id}
    </select>
</mapper>
```

**step3**

将sql映射文件注册在全局配置mybatis-config.xml中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///book?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="mooc+MOOC"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="EmpMapper.xml"/>
    </mappers>
</configuration>
```

**step4**

```java
SqlSession openSession = sqlSessionFactory.openSession();
try{
    Emp emp = openSession.selectOne("EmpMapper.selectEmp", 1);
    System.out.println(emp);
}finally {
    openSession.close();
}

//省略实体类的成员定义等等。。
```

**step***

考虑到前面会传不符合的数据类型，需要进行确认，特定义一个接口，使得sql配置文件与接口绑定，测试类直接通过接口调用即可。

接口式编程

```
//interface
public interface EmpMapper {
    public Emp getEmpById(Integer id);
}


//EmpMapper.xml
<mapper namespace="config.EmpMapper">
    <!--数据字段名若改动，则可以不需要更改Java代码，只需取一个java代码中的变量名作为别名即可-->
    <select id="getEmpById" resultType="demo1.Emp">
        select id,first_name Name,gender from mybatis where id = #{id}
    </select>
</mapper>


//测试类
try{
	//会为接口自动创建一个代理对象MapperProxy，代理对象去执行增删改查方法
    EmpMapper mapper = openSession.getMapper(EmpMapper.class);
    Emp emp = mapper.getEmpById(1);
    System.out.println(emp);
}
```

SqlSession代表和数据库的一次会话，用完必须关闭，SqlSession和connection一样都是非线程安全的，每次使用都应该获取一个新的对象。

## 全局配置文件

### 别名

在全局配置文件中设置别名处理器

```xml
<typeAliases>
	<!--<typeAlias type="demo1.Emp"/>-->
    <!--为包下所有批量起别名 默认为类名小写-->
    <package name="demo1"/>
</typeAliases>
```

使用批量起名时，可以特定某些类为特定别名

```java
@Alias("empper")
public class Emp {
    ......
}
```

最后在sql映射文件中绑定即可

```xml
<select id="getEmpById" resultType="empper">
    select id,first_name Name,gender from mybatis where id = #{id}
</select>
```

### 运行环境

```xml
<environments default="development">
    <!--id作为唯一标识，可以根据不同的情况选择不同的配置，比如开发和测试配置不同，通过更改default可以达到这个目的-->
    <environment id="test">
        ......
    </environment>
    
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```

### Mappers_SQL映射

```xml
<!--专门弄一个包放置映射文件-->
<mappers>
	<mapper url="..."/><!--引用网络或者磁盘路径下的SQL映射文件-->
    <mapper resource="..."/><!--引用类路径下的SQL映射文件-->
    <mapper class="..."/><!--注册接口，填的是全类名 1、有SQL映射文件，映射文件必须和接口同名，并且放在与接口同一目录下；2、没有SQL映射文件，所有的映射文件SQL都是利用注解（例如@Select("select * from xxx where xxx=#{xxx}")）写在接口上-->
    
    <!--比较重要的，复杂的Dao接口用SQL映射文件，不重要的可以直接用注解-->
    
    <!--批量注册，接口SQLxml映射文件须放在接口同包名下-->
    <package name=""/>
</mappers>
```

## Mapper XML Files

### 增删改查

```java
// EmpMapper接口定义
public interface EmpMapper {
    public Emp getEmpById(Integer id);
    public Emp getEmpByIdAndName(@Param("id") Integer id, @Param("Name") String Name);
    public Emp getEmpByMap(Map<String, Object> map);
    public long addEmp(Emp emp);
    public boolean updateEmp(Emp emp);
    public boolean deleteEmp(Integer id);
}
```

> 要在\<mapper namespace="xxx"></ mapper>"里面定义增删改查的SQL，xxx填接口文件的全类名
>

**select**

```xml
<!--只有一个参数的时候花括号内名字可以随便取-->
<select id="getEmpById" resultType="demo1.Emp">
    select id,first_name Name,gender from mybatis where id = #{id}
</select>

<!--多个参数会被封装成一个map，key:param1...paramN,value:传入的值
    多个参数可以命名参数，明确指定封装参数时map的key
    具体做法为：在参数前加上 @Param("id"),引号内的成分为该参数的命名
-->
<select id="getEmpByIdAndName" resultType="demo1.Emp">
    select id,first_name Name,gender from mybatis where id = #{id} and first_name = #{Name}
</select>

<select id="getEmpByMap" resultType="demo1.Emp">
    <!--#{}里面的东西是map的key-->
    select id,first_name Name,gender from mybatis where id = #{id} and first_name = #{Name}
</select>

<!--若多个参数不是业务模型中的数据，但是经常要用，建议编写一个TO(Transfer Object)数据传输对象:
	Page{
		int index;
		int size;
	}

    # 特别注意：
    如果参数是Collection(List、Set)类型或者是数组，会进行特殊处理，也是把传入的东西封装在map中，
    传过来是Collection，得用#{collection[0]}获取Collection的第一个成员，#{list}、#{set}、#{array}类似
-->

```

**insert**

```xml
<!--全局配置文件中的细节-->
<databaseIdProvider type="DB_VENDOR">
    <!--为不同厂商起别名-->
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle"/>
    <property name="SQL Server" value="sqlserver"/>
</databaseIdProvider>
```

```xml
<!--parameterType可以省略-->
<!--useGeneratedKeys表示用数据库里的主键值
    keyProperty指定mybatis将获取到自增主键的值封装给哪个属性
    databaseId指定为哪一个厂商的别名
-->
<insert id="addEmp" parameterType="demo1.Emp" useGeneratedKeys="true" keyProperty="id" databaseId="mysql">
    insert into mybatis(first_name,gender)
    values(#{Name},#{Gender})
</insert>


<!--不同厂商（Oracle）-->
<insert id="addEmp" databaseId="oracle">
    <!--keyProperty指定mybatis将获取到自增主键的值封装给哪个属性
        BEFORE表示查主键要先于insert
        resultType...
    -->
    <selectKey keyProperty="id" order="BEFORE" resultType="Integer">
        select id.nextval from mybatis
    </selectKey>
    insert into mybatis(id,first_name,gender)
    values(#{id},#{Name},#{Gender})

    <!--AFTER version获取下一个值，然后插入，然后把改过的当前值赋给实体-->
    <!--<selectKey keyProperty="id" order="AFTER" resultType="Integer">-->
    <!--select id.currval from mybatis-->
    <!--</selectKey>-->
    <!--insert into mybatis(id,first_name,gender)-->
    <!--values(id.nextval,#{Name},#{Gender})-->
</insert>
```

**update**

```xml
<update id="updateEmp">
    update mybatis
    set first_name=#{Name},gender=#{Gender}
    where id=#{id}
</update>
```

**delete**

```xml
<delete id="deleteEmp">
    delete from mybatis
    where id=#{id}
</delete>
```

### 映射文件


```java
//接口方法

public List<Emp> getEmpByNameLike(String name);
public Map<String,Object> getEmpByIdReturnMap(Integer id);

//告诉mybatis封装map用哪个属性作为key
@MapKey("Name")
public Map<String,Object> getEmpByNameLikeReturnMap(String name);
```

与之对应的映射文件写法：

```xml
<!--第一条:虽然接受类型是List，但是返回Emp类-->
<!--测试类里传值为"%a%"表示包含字符a-->
<select id="getEmpByNameLike" resultType="demo1.Emp">
    select id,first_name Name,gender from mybatis where first_name like #{name}
</select>


<!--第二条：mybatis已经为了常用的类起了别名-->
<select id="getEmpByIdReturnMap" resultType="map">
    select id,first_name Name,gender from mybatis where id = #{id}
</select>
<!--测试类结果为：{gender=0, id=1, Name=Mary}-->


<!--第三条-->
<select id="getEmpByNameLikeReturnMap" resultType="demo1.Emp">
    select id,first_name Name,gender from mybatis where first_name like #{name}
</select>
<!--结果为多个：key为Name，val为Emp-->
```

**自定义映射规则**

```xml
<!--resultMap和resultType二选一-->

<!--type自定义规则的java类型，id唯一id方便引用-->
<resultMap id="MyEmp" type="demo1.Emp">
    <!--指定主键列的封装规则 column对应哪一列，property对应JavaBean属性-->
    <id column="id" property="id"/>
    <!--定义普通属性，其他不指定的列会自动封装，建议全写方便排查-->
    <result column="first_name" property="Name"/>

    <!--假如Emp里面还有一个属性是其他类Dept(属性有id，name)，属性名为dept-->
    <association property="dept" javaType="demo1.Dept">
        <!--did是指Emp里面有一栏链接到了另一张表，另一张表也有id字段，需要区分-->
        <!--did另一张表里面的d.id字段的简称，这通过Sql语句可以设置，此外还可以通过sql设置表的简称-->
        <id column="did" property="id"/>
        <result column="dept_name" property="name"/>
    </association>
</resultMap>
```

另一种方式（分步查询）：

```xml
<!--同样是Emp里面有Dept属性-->

<resultMap id="MyEmpByStep" type="demo1.Emp">
    ...
    前面都是一样的，关键是association的写法
	这里假定DeptMapper接口有了相关的Dept类的数据库增删改查操作定义
    使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
    d_id是员工表里面最后一个链接部门表的字段
    <association property="dept" select="demo1.DeptMapper.getDeptById" column="d_id">
        
    </association>
</resultMap>
```

懒加载（全局配置里）

需要获取部门信息时才去查询部门表，减少SQL访问

```xml
<setting name="lazyLoadingEnabled" value="true"/>
<setting name="aggressiveLazyLoading" value="false"/>
```

反一下（查询某部门下的所有员工）：

在部门实体上定义了员工list后，映射文件中要添加：

```xml
<resultMap id="MyDept" type="demo1.Dept">
    <id column="did" property="id"/>
    <result column="dname" property="name"/>
    <collection property="emps" ofType="demo1.Emp">
        <id column="eid" property="id"/>
        <result column="ename" property="Name"/>
        <result column="gender" property="Gender"/>
    </collection>
</resultMap>

<select id="getDeptByIdPlus" resultMap="MyDept">
    select d.id did,d.name dname,e.id eid,first_name ename,gender
    from depart d
    left join mybatis e
    on d.id=e.depart
    where d.id=#{id}
</select>
```

**鉴别器discriminator**

```xml
<!--写在resultMap里面-->
<!--        鉴别器，根据不同的查询值做出相应的处理-->
<!--        <discriminator javaType="string" column="gender">-->
<!--            <case value="0" resultType="demo1.Emp">-->
<!--                <result>...</result>-->
<!--                <association>...</association>-->
<!--            </case>-->
<!--        </discriminator>-->
```

## 动态SQL

**注意提交事务！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！**

```xml
<!--where-->
<where>
    <!--where只会去除第一个多出来的and or-->
    <if test="id!=null">
        id=#{id}
    </if>
    <if test="firstName!=null &amp;&amp; firstName!=&quot;&quot;">
        and first_name like #{firstName}
    </if>
    <if test="gender==0 or gender==1">
        and gender=#{gender}
    </if>
</where>


<!--prefix加前缀 prefixOverrides覆盖前缀，去除前缀-->
<trim prefix="where" suffixOverrides="and">
    <if test="id!=null">
        id=#{id} and
    </if>
    <if test="firstName!=null &amp;&amp; firstName!=&quot;&quot;">
        first_name like #{firstName} and
    </if>
    <if test="gender==0 or gender==1">
        gender=#{gender} and
    </if>
</trim>


<select id="getEmpsByConditionChoose" resultType="emp">
    select * from mybatis
    <where>

        <!--bind可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量
                外面直接传不带%的字符串即可-->
        <bind name="_firstName" value="'%'+firstName+'%'"/>

        <!---只会进入一个分支!!!!-->
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="firstName!=null and firstName!=&quot;&quot;">
                first_name like #{_firstName}
            </when>
            <when test="gender!=null">
                gender=#{gender}
            </when>
            <otherwise>
                gender=0 or gender=1
            </otherwise>
        </choose>
    </where>
</select>


<select id="getEmpsByConditionForeach" resultType="emp">
    <!--两个内置参数：_parameter(传过来单个参数，这个就是单个参数，传过来多个参数，就代表一个map；_databaseId：若配置了databaseIdProvider，则可以依次根据不同厂商编写不同的操作逻辑)-->
    select * from mybatis
    <!--collection表示传过来的要遍历的是什么类型，list类型参数会被封装在map中，map的key为list-->
    <foreach collection="list" item="item_id" separator="," open="where id in (" close=")">
        #{item_id}
    </foreach>
</select>


<!--抽取可重用的sql片段，方便以后引用.
    include可以自定义一些property，sql标签内部就能使用自定义的属性，正确的取值方式是${xxx}，不能是#了-->
<sql id="insertColumn">
    <if test="_databaseId==&quot;mysql&quot;">
        first_name,gender,depart
    </if>
</sql>
```

## 缓存

### 两级缓存

> 一级缓存：（本地缓存）SqlSession级别的缓存，一级缓存是一直开启的
>
> > 与数据库同一次会话期间查询到的数据会放在本地缓存中
> >
> > 以后如果需要获取相同的数据，直接从缓存中拿，没有必要再去查询数据库
> >
> > 一级缓存失效情况：SQLSession不同，SQLSession相同但是查询条件不一样，SQLSession相同但是两次查询之间调用了增删改查（可能会对当前数据有影响，就必须更新缓存），SQLSession相同但是缓存被清空
>
> 
>
> 二级缓存：（全局缓存）基于namespace级别的缓存，一个namespace对应一个二级缓存
>
> > 一个会话查询一条数据就会被放到当前会话的一级缓存中，若会话关闭，一级缓存中的数据会被保存到二级缓存中
> >
> > 如果会话关闭，一级缓存中的数据会被保存到二级缓存中，新的会话查询信息就可以参照二级缓存（会话不关的话是放到一级缓存中的哦）
> >
> > sqlSession（EmpMapper===>Emp  DeptMapper===>Dept）不同namespace查出的数据会放到自己对应的缓存中
> >
> > > **开启全局二级缓存配置**
> > >
> > > ```xml
> > > <setting name="cacheEnabled" value="true"/>
> > > ```
> > >
> > > **去mapper.xml里面配置使用二级缓存**
> > >
> > > ```xml
> > > <!--eviction:缓存回收策略（LRU、FIFO、SOFT（软引用，移除基于垃圾回收器状态和软引用规则的对象）、WEAK（更积极地移除基于垃圾收集器状态和弱引用规则的对象））
> > >     flushInterval:缓存刷新间隔
> > >     readOnly:是否只读（只读：mybatis认为所有从缓存中获取数据的操作为只读操作，为了加快获取速度，直接将数据在缓存中的引用交给用户，这不安全，但是速度快
> > >                      非只读：mybatis认为数据可能会被修改，它会利用序列化、反序列化的技术克隆一份新的数据，安全，速度慢）
> > >     size:缓存存放多少元素
> > >     type:制定自定义缓存的全类名，实现Cache接口即可
> > > -->
> > > <cache eviction="FIFO" flushInterval="60000">
> > > 
> > > </cache>
> > > ```
> > >
> > > **POJO需要实现序列化接口（Serializable）**
>
> 和缓存相关的设置：
>
> > 1、cachedEnabled（二级缓存）
> >
> > 2、select标签的useCache（二级缓存）
> >
> > 3、每个增删改标签的flushCache都默认置为true，增删改完成后就会清除缓存，查询默认为false
> >
> > 4、sqlSession.clearCache()只是清除了一级缓存
> >
> > 5、localCacheScope：本地缓存作用域
>
> 总结：会话创建先看二级缓存，没有的话再一级缓存，二级缓存要等会话关闭才会加载新的东西

### 整合缓存