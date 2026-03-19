+++
title = 'Spring Sparrow API'
date = 2026-03-06T18:22:50+08:00
draft = false
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：API 篇章"
+++

## 關注點

1. Spring Framework Api Server 的啟動，實作健康檢查 API，確保服務正常運行。

## 建立 API

### 加入相關依賴

在 `pom.xml` 中加入 `spring-webmvc` 相關依賴：

- **spring-webmvc**：Spring 框架的一部分，提供用於構建基於 Servlet 的 Web 應用程式的功能，包括 RESTful API 的開發支援。
- **jackson-databind**：Jackson 的核心模組，提供將 Java 物件與 JSON 之間進行序列化和反序列化的功能。
- **jakarta.servlet-api**：提供 Servlet API 的定義，允許開發者使用 Servlet 技術來處理 HTTP 請求和響應，適用於基於 Servlet 的 Web 應用程式開發。

ps. 移除 `spring-webmvc` 已包含的相關依賴，例： `spring-context` 避免版本衝突。

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>7.0.5</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.21.1</version>
</dependency>

<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

### 新增 web 設定類別

使用 Java Config 的方式，取代傳統的 XML 設定，

初始化 Spring MVC 的相關設定。

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { AppConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { WebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

啟用 Spring MVC 功能並掃描指定的 base package。

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "sparrow")
public class WebConfig implements WebMvcConfigurer {
    
}
```

JPA 資料庫設定，'資料庫驅動'與'設定讀取'application.properties。

追加 **Class.forName("com.mysql.cj.jdbc.Driver");** // 確保 MySQL 驅動程式被載入。

當應用程式部署到 Tomcat 時，Tomcat 為了確保不同的 Web 應用程式 (WAR) 之間不會互相干擾，採用了階層式的**類別載入器 (ClassLoader) 機制**。

- Bootstrap ClassLoader (底層)：載入 Java 核心類別，包含 java.sql.DriverManager。

- Webapp ClassLoader (應用層)：載入你 WAR 檔裡 WEB-INF/lib 的套件，包含 mysql-connector-j.jar

```java
public DataSource dataSource() {
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
    } catch (ClassNotFoundException e) {
        throw new RuntimeException("無法載入 MySQL 驅動程式", e);
    }
    HikariConfig config = new HikariConfig();
    config.setJdbcUrl(env.getProperty("spring.datasource.url"));
    config.setUsername(env.getProperty("spring.datasource.username"));
    config.setPassword(env.getProperty("spring.datasource.password"));
    return new HikariDataSource(config);
}
```

### 實作 Controller 類別

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class Controller {
    @GetMapping("/status")
    public Map<String, String> getStatus() {
        Map<String, String> status = new HashMap<>();
        status.put("status", "ok");
        return status;
    }
}
```

## Tomcat 服務器

### 打包類型改變

jar -> war

```xml
    <packaging>war</packaging>
```

### 加入 war 套件

為了部署到 Tomcat，我們需要修改 Maven 的打包類型，並加入 maven-war-plugin 外掛。

由於採用了全 Java 設定（無 web.xml），需要將 **failOnMissingWebXml** 設為 **false**。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.4.0</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```

### 使用 Docker 部署 Tomcat 服務器

使用 Docker 與 Docker Compose 來建立本地的 Tomcat 執行環境

#### Dockerfile 設定

使用 OpenJDK 提供的 Ubuntu 基礎映像檔（temurin-noble）。

```dockerfile
FROM tomcat:11.0.18-jdk21-temurin-noble

EXPOSE 8080

CMD [ "catalina.sh", "run" ]
```

#### Docker Compose 設定

建立服務容器，將本機編譯產出的 WAR 檔掛載到 Tomcat 的 webapps 目錄下並命名為 **ROOT.war**（作為根目錄應用程式）。

額外掛載的 **context.xml** 和 **server.xml** 用於自訂 Tomcat 設定。

將 **server.xml** 中的 **autoDeploy** 設為 **true**，Tomcat 將自動部署更新的 WAR 檔而無須重啟容器。

```yaml
legacy:
    build: ./legacy
    ports:
      - "${LEGACY_PORT}:8080"
    networks:
      - spring-sparrow
    volumes:
      - ./legacy/target/legacy-1.0.war:/usr/local/tomcat/webapps/ROOT.war
      - ./legacy/context.xml:/usr/local/tomcat/conf/context.xml:ro
      - ./legacy/server.xml:/usr/local/tomcat/conf/server.xml:ro
    env_file:
      - .env

networks:
  spring-sparrow:
    driver: bridge
```

#### 啟動

```shell
docker compose up -d

mvn clean package
```

### 測試服務

啟動 Docker 容器後，開啟瀏覽器或使用 Postman 訪問 API 端點（假設 LEGACY_PORT 為 8080）：

url: <http://localhost:8080/api/status>

```json
{
    "status": "ok"
}
```

## 參考資料

- [Spring MVC 官方文件](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)
- [Tomcat Image](https://hub.docker.com/_/tomcat)
- [Docker Compose 官方文件](https://docs.docker.com/compose/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
