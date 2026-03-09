+++
title = 'Spring Sparrow AOP'
date = 2026-03-04T16:44:22+08:00
draft = true
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：AOP 篇章"
+++

## 使用 AOP

AOP (Aspect-Oriented Programming, 切面導向程式設計) 允許開發者將「切面關注點」（如日誌、安全、事務管理）從業務邏輯中分離。

在開始之前，我們先釐清幾個核心術語：

- **Aspect (切面)**：橫切關注點的模組化（如：`LoggingAspect`）。
- **Join Point (連接點)**：程式執行中的特定點（在 Spring 中通常指方法呼叫）。
- **Advice (通知)**：在連接點執行的動作（如：`@Before`, `@After`, `@Around`）。
- **Pointcut (切入點)**：定義 Advice 應該在哪些連接點執行的表達式。

### 加入 Spring AOP 依賴

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>7.0.5</version>
    </dependency>
</dependencies>
```

### 定義一個切面

```java
package com.example;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {
    // 定義切入點：當 sparrow 套件下的 MessageService 類別的 getMessage() 被呼叫前
    @Before("execution(* sparrow.MessageService.getMessage(..))")
    public void logBefore() {
        System.out.println("LoggingAspect: Before executing getMessage()");
    }
}
```

### 開啟 AOP 功能

```java
@Configuration
@ComponentScan(basePackages = "sparrow") // 掃描 sparrow 套件下的元件
@EnableAspectJAutoProxy // 啟用 AOP 功能
public class AppConfig {
    @Bean
    public MessageService messageService() {
        return new MessageService();
    }
}
```

## AOP 的運作流程

1. 當 `messageService.getMessage()` 被呼叫時，Spring AOP 會攔截這個方法呼叫。
2. 在方法執行前，`LoggingAspect.logBefore()` 會被執行，輸出 "LoggingAspect: Before executing getMessage()"。
3. 最後，執行 `messageService.getMessage()` 的原始邏輯。

### 設計模式 Proxy Design Pattern (代理模式)

AOP 的實作原理是基於代理模式。Spring AOP 會在執行時期根據情況選擇不同的代理機制：

1. **JDK Dynamic Proxy**：若目標類別實作了介面，Spring 預設使用 JDK 動態代理。
2. **CGLIB**：若目標類別沒有實作介面，Spring 會透過 CGLIB 在記憶體中建立目標類別的子類別來實作代理。

代理物件會攔截對目標物件的方法呼叫，並在適當的時機執行切面邏輯。

## 衍伸應用

### 事務管理 (Transaction Management)

AOP 可以用來實作事務管理，當方法執行時自動開始事務，執行完成後自動提交或回滾。

```java
@Transactional
public void exec() {
    // 這裡的邏輯會在事務中執行
    // 如果發生例外，事務會自動回滾
}
```

### 快取處理 (Caching)

AOP 可以實作快取機制，呼叫方法前先檢查快取，若有結果則直接回傳。

- **@Cacheable**：方法結果會被快取。
- **@CacheEvict**：清除特定快取資料。
- **@CachePut**：更新快取資料。

### 權限控制

結合 Spring Security 實作細粒度的權限控制：

- **@PreAuthorize**：方法執行前檢查權限。
- **@PostAuthorize**：方法執行後檢查回傳結果。
- **@Secured**：基於角色的傳統權限檢查。

*註：在 Spring Boot 3 / Spring Security 6 中，建議使用 **`@EnableMethodSecurity`** 來啟用此功能。*

```java
@PreAuthorize("hasRole('ADMIN')")
public void adminOnlyMethod() {
    // 只有具備 ADMIN 角色的使用者才能執行
}
```

### 非同步處理

透過 AOP 實作非同步呼叫，方法會在新的執行緒中執行，不會阻塞主程式。

```java
@Async
public void asyncMethod() {
    // 此邏輯會非同步執行
}
```

### 監控統計

利用 `@Around` 通知來記錄方法的執行時間。

```java
@Aspect
@Component
public class PerformanceAspect {
    @Around("execution(* sparrow..*(..))")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return joinPoint.proceed(); // 執行初始方法
        } finally {
            long executionTime = System.currentTimeMillis() - start;
            System.out.println(joinPoint.getSignature() + " 執行耗時: " + executionTime + "ms");
        }
    }
}
```

## 參考資料

[Spring AOP 官方文件](https://docs.spring.io/spring-framework/reference/core/aop.html)
