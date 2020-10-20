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