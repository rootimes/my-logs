+++
title = 'PHP Clean Code One'
date = 2024-09-21T02:19:11+08:00
draft = true
categories = [
    "coding-style",
]
tags = [
    "php",
]
description = "PHP 的 clean code 範例 (一)"
+++

## PHP Clean Code

Clean Code 是一種精神，本篇主要蒐集在 PHP 這個語法的實踐

### 變數使用

使用有意義且常見的變數名稱

#### 舉例

```php
$ymdstr = $moment->format('y-m-d');
```

#### 調整
```php
$currentDate = $moment->format('y-m-d');
```

相同的實體使用相同的變數

#### 舉例

```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```

#### 調整
```php
getUser();
```

使用易讀和容易搜尋的名稱，取代特定的數值

#### 舉例 (一)

```php
// What the heck is 448 for?
$json = $serializer->serialize($data, 448);
```

#### 調整
```php
$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
```

#### 舉例 (二)

```php
class User
{
    // What the heck is 8 for?
    public $access = 8;
}

// What the heck is 4 for?
if ($user->access & 4) {
    // ...
}

// What's going on here?
$user->access ^= 2;
```

#### 調整
```php
class User
{
    public const ACCESS_READ = 1;   // 0001

    public const ACCESS_CREATE = 2; // 0010

    public const ACCESS_UPDATE = 4; // 0100

    public const ACCESS_DELETE = 8; // 1000

    // User as default can read, create and update something
    public $access = self::ACCESS_READ | self::ACCESS_CREATE | self::ACCESS_UPDATE;
}

if ($user->access & User::ACCESS_UPDATE) {
    // do edit ...
}

// Deny access rights to create something
$user->access ^= User::ACCESS_CREATE;
```

使用有意義單詞作為 Array 的 Key

#### 舉例

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

#### 調整
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```

減少不需要的贅詞命名

#### 舉例

```php
class Car
{
    public $carMake;

    public $carModel;

    public $carColor;

    //...
}
```

#### 調整
```php
class Car
{
    public $make;

    public $model;

    public $color;

    //...
}
```

### if 使用

避免過深的巢狀迴圈

#### 舉例 (一)

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            }
            return false;
        }
        return false;
    }
    return false;
}
```

#### 調整

使用提前回傳和 Array 鍵判斷

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = ['friday', 'saturday', 'sunday'];

    return in_array(strtolower($day), $openingDays, true);
}
```

#### 舉例 (二)

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            }
            return 1;
        }
        return 0;
    }
    return 'Not supported';
}
```

#### 調整

使用提前回傳和整合 if 判斷條件

```php
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n >= 50) {
        throw new Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```

封裝判斷式

#### 舉例

```php
if ($article->state === 'published') {
    // ...
}
```

#### 調整

```php
if ($article->isPublished()) {
    // ...
}
```

避免過多的條件聲明，改變函式目的
維持單一職責原則

#### 舉例

```php
class Airplane {
  // ...
  public function getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return $this->getMaxAltitude() - $this->getPassengerCount();
      case 'Air Force One':
        return $this->getMaxAltitude();
      case 'Cessna':
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
  }
}
```

#### 調整

```php
class Airplane {
  // ...
}

class Boeing777 extends Airplane {
  // ...
  public function getCruisingAltitude() {
    return $this->getMaxAltitude() - $this->getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  public function getCruisingAltitude() {
    return $this->getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  public function getCruisingAltitude() {
    return $this->getMaxAltitude() - $this->getFuelExpenditure();
  }
}
```

避免類型檢查，使用類型聲明

#### 舉例

```php
function combine($val1, $val2) {
  if (is_numeric($val1) && is_numeric(val2)) {
    return val1 + val2;
  }

  throw new \Exception('Must be of type Number');
}
```

#### 調整

```php
function combine(int $val1, int $val2) {
  return $val1 + $val2;
}
```

避免反向的判斷情形

#### 舉例

```php
if (! isDOMNodeNotPresent($node)) {
    // ...
}
```

#### 調整

```php

if (isDOMNodePresent($node)) {
    // ...
}
```

## 參考資料
[piotrplenik/clean-code-php](https://github.com/piotrplenik/clean-code-php)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論