+++
title = 'Spring Sparrow REST Docs'
date = 2026-03-08T22:08:04+08:00
draft = true
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：REST Docs 篇章"
+++

## 關注點

1. Spring REST Docs 服務的啟動，生成 API 文件，並將文件部署到網頁上供外部訪問。

## Spring REST Docs

### 什麼是 Spring REST Docs？

Spring REST Docs 是由 Spring 官方提供且維護的工具，

結合了測試驅動開發（TDD）的理念，透過執行測試來自動生成準確的 RESTful API 文件，確保程式碼與文件始終保持同步。

### 加入相關依賴

- **junit-jupiter**: 提供 JUnit 5 的核心測試功能，支持撰寫與執行單元測試和整合測試。
- **spring-test**: 提供對 Spring 應用程式的測試支援，包括模擬 Web 環境、測試上下文管理等。
- **spring-restdocs-mockmvc**: Spring REST Docs 的 MockMvc 模組，允許在測試中截獲 API 請求與回應的細節，並生成 API 文件片段 (snippets)。

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>6.0.3</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>7.0.5</version> <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <version>3.0.1</version> <scope>test</scope>
</dependency>
```

### 加入 Asciidoctor 套件

- **asciidoctor-maven-plugin**: 將 AsciiDoc 文件轉換為 HTML、PDF 等格式的功能，支持在 Maven 建置過程中自動生成文件。
- **spring-restdocs-asciidoctor**: 提供與 Asciidoctor 套件整合的功能，讓產生的片段能以 AsciiDoc 格式進行排版。

```xml
<properties>
    <snippetsDirectory>${project.build.directory}/generated-snippets</snippetsDirectory>
</properties>

<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
      <execution>
        <id>generate-docs</id>
        <phase>prepare-package</phase>
        <goals>
          <goal>process-asciidoc</goal>
        </goals>
        <configuration>
          <backend>html</backend>
          <doctype>book</doctype>
          <attributes>
            <snippets>${snippetsDirectory}</snippets>
          </attributes>
          <sourceDirectory>src/main/asciidoc</sourceDirectory>
          <outputDirectory>target/generated-docs</outputDirectory>
        </configuration>
      </execution>
    </executions>
    <dependencies>
      <dependency>
        <groupId>org.springframework.restdocs</groupId>
        <artifactId>spring-restdocs-asciidoctor</artifactId>
        <version>4.0.0</version>
      </dependency>
    </dependencies>
</plugin>
```

### 撰寫測試範例

#### RestDocsTest 父類別實作

抽象出共用的測試設定與工具方法。

設定 MockMvc 並整合 Spring REST Docs。

```java
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.restdocs.RestDocumentationContextProvider;
import org.springframework.restdocs.RestDocumentationExtension;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@ExtendWith({ SpringExtension.class, RestDocumentationExtension.class })
@WebAppConfiguration
public class RestDocsTest {
    protected MockMvc mockMvc;

    @BeforeEach
    public void setUp(WebApplicationContext webApplicationContext,
            RestDocumentationContextProvider restDocumentation) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .apply(documentationConfiguration(restDocumentation))
                .build();
    }
}
```

#### AppTest 類別實作

撰寫具體的測試方法，使用 MockMvc 執行 API 請求並自動產生文件片段。

```java
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.payload.PayloadDocumentation.fieldWithPath;
import static org.springframework.restdocs.payload.PayloadDocumentation.responseFields;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import sparrow.config.WebConfig;


import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

import sparrow.controller.Controller;

@ContextConfiguration(classes = { WebConfig.class })
public class AppTest extends RestDocsTest {
    @Test
    public void documentGetStatusApi() throws Exception {
        this.mockMvc.perform(get("/api/status"))
                .andExpect(status().isOk())
                .andDo(document("status-get",
                        responseFields(
                                fieldWithPath("status").description("API 處理狀態"),
                                fieldWithPath("message").description("詳細回應訊息"))));
    }
}
```

### AsciiDoc 文件撰寫

在專案根目錄建立 **src/main/asciidoc/** 資料夾，並新增 **index.adoc** 檔案。
這個檔案是測試片段的模板文件，包含了目錄結構與內容說明。

設定好後，當執行 Maven 的 **package** 階段時，
**asciidoctor-maven-plugin** 會自動將 **index.adoc** 轉換為 HTML 文件，並將測試中產生的片段 (snippets) 插入對應位置。

在 **target/generated-docs/** 目錄下會生成最終的 **index.html**，用於本地預覽或部署到伺服器。

#### 標籤說明

- toc: left：將目錄放置在頁面左側。
- toclevels: 3：設定目錄的層級深度為 3
- sectnums：啟用章節自動編號。

```asciiDoc
= Sparrow Legacy API Guide
:toc: left
:toclevels: 3
:sectnums:

== Introduction
Sparrow 的 API 文件

== Status API
This API returns the current status of the application.

=== HTTP Request 範例
include::{snippets}/status-get/http-request.adoc[]

=== cURL Request 範例
include::{snippets}/status-get/curl-request.adoc[]

=== HTTP Response 範例
include::{snippets}/status-get/http-response.adoc[]

=== 回應欄位說明
include::{snippets}/status-get/response-fields.adoc[]
```

## 文件網頁

### 加入資源管理套件

- **maven-resources-plugin**: 將 Asciidoctor 生成的 HTML 文件複製到 Spring MVC 的靜態資源目錄中。

```xml
<plugin>
  <artifactId>maven-resources-plugin</artifactId>
  <version>3.4.0</version>
  <executions>
    <execution>
      <id>copy-resources</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>copy-resources</goal>
      </goals>
      <configuration>
        <outputDirectory>${project.build.outputDirectory}/static/docs</outputDirectory>
        <resources>
          <resource>
            <directory>${project.build.directory}/generated-docs</directory>
          </resource>
        </resources>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 在 WebConfig 中加入資源處理器

**@EnableWebMvc** 註解會啟動一套預設的 Web 配置。

為了讓 Spring MVC 能對外提供我們打包好的靜態 HTML 文件，必須實作 **WebMvcConfigurer** 來註冊靜態資源路徑。

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "sparrow")
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/docs/**")
                .addResourceLocations("classpath:/static/docs/");
    }
}
```

### 測試網頁

在瀏覽器中訪問 <http://localhost:8080/docs/index.html> 就可以看到生成的 API 文件網頁了。

## 參考資料

- [Spring REST Docs 官方文件](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/)
- [Asciidoctor 語法快速參考指南 (AsciiDoc Syntax Quick Reference)](https://docs.asciidoctor.org/asciidoc/latest/syntax-quick-reference/)
- [JUnit 5 User Guide (官方測試框架文件)](https://junit.org/junit5/docs/current/user-guide/)
- [Spring Web MVC 官方文件 (靜態資源處理與 WebMvcConfigurer)](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config.html)
- [Apache Maven Resources Plugin 官方文件](https://maven.apache.org/plugins/maven-resources-plugin/)
- [Asciidoctor Maven Plugin 官方文件](https://docs.asciidoctor.org/maven-tools/latest/plugin/introduction/)
