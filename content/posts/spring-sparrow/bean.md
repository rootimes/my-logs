+++
title = 'Spring Sparrow Start'
date = 2026-03-04T11:00:41+08:00
draft = true
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：初始篇章"
+++

## 關注點

1. Spring Framework 的服務啟動流程：從建立 ApplicationContext 到註冊 Bean，再到啟動服務的整個過程。

## 啟動專案

### maven cli 建立

```shell
mvn archetype:generate -DgroupId=sparrow -DartifactId=legacy -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

- DgroupId : 組織名稱
- DartifactId : 專案名稱
- DarchetypeArtifactId : 專案模板
- DinteractiveMode : 跳出詢問視窗

### 加入 Spring Framework 相關依賴

在 `pom.xml` 中加入 `spring-context`，
它會自動包含 `spring-core`、`spring-beans` 與 `spring-aop` 等核心模組：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>7.0.5</version>
    </dependency>
</dependencies>
```

### 嘗試編譯

```bash
cd legacy # 進入 artifactId 定義的資料夾
mvn compile
```

## 創建一個 Bean

Spring 的核心在於 **IoC (Inversion of Control, 控制反轉)** 與 **DI (Dependency Injection, 依賴注入)**。

開發者將物件的生命週期管理交給容器（IoC），並透過注入的方式解決物件間的依賴關係（DI）。

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

表示 Spring 需要掃描指定 package 下的類，並將帶有以下註解的類註冊為 Bean：

- **@Component**：最基礎的組件註解。
- **@Service**：表示業務邏輯層。
- **@Repository**：表示數據訪問層（DAO），並具備數據庫異常轉換功能。
- **@Controller**：表示控制層（MVC）。

這些註解本質上都是 `@Component` 的特化（Stereotype）。

```java
@Configuration
@ComponentScan(basePackages = "sparrow")
public class AppConfig {
}
```

### @Autowired

表示 Spring 會自動注入依賴的 Bean。

建議優先使用 **建構子注入 (Constructor Injection)**，因為它能確保依賴項不為 null 且方便進行單元測試。

*註：從 Spring 4.3 開始，如果類別只有一個建構子，`@Autowired` 註解可以省略。*

```java
@Service
public class OrderService {
    private final UserService userService;

    // 只有一個建構子時，@Autowired 可省略
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

## 參考

[Spring Beans 官方文檔](https://docs.spring.io/spring-framework/reference/core/beans.html)
