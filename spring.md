# Spring框架

## IOC（控制反转）

> IOC思想基于IOC容器完成，IOC容器底层是对象工厂

### Spring提供IOC容器的两种基本实现方式（两个接口）：

> 1、BeanFactory：它是spring内部接口，不提供开发人员使用。加载配置文件时不会创建对象，在获取适用对象的时候才去创建对象。
>
> 2、ApplicationContext：它是BeanFactory接口的子接口，提供更多功能。加载配置文件时就会把配置文件对象进行创建。（更好，使得服务器启动时就加载资源）

### Bean管理：Spring创建对象、Spring注入属性

> 两种实现方式：xml、注解方式 

#### xml方式
> id--唯一标识  class--包全路径

##### 创建对象时
默认执行无参构造方法完成对象创建，若定义了有参构造方法而不声明无参方法，调用会报错。

##### 注入属性时

###### set方法进行注入（alt+insert）
>```java
>package spring5;
>
>public class User {
>private String bname;
>private String bauthor;
>
>public void setBname(String bname) {
>this.bname = bname;
>}
>
>public void setBauthor(String bauthor) {
>this.bauthor = bauthor;
>}
>
>public void out() {
>System.out.println(this.bname + this.bauthor);
>}
>}
>```
>
>```xml
><bean id="user" class="spring5.User">
><property name="bname" value="unknown"></property>
><property name="bauthor" value="unknown"></property>
></bean>
>```

###### 用有参构造方法注入
>
>```java
>package spring5;
>
>public class User {
>private String bname;
>private String bauthor;
>
>public User(String bname, String bauthor) {
>    this.bname = bname;
>    this.bauthor = bauthor;
>}
>
>public void out() {
>    System.out.println(this.bname + ' ' + this.bauthor);
>}
>}
>```
>
>```xml
><!--调用有参构造函数-->
><bean id="user" class="spring5.User">
><constructor-arg name="bname" value="Nicky"></constructor-arg>
><constructor-arg name="bauthor" value="Minaj"></constructor-arg>
></bean>
>```

###### p名称空间注入
>
>```xml
><beans xmlns="http://www.springframework.org/schema/beans"
>       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>       xmlns:p="http://www.springframework.org/schema/p"
>       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
>	<bean id="user" class="spring5.User" p:bname="Hello" p:bauthor="World"></bean>
></beans>
>```

**属性值包含特殊符号**：<value><!CDATA[属性值]></value>（给value属性赋值），或者使用转义字符

###### 外部bean注入
>
> 注意bean的定义得是类能生成的，接口没法生成类，只能定义其实现类
>
> ```xml
> <bean id="service" class="spring5.service.service">
>     <!--ref的值要和选定的bean标签的id一致，这就叫做注入外部bean-->
>     <property name="dao" ref="dao"></property>
> </bean>
> <bean id="dao" class="spring5.dao.DaoImpl"></bean>
> ```
>
> 在service中调用接口的实现类对象
>
> ```java
> public class service {
>     private Dao dao;
> 
>     public void setDao(Dao dao) {
>         this.dao = dao;
>     }
> 
>     public void add(){
>         System.out.println("service added.....");
>         dao.update();
>     }
> }
> ```

###### 内部bean
>
> ```xml
> <bean id="emp" class="spring5.bean.Emp">
>     <property name="ename" value="Lily"></property>
>     <property name="gender" value="girl"></property>
>     <property name="dept">
>         <bean id="dep" class="spring5.bean.Dept">
>             <property name="dname" value="Boss"></property>
>         </bean>
>     </property>
> </bean>
> ```
>
> ```java
> public class Dept {
>         private String dname;
> 
>         public void setDname(String dname) {
>             this.dname = dname;
>         }
> 
>         @Override
>         public String toString() {
>             return "Dept{" +
>                     "dname='" + dname + '\'' +
>                     '}';
>         }
> }
> 
> public class Emp {
>         private String ename;
>         private String gender;
> 
>         private Dept dept;
> 
>         public void setDept(Dept dept) {
>             this.dept = dept;
>         }
> 
>         public void setEname(String ename) {
>             this.ename = ename;
>         }
> 
>         public void setGender(String gender) {
>             this.gender = gender;
>         }
> 
>         public void add() {
>             System.out.println(ename + '(' + gender + ") is affiliated to " + dept);
>         }
> }
> ```

###### 级联赋值
>
> ```xml
> <!--method 1-->
> <bean id="emp" class="spring5.bean.Emp">
>     <property name="ename" value="Lily"></property>
>     <property name="gender" value="girl"></property>
>     <property name="dept" ref="dep"></property>
> </bean>
> <bean id="dep" class="spring5.bean.Dept">
>     <property name="dname" value="Boss"></property>
> </bean>
> ```
>
> ```xml
> <!--method 2-->
> <bean id="emp" class="spring5.bean.Emp">
>     <property name="ename" value="Lily"></property>
>     <property name="gender" value="girl"></property>
>     <!--此法要保证Emp中有获取dept的方法getDept，否则对象得不到，值也赋不了-->
>     <!--realBoss覆盖了Boss-->
>     <property name="dept.dname" value="realBoss"></property>
> </bean>
> <bean id="dep" class="spring5.bean.Dept">
>     <property name="dname" value="Boss"></property>
> </bean>
> ```

######  工厂bean（FactoryBean）

Spring有两种bean，一种是普通bean，另外一种是工厂bean

普通bean：配置文件中定义bean类型就是返回类型

工厂bean：配置文件中bean类型可以和返回类型不一样

###### 单个实例和多个实例
scope-singleton表示单实例 -prototype表示多实例
singleton在加载配置文件时就会创建单实例对象，prototype不是在加载配置文件时创建对象，而是在调用getBean的时候创建多实例对象

###### bean生命周期

> 1、通过构造器创建bean实例（无参构造）
>
> 2、为bean的属性设置值和对其他bean引用（set方法）
>
> 3、调用bean的初始化的方法（需要进行配置初始化的方法：创建自定义要初始化的方法，之后在bean标签里设置init-method属性（就写个方法的名字就行了））
>
> 4、可以使用bean
>
> 5、当容器关闭时，调用bean的销毁的方法（销毁的方法要自定义，ApplicationContext的子接口实现类里头有close方法，调用一下，bean里头属性destroy-method设置一下）
>
> 
>
> bean后置处理器（加了之后生命周期变成7步）（处理器实现BeanPostProcessor 复写两个方法，并且在配置文件简单定义）
>
> 1、通过构造器创建bean实例（无参构造）
>
> 2、为bean的属性设置值和对其他bean引用（set方法）
>
> *、把bean实例传递到bean后置处理器的方法中（postProcessBeforeInitialization）
>
> 3、调用bean的初始化的方法（需要进行配置初始化的方法：创建自定义要初始化的方法，之后在bean标签里设置init-method属性（就写个方法的名字就行了））
>
> *、把bean实例传递到bean后置处理器的方法中（postProcessAfterInitialization）
>
> 4、可以使用bean
>
> 5、当容器关闭时，调用bean的销毁的方法（销毁的方法要自定义，ApplicationContext的子接口实现类里头有close方法，调用一下，bean里头属性destroy-method设置一下）

###### xml自动装配.

根据指定的装配规则，Spring自动将匹配的属性值进行装入

> bean标签中的autowire
>
> 1、byName：根据属性名称注入，注入值bean的id值和类属性名称一样
>
> ```java
> private Dep dept; // 这个dept要和下面的id="dept"一致
> ```
>
> ```xml
> <bean id="dept" class="spring5.autoInject.Dep"></bean>
> <bean id="emp" class="spring5.autoInject.Emp" autowire="byName"></bean>
> ```
>
> 2、byType：根据属性类型注入（配置文件定义多个同类型的bean会报错，因为不知道是哪一个匹配，而byName直接根据id去匹配变量名了）

###### 外部属性文件

比如拿数据库类作为例子：

```xml
<!--配置文件-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--引入命名空间context-->
    <context:property-placeholder location="classpath:jdbc.properties" />
    <!--配置连接池 外部文件引入-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClassName}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.username}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
</beans>
```

```properties
# xxx.properties外部定义的属性
prop.driverClassName=com.mysql.jdbc.Driver
prop.url=jdbc:mysql://localhost:3306/xxx
prop.username=xxx
prop.password=xxx
```



#### 注解方式

>  注解是代码里面的特殊标记：@注解名称(属性名称=属性值,属性名称=属性值...)
>
> 注解可以作用在类、方法、属性上面
>
> 注解简化xml配置

Spring针对Bean管理中创建对象提供注解

>@Component
>
>@Service
>
>@Controller
>
>@Repository
>
>\* 四个注解功能一样，都可以用来创建bean实例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--定义命名空间-->
    <!--开启组件扫描！！！！！！！！！！！！！！
        1 多个包中间用逗号隔开
        2 写目标包的上层目录(包括更多的类)
    -->
    <context:component-scan base-package="spring5.notation"></context:component-scan>
</beans>
```

```java
package spring5.notation;

import org.springframework.stereotype.Component;

@Component(value = "note") //<bean id="note" class="..." />不写value则id默认为首字母小写的类名
public class Note {
    public void test(){
        System.out.println("starting note");
    }
}
```

###### 过滤扫描结果

```xml
<!--demo1-->
<!--use-default-filters若置成false，则表示不使用默认filter，自己配置filter
        context:include-filter设置扫描哪些内容，比如下面的例子是扫描带Controller的
    -->
<context:component-scan base-package="spring5.notation" use-default-filters="false">
    <context:include-filter type="annotation"
                            expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--demo2-->
<!--下面扫描包内所有内容
        context:exclude-filter设置哪些内容不进行扫描，比如下面设置不扫描Controller
    -->
<context:component-scan base-package="spring5.notation">
    <context:exclude-filter type="annotation"
                            expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

###### 基于注解方式实现属性注入

> @AutoWired：根据属性类型进行自动装入
>
> @Qualifier：根据属性名称进行注入
>
> @Resource：both（javax包）
>
> @Value：注入普通类型属性

```java
@Repository
public class noteDaoImpl implements noteDao{
    public void add() {
        System.out.println("Dao added......");
    }
}


@Service
public class note {

//    @Autowired //根据类型注入
//    @Qualifier(value = "noteDaoImpl") //根据名称注入
    @Resource(name = "noteDaoImpl") //both
    private noteDao dao;

    public void add() {
        dao.add();
    }
}


//xml里面就进行扫描就好
```

###### 纯注解开发(结合Springboot)

> 1、创建配置类，代替配置文件xml
>
> ```java
> package spring5.config;
> 
> import org.springframework.context.annotation.ComponentScan;
> import org.springframework.context.annotation.Configuration;
> 
> @Configuration //替换xml
> @ComponentScan(basePackages = {"spring5"}) //扫描spring5包下的bean
> public class SpringConfig {
> 
> }
> 
> 
> // 在测试类中不再是xml的路径了，而是配置类的class
> ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
> ```
>
> 

## AOP（面向切面编程）

### 底层原理

Aop底层使用动态代理：

> 1、有接口时，使用JDK动态代理（创建接口实现类代理对象，增强类的方法）
>
> 2、没有接口时，使用CGLIB动态代理（创建当前类子类的代理对象）

#### JDK动态代理


```java
//接口
package spring5_aop.proxy;

public interface UserDao {
    public int add(int a,int b);
    public String update(String id);
}
```

```java
//接口实现类
package spring5_aop.proxy;

public class UserDaoImpl implements UserDao{
    public int add(int a,int b){
        return a+b;
    }

    public String update(String id) {
        return id;
    }
}
```

java.lang.reflect.Proxy调用newProxyInstance方法

三个参数：ClassLoader类加载器、被增强的类所实现的接口（支持多个接口）、实现接口InvocationHandler，创建代理对象

```java
//代理增强
package spring5_aop.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

public class JDKProxy {
    public static void main(String[] args) {
        // 可以传多个接口
        Class[] interfaces = {UserDao.class};
		// 此处接收的接口类型注意一下
        UserDao dao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(new UserDaoImpl()));
        int re = dao.add(1,2);
        System.out.println(re);
    }
}

// 可以定义外部类然后传递对象，也可以在上面直接定义匿名类的对象
class UserDaoProxy implements InvocationHandler {
    private Object obj;

    public UserDaoProxy(Object obj) {
        this.obj = obj;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 可以根据method.getName来根据不同的方法赋予不同的增强逻辑
        System.out.println("Before executed..." + method.getName() + " params:..." + Arrays.toString(args));
		// 执行原来的方法
        Object res = method.invoke(obj,args);

        System.out.println("After executed..." + obj);

        return res;
    }
}
```

> **AOP术语**：
>
> 1、连接点（类里面哪些方法可以被增强，这些方法称为连接点）
>
> 2、切入点（实际被增强的方法，成为切入点）
>
> 3、通知（或增强）（实际增强的逻辑部分成为通知，通知有多种类型：前置、后置、环绕、异常、最终（类似finally）。）
>
> 4、切面（是动作，指把通知应用到切入点的过程）

#### AspectJ

定义被增强的类：

```java
@Component
public class User {
    public void add(int a, int b) {
        System.out.println(a + b);
    }
}
```

多个代理类同时增强：

```java
// 1
@Component
@Aspect //生成代理对象
@Order(3) //等级数值越小级别越高
public class UserProxy {

    //相同切入点抽取
    @Pointcut(value = "execution(* spring5_aop.anno.User.add(..))")
    public void point(){

    }

    //Before注解表示作为前置通知
    @Before(value = "point()")
    public void before() {
        System.out.println("before...");
    }
    //最终通知
    @After(value = "point()")
    public void after(){
        System.out.println("after...");
    }
    //异常
    @AfterThrowing(value = "point()")
    public void afterthrowing(){
        System.out.println("afterthrowing...");
    }
    //后置通知
    @AfterReturning(value = "point()")
    public void afterreturning(){
        System.out.println("afterreturning...");
    }
    //环绕通知
    @Around(value = "execution(* spring5_aop.anno.User.add(..))")
    public void around(ProceedingJoinPoint point) throws Throwable {
        System.out.println("之前");
        point.proceed();
        System.out.println("之后");
    }
}

// 2
@Component
@Aspect
@Order(10)
public class UserProxy_2 {
    @Before(value = "execution(* spring5_aop.anno.User.add(..))")
    public void before(){
        System.out.println("new before");
    }
}
```

配置文件：

```xml
<!--注意名称空间的声明-->
<context:component-scan base-package="spring5_aop.anno"></context:component-scan>

<!--开启aspectj 有注解就生成代理对象-->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

> xml方式（重点掌握注解方式）
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>        xmlns:aop="http://www.springframework.org/schema/aop"
>        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
>                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
> 
>     <bean id="book" class="spring5_aop.xml.Book"></bean>
>     <bean id="bookproxy" class="spring5_aop.xml.BookProxy"></bean>
> 
>     <!--配置aop增强-->
>     <aop:config>
>         <!--切入点-->
>         <aop:pointcut id="p" expression="execution(* spring5_aop.xml.Book.buy(..))"/>
> 
>         <!--切面-->
>         <aop:aspect ref="bookproxy">
>             <!--增强作用在具体的方法上-->
>             <aop:before method="before" pointcut-ref="p" />
>         </aop:aspect>
>     </aop:config>
> </beans>
> ```
>
> 

## JDBCTemplate

封装了jdbc对数据库的操作

```xml
<!--配置文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--组件扫描-->
    <context:component-scan base-package="spring5_jdbc"></context:component-scan>
    <!--数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql:///book?serverTimezone=UTC" />
        <property name="username" value="root" />
        <property name="password" value="xxx" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>
    <!--jdbcTemplate对象 里面包一个德鲁伊-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>
```

实现细节

```java
//entity类
public class Book {
    private String userId;
    private String username;
    private String userstatus;
    
    //此处省略若干set、get方法
	//以及toString方法
}


//接口为BookDao，这里直接实现它
@Repository
public class BookDaoImpl implements BookDao{
    @Autowired
    private JdbcTemplate template;
	//添加
    public void add(Book book) {
        String sql = "insert into t_book values(?,?,?)";
        Object[] args = {book.getUserId(),book.getUsername(),book.getUserstatus()};
        int result = template.update(sql,args);
        System.out.println("result " + result);
    }
	//搜索获取
    public Book search(String id) {
        String sql = "select * from t_book where userId=?";
        //BeanPropertyRowMapper根据返回类型看，返回什么就传什么类
        //结果是根据数据表中的key和对象成员的变量名相匹配而定的，不区分大小写
        Book book = template.queryForObject(sql,new BeanPropertyRowMapper<Book>(Book.class),id);
        //如果是想获取集合的话，用query方法，然后接收的变量为List<Book>，BeanPropertyRowMapper里面类型不变
        return book;
    }
	//获取全体个数
    public int searchAll() {
        String sql ="select count(*) from t_book";
        int result = template.queryForObject(sql,Integer.class);
        return result;
    }
}


//把访问细节封装成服务类共测试调用
@Service
public class BookService {
    @Autowired
    private BookDao bookDao;

    public void addBook(Book book){
        bookDao.add(book);
    }

    public Book searchBook(String id){
        return bookDao.search(id);
    }

    public int searchAll(){
        return bookDao.searchAll();
    }
}
```

批量操作

```java
public void batchAdd(List<Object[]> batchArgs) {
    String sql = "insert into t_book values (?,?,?)";
    int[] ints = template.batchUpdate(sql,batchArgs);
    System.out.println(Arrays.toString(ints));
}
```

## 事务

事务是数据库操作的最基本单元，逻辑上是一组操作，要么都成功，要么一个失败所有失败。

事务的四个特性（ACID）

A：原子性：不可分割，要么成功要么失败

C：一致性：操作之前和操作之后的总量不变

I：隔离性：多事务操作不产生影响

D：持久性：事务提交后表中数据真正发生变化

**注意**

> 事务添加到JavaEE三层结构的Service层（业务逻辑层）
>
> Spring有两种事务管理方式：编程式事务管理、声明式事务管理（√）
>
> 声明式事务管理方式：注解（√）、xml
>
> 声明式事务管理底层使用AOP
>
> Spring事务管理提供一个接口，即事务管理器，这个接口针对不同框架提供不同的实现类
>
> > PlatformTransactionManager
> >
> > 针对jdbc、mybatis提供DataSourceTM，针对Hibernate提供HibernateTM



### 事务实例（转账）

创建数据表

创建service，service注入dao，dao注入JdbcTemplate，JdbcTemplate注入DataSource

> 开启事务
> 业务操作
> 无异常提交事务，有异常事务回滚

事务操作

> xml配置文件中配置事务管理器（注意名称空间tx）
>
> ```xml
> <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
> ```
>
> 开启事务注解@Transactional，多个属性用逗号隔开
>
> > propagation：事务传播行为（7种事务传播行为。。。）
> >
> > ioslation：事务隔离级别，隔离性使得多事务操作不会产生影响，若不考虑隔离性，会产生三个问题：脏读、不可重复读、虚（幻）读
> >
> > > 脏读：一个未提交事务读取了另一个未提交事务的数据
> > >
> > > 不可重复读：一个未提交事务读取到了另一个提交事务所修改的数据
> > >
> > > 虚读：一个未提交事务读取了另一个提交事务所添加的数据
> > >
> > > 通过设置事务隔离级别解决问题 MySQL默认使用REPEATABLE READ（会产生幻读）
> >
> > timeout：超时时间，事务要在一定时间内提交，否则超时会回滚，设置时间以秒为单位
> >
> > readOnly：是否只读
> >
> > rollbackFor：回滚，可以设置出现哪些异常进行事务回滚
> >
> > noRollbackFor：不回滚，可以设置出现哪些异常不进行事务回滚

### XML声明事务管理

配置事务管理器、通知、切入点和切面

```xml
<!--配置通知-->
<tx:advice id="txadvice">
    <tx:attributes>
        <!--可以加入前缀为xxx的方法，当前就只有一个测试方法transaction-->
        <tx:method name="transact*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!--配置切入点和切面-->
<aop:config>
    <aop:pointcut id="p" expression="execution(* spring5_transaction.Service.UserService.*(..))"/>
    <!--通知加入切面-->
    <aop:advisor advice-ref="txadvice" pointcut-ref="p"></aop:advisor>
</aop:config>
```

底层是Aop！！！

