+++
title = 'Spring Sparrow'
date = 2026-03-04T11:00:41+08:00
draft = true
tags = [
    "spring",
]
description = "這是一則描述"
+++

## 啟動專案

### maven cli 建立

```shell
mvn archetype:generate -DgroupId=sparrow -DartifactId=legacy -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

- DgroupId : 組織名稱
- DartifactId : 專案名稱
- DarchetypeArtifactId : 專案模板
- DinteractiveMode : 跳出詢問視窗

### 加入 Spring dependency

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.4</version>
    </dependency>
</dependencies>
```

### 嘗試編譯

```bash
cd folder
mvn compile
```

## 創建一個 Bean

### GenericApplicationContext

最基礎、最通用的 Spring 容器實作

```java

GenericApplicationContext context = new GenericApplicationContext(); // 建立容器

context.registerBean(String.class, () -> "Hello Bean"); // 手動 registerBean

context.refresh(); // 刷新容器，完成註冊

String bean = context.getBean(String.class); // 獲取 Bean

context.close();
```

### AnnotationConfigApplicationContext

專門用來處理 Annotation 的 Spring 容器實作

支持 @Configuration、@Component、@Bean、@ComponentScan、@Autowired 等註解

註解稍後再說明

```java
// App.java
AnnotationConfigApplicationContext context =
    new AnnotationConfigApplicationContext(AppConfig.class);

HelloBean bean = context.getBean(HelloBean.class);
        
System.out.println(bean.helloBean());

context.close();
```

```java
// AppConfig.java
@Configuration
public class AppConfig {
    @Bean
    public HelloBean helloBean() {
        return new HelloBean();
    }
}

// HelloBean.java
public class HelloBean {
    public String helloBean() {
        return "Hello Bean";
    }
}
```

### ClassPathXmlApplicationContext

專門用來處理 XML 配置的 Spring 容器實作

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
```

```xml
<bean id="userService" class="sparrow.MessageService"/>
```

### FileSystemXmlApplicationContext

專門用來處理 XML 配置的 Spring 容器實作，與 ClassPathXmlApplicationContext 類似，但從文件系統路徑讀取配置文件

```java
ApplicationContext context = new FileSystemXmlApplicationContext("config/beans.xml");
```

### WebApplicationContext

spring web 模組專用的 Spring 容器實作，提供了 Web 相關的功能

```java
WebApplicationContext context = new AnnotationConfigWebApplicationContext();
context.register(AppConfig.class);
```

## Spring Framework 的註解

### @Configuration

表示這個類是 Spring 的配置類，類似於 XML 配置文件，可以用來定義 Bean 和其他配置

```java
@Configuration
public class AppConfig {
    @Bean
    public HelloBean helloBean() {
        return new HelloBean();
    }
}
```

### @Bean

表示這個方法會返回一個 Bean，Spring 會將這個 Bean 註冊到容器中，方法名稱默認為 Bean 的名稱

```java
@Bean
public HelloBean helloBean() {
    return new HelloBean();
}
```

### @Component

表示這個類是一個 Spring 管理的組件，Spring 會自動掃描並註冊這個類為 Bean

```java
@Component
public class HelloBean {
    public String helloBean() {
        return "Hello Bean";
    }
}
```

### @ComponentScan

表示 Spring 需要掃描指定 package 下的類，

將以下幾種註解的類註冊為 Bean：

- @Component
- @Service
- @Repository
- @Controller

```java
@Configuration
@ComponentScan(basePackages = "sparrow")
public class AppConfig {
}
```

### @Autowired

表示 Spring 會自動注入依賴的 Bean，可以用在構造器、方法或字段上

```java
@Service
public class OrderService {
    private final UserService userService;

    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

## 參考

- [Spring Framework 官方文檔](https://spring.io/projects/spring-framework)