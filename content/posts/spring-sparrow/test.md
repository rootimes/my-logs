+++
title = 'Spring Sparrow Integration Test'
date = 2026-03-12T14:02:09+08:00
draft = false
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：整合測試篇章"
+++

## 關注點

1. CRUD 的整合測試

## 簡單介紹測試

軟體測試大致可以分為幾種類型：

- **單元測試（Unit Test）**  
  測試單一方法或類別的功能，通常會使用模擬物件（Mock）來隔離外部依賴，以確保測試專注於單一邏輯單元。

- **整合測試（Integration Test）**  
  測試多個組件或系統之間的交互，確保它們能夠正確協作，例如 Controller、Service、Repository 以及框架設定之間的整體運作。

- **端對端測試（End-to-End Test, E2E）**  
  從使用者角度出發，測試整個應用程式的完整流程，驗證系統在真實使用情境下的行為是否符合預期。

### Spring Sparrow 專案的測試策略

測試方式會依據**專案需求與關注點**而有所不同。

在 **Spring Sparrow** 這個專案中，其定位是 **API Resource Server**，主要關注點在於：

- API 是否能正確回傳資料  
- API 的整體功能是否正常運作  

由於此專案的 **內部業務邏輯並不複雜**，因此選擇使用 **整合測試（Integration Test）** 來驗證 API 的功能。

### 個人實務經驗中的測試使用情境

#### 單元測試（Unit Test）

通常會用在：

- **邏輯複雜度高**
- **需要隔離外部依賴**
- **失敗成本高**

例如 **身分驗證服務（Auth Server）**。

此類服務通常屬於**核心基礎設施**，包含：

- 加密處理
- 權限判定
- Token 驗證等核心邏輯

這些邏輯的**變動頻率通常較低**，但一旦出錯，可能導致整個系統無法正常運作。

因此會透過單元測試，確保每個細節都具有高度穩定性。

#### 整合測試（Integration Test）

通常會用在關注以下情境：

- 資料庫互動
- 框架設定
- Controller / Service / Repository 之間的協作

例如 **資源存取 API（CRUD Resource）**。

這類服務的業務邏輯通常較為直觀，透過整合測試即可在**較低開發成本**下，驗證整體 **Feature 是否能正常運作**。

#### 端對端測試（End-to-End Test）

目前個人在實務專案中尚未實際使用過。

以網站應用為例，E2E 測試通常需要撰寫大量**使用者行為腳本**，透過瀏覽器模擬操作，例如：

- 點擊按鈕
- 填寫表單
- 驗證畫面結果

某種程度上類似於**自動化爬蟲操作 UI**。

然而這類測試的維護成本相當高，尤其是在前端畫面變動頻繁的專案中，測試腳本往往剛完成不久就需要重新調整。

說這麼多，其實只是是想表達：

> **Spring Sparrow 目前只撰寫整合測試，並不是偷懶，而是根據專案需求與關注點所做出的技術選擇。**

### 加入相關依賴

有部分的測試依賴在 REST Docs 那個篇章已經加入，這裡就不重複了。

- **json-path**：用於解析和驗證 JSON 回應。
- **hamcrest**：提供更靈活的斷言語法。

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>3.0</version>
    <scope>test</scope>
</dependency>
```

### 撰寫測試

以下舉一些測試方法的範例，實際上專案中還有更多的測試，涵蓋了各種不同的 API 行為與邏輯驗證。

由於是整合測試，所以先設定一下測試環境，在 **test/java/sparrow/resources** ，

建立一個 application-test.properties，裡面放一些測試專用的設定。

### 測試類別設定

在測試類別上使用 @Transactional，確保每個測試方法執行完畢後會自動回滾（Rollback），

維持資料庫的潔淨度，避免測試間的資料干擾。

```java
@Transactional
public class PostTest extends RestDocsTest
```

#### 建立文章 (Create Post) 測試

驗證點：請求發送後，伺服器是否回傳 **201 Created**，且 **status** 能正確套用預設值 **DRAFT。**

- @Test 測試方法宣告
- createPost_shouldReturnCreated 是行為命名法，表達意思是「當呼叫 createPost API 時，應該回傳 Created 狀態碼」。

先建立一個使用者的 Entity 產生一個 unique slug 用來建立一個 Request 服務。

```java
    private PostRequest buildRequest(Integer userId, String title, String slug, Meta meta, PostStatus status) {
        PostRequest request = new PostRequest();
        request.setUserId(userId);
        request.setTitle(title);
        request.setSlug(slug);
        request.setMeta(meta);
        request.setStatus(status);
        return request;
    }
```

現在 PostRequest 相當於

```json
{
  "userId": 1,
  "title": "title-1",
  "slug": "slug-1",
  "meta": {
    "meta-1": "val-1"
  },
  "status": null //測試，預設會轉為 DRAFT
}
```

使用 MockMvc 模擬 HTTP Request，不啟動真正 HTTP Server 的整合測試方式。

- .contentType 用來指定請求的內容類型，這裡是 application/json。
- .content 用來設定請求的內容，這裡將 PostRequest 物件轉換為 JSON 字串。
- .andExpect 用來驗證 API 回應和內容是否符合預期。
- .andDo 用來生成 REST Docs 的文件片段，這裡指定了文件的名稱為 "post-create"。

```java
    @Test
    public void createPost_shouldReturnCreated() throws Exception {
        User user = createUser();
        String slug = uniqueSlug("slug-1");
        PostRequest request = buildRequest(user.getId(), "title-1", slug, buildMeta("meta-1", "val-1"), null);

        this.mockMvc.perform(post("/api/posts")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(notNullValue()))
                .andExpect(jsonPath("$.userId").value(user.getId()))
                .andExpect(jsonPath("$.title").value("title-1"))
                .andExpect(jsonPath("$.slug").value(slug))
                .andExpect(jsonPath("$.meta['meta-1']").value("val-1"))
                .andExpect(jsonPath("$.status").value("DRAFT"))
                .andDo(document("post-create"));
    }
```

#### 分頁查詢 (Get List) 測試

驗證點：API 是否能正確回傳分頁結果，且包含剛剛建立的文章。

準備測試資料

```java
    private Post createPostEntity(User user, String title, String slug, Meta meta, PostStatus status) {
        Post post = new Post();
        post.setUser(user);
        post.setTitle(title);
        post.setSlug(slug);
        post.setMeta(meta);
        post.setStatus(status);
        return postRepository.save(post);
    }
```

- .param 用來設定查詢參數，這裡設定了分頁和排序的參數。

```java
    @Test
    public void getAllPosts_shouldReturnPagedResult() throws Exception {
        User user = createUser();
        String firstSlug = uniqueSlug("slug-1");
        String secondSlug = uniqueSlug("slug-2");

        createPostEntity(user, "title-1", firstSlug, buildMeta("meta-1", "val-1"), PostStatus.DRAFT);
        createPostEntity(user, "title-2", secondSlug, buildMeta("meta-2", "val-2"), PostStatus.PUBLISHED);

        this.mockMvc.perform(get("/api/posts")
                .param("page", "0")
                .param("size", "20")
                .param("sort", "id,desc"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content.length()").value(greaterThanOrEqualTo(2)))
                .andExpect(jsonPath("$.content[?(@.slug == '" + firstSlug + "')].title").value(hasItem("title-1")))
                .andExpect(jsonPath("$.content[?(@.slug == '" + secondSlug + "')].title").value(hasItem("title-2")))
                .andDo(document("post-list"));
    }
```

#### 更新文章 (Update Post) 測試

驗證點：更新文章的 API，驗證更新後的資料是否正確替換掉原本的資料。

```java
    public void updatePost_withTags_shouldReplaceTags() throws Exception {
        User user = createUser();
        Post created = createPostEntity(user, "title-1", uniqueSlug("slug-1"),
                buildMeta("meta-1", "val-1"), PostStatus.DRAFT);

        String updatedSlug = uniqueSlug("slug-2");
        PostRequest updateRequest = buildRequest(user.getId(), "title-2", updatedSlug,
                buildMeta("meta-2", "val-2"), PostStatus.PUBLISHED);
        updateRequest.setTags(Set.of("tag-2", "tag-3"));

        this.mockMvc.perform(put("/api/posts/{id}", created.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updateRequest)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.meta['meta-2']").value("val-2"))
                .andExpect(jsonPath("$.tags").isArray())
                .andExpect(jsonPath("$.tags", hasItem("tag-2")))
                .andExpect(jsonPath("$.tags", hasItem("tag-3")))
                .andDo(document("post-update-with-tags"));
    }
```

#### 刪除文章 (Delete Post) 測試

驗證點：刪除文章後，確認該文章已經不存在。

```java
    @Test
    public void deletePost_shouldRemovePost() throws Exception {
        User user = createUser();
        Post created = createPostEntity(user, "title-1", uniqueSlug("slug-1"), null, PostStatus.DRAFT);

        this.mockMvc.perform(delete("/api/posts/{id}", created.getId()))
                .andExpect(status().isNoContent())
                .andDo(document("post-delete"));

        // 驗證副作用：確認資源已不存在
        this.mockMvc.perform(get("/api/posts/{id}", created.getId()))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.error").value("Post not found"));
    }
```

### REST Docs 文件撰寫

Test-Driven Documentation：REST Docs 的一個重要特點是**測試驅動文件撰寫**，也就是說，文件的內容是從測試方法中自動生成的。

核心理念是「唯有通過測試，才會有文件」。

所以也要補一下相關的 **adoc** 文件片段。

#### 結構化文件定義

在專案中撰寫 **index.adoc** 與 **post.adoc**，透過 include 指令將測試生成的片段（Snippets）動態引入。

```adoc
= Spring Sparrow Legacy API Guide
:toc: left
:toclevels: 3
:sectnums:

== Introduction

Spring Sparrow Legacy 的 API Guide 文件。

=== Base URL

所有 API 的 base URL 為：

-
http://localhost:8080
-

=== Request / Response 格式

* 所有請求與回應均使用 `application/json`
* 時間格式為 ISO 8601（`yyyy-MM-dd'T'HH:mm:ss`）

=== HTTP 狀態碼

[cols="1,3"]
|===
| 狀態碼 | 說明

| `200 OK`
| 請求成功

| `201 Created`
| 資源建立成功

| `204 No Content`
| 刪除成功，無回應內容

| `400 Bad Request`
| 請求格式錯誤或參數驗證失敗

| `404 Not Found`
| 指定資源不存在

| `409 Conflict`
| 資源衝突，例如 slug 重複
|===

include::status.adoc[]

include::post.adoc[]

```

post.adoc

```adoc
== Post API
This API manages posts including create, list, detail, update, and delete operations.

=== Create Post
==== 請求範例 (HTTP Request)
include::{snippets}/post-create/http-request.adoc[]

==== 請求範例 (cURL)
include::{snippets}/post-create/curl-request.adoc[]

==== 回應範例 (HTTP Response)
include::{snippets}/post-create/http-response.adoc[]

// 其他操作的文件片段同樣以此方式引入
```

![AsciiDoc](../../../images/spring-sparrow/asciidoc.png)

## 參考資料

- [MockMvc 官方文件](https://docs.spring.io/spring-framework/reference/6.1/testing/spring-mvc-test-framework.html)
- [Spring REST Docs 官方文件](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/)
- [Asciidoctor 官方文件](https://docs.asciidoctor.org/asciidoc/latest/)
