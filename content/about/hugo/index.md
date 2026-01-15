+++
title = 'Hugo'
date = 2025-05-13T00:36:16+08:00
draft = true
+++

## cheat sheet

### 建立文章

1. 預設模板
```bash
hugo new content about/{folder}/{filename}.md
```

2. 不同的模板
```bash
hugo new content --kind {archetypes} {folder}/{filename}.md
```

### 啟動服務器

1. 啟動服務器
```bash
hugo server
```

2. 啟動服務器，並指定 port
```bash
hugo server -p {port}
```

3. 啟動服務器，並查閱草稿
```bash
hugo server -D
```

## 參考資料
- [Hugo](https://gohugo.io/)