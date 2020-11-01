# SpringBoot

简化Spring应用开发的一个框架，整个Spring技术栈的一个大集合，J2EE开发的一站式解决方案

## 微服务

一个应用应该是一组小型服务，可以通过http进行互通

每一个功能元素最终都是一个可独立替换和独立升级的软件单元

## HelloWorld

导包配环境之后：

### 编写一个主程序

```java
/**
 * @SpringBootApplication 标注一个主程序类，说明这是一个Spring boot应用
 */
@SpringBootApplication
public class HelloWorld {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorld.class, args);
    }
}
```

### 编写相关Controller、Service

```java
@Controller
public class HelloWorldController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

### 运行主程序

### 简化部署

```xml
<!--pom里面加入这个插件，可以将应用打包成一个可执行的jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

打成jar包后，直接使用java -jar命令进行执行

### 解释

#### pom.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<!--父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
作为Spring Boot的版本仲裁中心-->


<!--spring-boot-starter:spirng-boot场景启动器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 主程序类

```java
@SpringBootApplication
public class HelloWorld {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorld.class, args);
    }
}
```

@SpringBootApplication：Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用

@SpringBootApplication是一个组合注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration 
@EnableAutoConfiguration //自动配置功能
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

##### @SpringBootConfiguration 

标注在某个类上表示是Spring Boot的配置类(@Configuration)，配置类也是容器中的一个组件(@Component)

##### @EnableAutoConfiguration

自动配置功能

```java
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

###### @AutoConfigurationPackage

自动配置包

```
@Import({Registrar.class})
```

@Import是Spring底层注解，它给容器导入一个组件，导入的组件由Registrar.class确定

**将主配置类（@AutoConfigurationPackage标注的类）所在包及下面所有子包**里面所有组件扫描到Spring容器

> EnableAutoConfigurationImportSelector：将所有需要导入的组件以全类名的方式返回，这会给容器中导入非常多的自动配置类，即给容器中导入这个场景需要的所有组件，免去了手动配置的工作。
>
> > SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
> >
> > Spring boot 在启动时从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中
> >
> > J2EE整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar里面

## Spring Initializer快速创建项目

勾选需要的项目支持，pom文件就会相应添加好库，

@ResponseBody、@Controller可以合并写为@RestController

### resources文件夹中的目录结构

* static：保存所有静态资源（js、css、img）

* template：保存所有的模板页面（SpringBoot默认jar包使用嵌入式的tomcat，默认不支持JSP页面，但可以使用模板引擎freemarker、thymeleaf）

* application.properties：SpringBoot应用的配置文件

## Spring Boot配置

### 配置文件

全局配置文件，配置文件名是固定的

* application.properties
* application.yml

yml是YAML语言的文件，以数据为中心，比json和xml更适合做配置文件

### YAML

#### 基本语法

**key:(space)value**（***<u>==Space is required！==</u>***它以空格的缩进来控制层级关系）

属性和值大小写敏感

#### 值的写法

##### 字面量（数字、字符、布尔）

k: v（字符串默认不用加上引号）

​		双引号：会转义字符串里面的特殊字符，比如\n转移为换行

​		单引号：不会转义字符串里面的特殊字符，是什么样就输出什么样

##### 对象、Map

k: v（在下一行来写对象的属性和值的关系，要注意缩进！）

对象里面还是k: v的模式

```yml
friends:

	lastName: xx

	age: xx

# 行内写法
friends: {lastName: xx,age: xx}
```

##### 数组（List、Set）

用- 值来表示数组中的一个元素

```yaml
pets: 
	- cat
	- dog
	- pig

# 行内写法
pets: [cat,dog,pig]
```

### Properties

中文乱码问题：settings 》 file encoding 》 properties files（改成utf-8，方框勾上）

装载数据的时候会优先使用properties里面的参数，若无，则用yml里面的补上

### 配置文件注入

#### @ConfigurationProperties、  @Value

|                    | @ConfigurationProperties                                     | @Value                 |
| :----------------- | ------------------------------------------------------------ | ---------------------- |
| 功能               | 批量注入属性                                                 | 单个注入               |
| 松散绑定           | 支持（驼峰命名：配置文件中可以写xxx-xxx，命名为xxxXxx）      | 不支持                 |
| SpEL               | 不支持                                                       | 支持（值可以直接计算） |
| JSR303数据校验     | 支持(类上加@Validated，里面可以加@Email之类的，验证格式是否对的) | 不支持                 |
| 复杂类型封装（类） | 支持                                                         | 不支持                 |

如果说只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value

如果说专门有一个javaBean来和配置文件进行映射，就直接使用@ConfigurationProperties  

#### @PropertySource、  @ImportResource

**@PropertySource**读取指定路径下的properties文件，可以多个（@PropertySource(value = {"classpath:person.properties"})）

**@ImportResource**导入Spring配置文件，让配置文件里面的内容有效

> * Spring Boot里面没有Spring配置文件，自己编写的文件不能被自动识别，若想要生效，用@ImportResource标注在一个配置类上（@ImportResource(locations = {"classpath:beans.xml"})）

> * Spring Boot推荐给容器中添加组件的方式：推荐使用全注解方式

```java
/**
 * 指明为配置类
 * 在配置文件中使用<bean><bean/>来添加组件，对应地，在此处用@Bean来表示
 */
@Configuration
public class MyConfig {

    @Bean
    public HelloService helloService(){
        System.out.println("@Bean给容器中添加组件啦~");
        return new HelloService();
    }
}
```

### 配置文件占位符

#### 随机数

```
${random.uuid}
${random.int}
${random.int(10)}
${random.int[2024,2000]}
...
```

#### 占位符获取之前的值

```properties
person.dog.name=${person.hello:hello}'s dog 
# 表示若获取不到person.hello就把hello当做默认值，否则没有的话输出是person.hello
```

没有的话用冒号后面的值

### Profile

* #### 多profile文件

  主配置文件的文件名可以使application-{profile}.properties/yml

  默认使用application.properties的配置

* #### yml支持多文档块方式

  三个横杠（---）回车后分隔开了一个文档块

  若有需要，可以在第一个文档块中指定要激活的文档块

  ```yaml
  spring:
    profiles:
      active: dev
  ```

  其他文档块的命名为

  ```yaml
  spring:
    profiles: dev
  ```

* #### 激活制定profile

  * 在配置文件中制定spring.profiles.active=dev

  * 命令行：

    添加Run/Debug配置的程序运行参数（Program arguments）--spring-profiles-active=dev

    也可以在打包之后用命令行启动并添加参数==java -jar xxx.jar --spring.profiles.active=dev==

  * 虚拟机参数（VM options）-Dspring.profiles.active=dev

### 配置文件加载位置

springboot启动会扫描以下位置的application.properties或者application.yml文件作为Spring Boot的默认配置文件（以下为优先级从高到低，高优先级会覆盖低优先级的相同配置，形成**互补配置**）

* 项目路径/config/application.properties
* 项目路径/application.properties
* 类路径/config/application.properties
* 类路径/application.properties

可以通过spring.config.location来改变默认的配置文件位置（项目打包以后，可以使用命令行参数的形式启动项目的时候来指定配置文件的新位置，指定配置文件会和默认加载的配置文件一起作用）**（运维使用）**



配置修改项目的访问路径（在配置文件中加）：server.context-path=/xxx

#### 外部配置加载顺序（按照优先级）（常用，不全）

* 命令行参数 java -jar xxx.jar --server.port=8082（这样的话，启动的端口号为8082）

**由jar包外向jar包内的顺序**

* 带profile的配置文件
  * jar包外部的application-{profile}.properties或application.yml（有spring.profile）
  * jar包内部的application-{profile}.properties或application.yml（有spring.profile）

* 不带profile的配置文件
  * jar包外部的application.properties或application.yml
  * jar包内部的application.properties或application.yml

* @Configuration注解类上面的@PropertySource
* 通过SpringApplication.setDefaultProperties指定的默认属性

其他详见官方文档！！！

### ==[自动配置原理](https://www.bilibili.com/video/BV1Et411Y7tQ?p=18)==（再看一遍！！！）

[官方文档（涉及的配置）](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties)

* SpringBoot启动时加载主配置类，开启了自动配置功能==@EnableAutoConfiguration==

* @EnableAutoConfiguration：

  * 使用AutoConfigurationImportSelector给容器中导入一些组件

    可以查看selectImports（getAutoConfigurationEntry）的代码看看导入了什么

    ```java
    List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);//获取候选的配置
    ```

    ```java
    SpringFactoriesLoader.loadFactoryNames();
    //扫描所有jar包类路径下面的 META-INF/spring.factories
    //把扫描到的文件内容包装成properties对象
    //从properties中获取到EnableAutoConfiguration.class类名对应的值，然后添加在容器中
    ```

* 每一个自动配置类进行自动配置功能

  * e.g.

    ```java
    @Configuration(proxyBeanMethods = false)
    //表征这是一个配置类
    
    @EnableConfigurationProperties({ServerProperties.class})
    //启用ConfigurationProperties功能，将配置文件中对应的值和ServerProperties绑定起来，并加入到ioc容器中
    
    @ConditionalOnWebApplication(type = Type.SERVLET)
    //Spring底层注解@Conditional，根据不同的条件，若满足指定的条件，整个配置类里面的配置就会生效（目前是判断是否为Web应用）
    
    @ConditionalOnClass({CharacterEncodingFilter.class})
    //判断当前项目有没有这个类，CharacterEncodingFilter：Spring MVC中进行乱码解决的过滤器
    
    @ConditionalOnProperty(prefix = "server.servlet.encoding",value = {"enabled"},matchIfMissing = true)
    //判断配置文件中是否存在某个配置server.servlet.encoding，“matchIfMissing = true”表示若不存在则也认为判断是成立的
    //即使配置文件中不配置server.servlet.encoding=true，也是默认生效的
    
    public class HttpEncodingAutoConfiguration {
        
        //他已经和SpringBoot配置文件映射了
        private final Encoding properties;
        
        //只有一个有参构造器的情况下，参数的值就会从容器中获取
        public HttpEncodingAutoConfiguration(ServerProperties properties) {
            this.properties = properties.getServlet().getEncoding();
        }
    
        
        
        //判断通过后生成一个bean，但bean中的某些值由properties确定
        @Bean
        @ConditionalOnMissingBean
        public CharacterEncodingFilter characterEncodingFilter() {
            CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
            filter.setEncoding(this.properties.getCharset().name());
            filter.setForceRequestEncoding(
            	this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.REQUEST));
            filter.setForceResponseEncoding(
                this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.RESPONSE));
            return filter;
        }
    }
    ```

    根据当前不同的条件判断，决定这个类是否生效

    一旦配置类生效，这个配置类就会给容器中添加各种组件，这些组件是从对应的properties中获取的，这些类里面的每一个属性都是和配置文件绑定的

    所有配置文件中能配置的属性都是在xxxProperties类中封装着，配置文件能配置什么就可以参照某个功能对应的这个属性类

    ```java
    @ConfigurationProperties(
        prefix = "server",
        ignoreUnknownFields = true
    )
    //从配置文件中获取指定的值和bean的属性进行绑定
    
    public class ServerProperties {}
    ```

精髓：

* SpringBoot启动会加载大量的自动配置类
* 我们看需要的功能有没有SpringBoot默认写好的自动配置类
* 有的话再来看这个自动配置类中到底配置了哪些组件（组件有，就不需要自己配置了）
* 组件没有的话要自己配置，给容器中自动配置类添加组件的时候，会从properties中获取某些属性，我们就可以在配置文件中指定这些属性的值

xxxAutoConfiguration：自动配置类，会给容器中添加组件

xxxProperties：封装配置文件中相关属性



