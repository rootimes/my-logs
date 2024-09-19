+++
title = 'Laravel Style'
date = 2024-09-19T23:04:43+08:00
draft = true
+++

## Laravel coding style 的一些實踐

這篇主要整理各種在 Laravel 框架上的良好風格實踐

### 單一職責原則
一個類別與方法應只有一個職責

#### 舉例
```php
public function getFullNameAttribute(): string
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

#### 調整
1. 使用語意化命名 function 使程式目的更清晰
2. 縮減每一個 function 的用途，簡化職責

```php
public function getFullNameAttribute(): string
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient(): bool
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong(): string
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort(): string
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

## 參考資料
[laravel best practices](https://github.com/alexeymezenin/laravel-best-practices)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論