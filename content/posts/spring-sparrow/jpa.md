+++
title = 'Spring Sparrow JPA'
date = 2026-03-04T18:50:49+08:00
draft = false
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：JPA 篇章"
+++

## 關注點

1. 使用 JPA 連接 MySql，連通創建資料。

## 使用 JPA 連接 MySql

### 加入相關依賴

以下是我們需要在 `pom.xml` 中加入的相關依賴：

- **logback-classic**：提供高效能與靈活的日誌記錄功能，為 SLF4J (Simple Logging Facade for Java) 的標準實作。
- **hibernate-core**：提供 Java 物件與關聯式資料庫之間的映射和管理功能（ORM）。它支援 JPA 規範，包含快取、事務管理與查詢語言等功能。`spring-orm` 和 `spring-data-jpa` 皆可透過 Hibernate 作為底層實作。
- **mysql-connector-j**：MySQL 的 JDBC 驅動程式，允許 Java 應用程式與 MySQL 資料庫進行連線與互動。
- **HikariCP**：高效能的 JDBC 連線池實作，提供優異的連線管理與效能最佳化。
- **spring-orm**：Spring 框架的一部分，提供對 ORM 框架的整合支援，結合 Spring 的依賴注入與事務管理功能。
- **spring-data-jpa**：Spring 提供用來簡化 JPA 使用的抽象層，具備自動生成查詢方法、分頁與排序等功能，大幅降低資料庫互動的開發成本。

```xml
    <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.32</version>
    </dependency>

    <dependency>
      <groupId>org.hibernate.orm</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>7.2.5.Final</version>
    </dependency>

    <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <version>9.6.0</version>
    </dependency>

    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>5.1.0</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
      <version>7.0.5</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-jpa</artifactId>
      <version>4.0.3</version>
    </dependency>
```

### 設定資料庫連線

資料庫連線資訊通常統一設定在 application.properties 中。

以下是相關的參數定義：

#### JDBC 與 DataSource 設定

- spring.datasource.url：資料庫的連接 URL，格式為 jdbc:mysql://hostname:port/databaseName?param1=value1&param2=value2。
- spring.datasource.username：資料庫的使用者名稱。
- spring.datasource.password：資料庫的密碼。
- spring.datasource.driver-class-name：JDBC 驅動程式的類名，對於 MySQL 是 com.mysql.cj.jdbc.Driver。

#### Hibernate 與 JPA 設定

- spring.jpa.hibernate.ddl-auto：Hibernate 的 DDL (Data Definition Language) 自動生成策略。設為 update 代表會根據 Entity 自動更新資料庫結構。
- spring.jpa.show-sql：設為 true 時，會在控制台印出 Hibernate 實際執行的 SQL 語句，方便 Debug。

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/dev?useUnicode=true&characterEncoding=UTF-8
spring.datasource.username=account
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### 讀取資料庫連線資訊與配置 Spring 容器

在純 Spring 環境下讀取外部屬性檔案有幾種常見做法：

1. **Environment 物件**：透過注入 Spring 的 **Environment** 物件，使用 getProperty 方法讀取參數值。

2. **@Value 註解**：直接在屬性上標記並讀取 **application.properties** 中的值。

3. **@ConfigurationProperties**：將屬性綁定到特定的 Java 類別中（較常用於 Spring Boot）。

此處我們採用 **Environment 物件** 的方式配置基礎建設。

**Environment** 允許從 **application.properties** 讀取屬性，也支援系統環境變數等來源。

以下是相關程式碼：

```java
import java.util.Properties;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import jakarta.persistence.EntityManagerFactory;

@Configuration
@PropertySource("classpath:application.properties")
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "sparrow.repository")
public class JPAMySqlConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(env.getProperty("spring.datasource.url"));
        config.setUsername(env.getProperty("spring.datasource.username"));
        config.setPassword(env.getProperty("spring.datasource.password"));

        return new HikariDataSource(config);
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
        factoryBean.setDataSource(dataSource());

        factoryBean.setPackagesToScan("sparrow.entity");

        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        factoryBean.setJpaVendorAdapter(vendorAdapter);

        Properties props = new Properties();

        props.put("hibernate.show_sql", env.getProperty("spring.jpa.show-sql"));
        props.put("hibernate.hbm2ddl.auto", env.getProperty("spring.jpa.hibernate.ddl-auto"));
        props.put("hibernate.dialect", "org.hibernate.dialect.MySQLDialect");

        factoryBean.setJpaProperties(props); // 設定 Hibernate 的相關屬性，顯示 SQL、DDL 自動生成策略和方言等
        return factoryBean;
    }

    @Bean
    public PlatformTransactionManager transactionManager(
            EntityManagerFactory emf) {
        if (emf != null) {
            return new JpaTransactionManager(emf);
        }
        throw new IllegalArgumentException("EntityManagerFactory cannot be null");
    }
}
```

## 新增一個使用者

### Entity 類別實作

**Entity** 是記憶體中的 Java 類別，透過 JPA 註解與資料庫中的資料表進行映射綁定。

```java
@Entity
@Table(name = "users")
public class User {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, length = 50)
  private String name;

  @Column(nullable = false, length = 50)
  private String password;

  @Column(nullable = false, length = 50)
  private String email;

  @Column(columnDefinition = "TEXT")
  private String description;

  // 必須提供無參數的建構子，JPA 需要使用反射來創建實體對象
  public User() {
  }

  // getter 和 setter 方法，用於訪問和修改實體在記憶體中的屬性
  // ... 省略 getter 和 setter 方法 ...
}
```

### Repository 類別實作

宣告 Repository 介面並繼承 JpaRepository。

Spring Data JPA 會在執行時期自動提供其實作類別，開發者無須手動撰寫基礎的 CRUD 邏輯。

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import sparrow.entity.User;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // 預設已繼承標準的 CRUD 方法
    // 若有複雜查詢需求，可依照 Spring Data 命名規範自訂方法名稱
}
```

### 實際運作

這是一個將 Entity 寫入資料庫的簡單範例。

```java
 GenericApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

    try {
      UserRepository userRepository = context.getBean(UserRepository.class);

      System.out.println("正在執行...");
      User newUser = new User();

      newUser.setPassword("password");
      newUser.setName("Tester");
      newUser.setEmail("test@sparrow.dev");
      newUser.setDescription("這是一個測試使用者");

      userRepository.save(newUser);
      System.out.println("使用者已儲存，ID 為: " + newUser.getId());

      List<User> users = userRepository.findAll();
      System.out.println("目前資料庫中的使用者總數: " + users.size());
      users.forEach(u -> System.out.println(" - " + u.getName() + " (" + u.getEmail() + ")"));

    } catch (Exception e) {
      System.err.println("發生錯誤:");
      e.printStackTrace();
    } finally {
      System.out.println("關閉 Spring 容器...");
      context.close();
    }
```

## 參考資料

[Spring Data JPA Documentation](https://docs.spring.io/spring-data/jpa/reference/)

[JpaRepository API](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)

[Hibernate ORM Documentation](https://hibernate.org/orm/documentation/7.2/)

[MAVEN Dependency](https://mvnrepository.com/)
