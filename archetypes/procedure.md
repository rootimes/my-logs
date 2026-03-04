+++
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
date = {{ .Date }}
draft = true
categories = [
    "category",
]
tags = [
    "tag",
]
description = "這是一則描述"
+++

## 前置

提供前置的狀態，避免因初始條件不同導致的潛在錯誤

```yaml
## 系統
ubuntu 24.04
```

## 主要內容

開始撰寫主要內容
依照不同種類內容，會有不一樣的格式
以下舉幾個例子

### 操作

各種功能的操作方法，主要記錄操作流程，還有關鍵的提示與指令

#### Step 1

#### Step 2

## 參考資料

[示範連結](https://example.com)
