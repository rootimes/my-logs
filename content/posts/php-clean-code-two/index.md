+++
title = 'PHP Clean Cod Two'
date = 2024-09-21T02:19:11+08:00
draft = true
categories = [
    "coding-style",
]
tags = [
    "php",
]
description = "PHP 的 clean code 範例"
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

### For 迴圈使用

避免使用意義不明的 Array

#### 舉例

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    // Wait, what is `$li` for again?
    dispatch($li);
}
```

#### 調整

使用 foreach 來遍歷 Array

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```
```php
$locations = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($locations); $i++) {
    $location = $locations[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```

### Comparison

型別轉換會導致比較失敗，字串轉為整數後比較，但事實字串不是數字

#### 舉例 (一)

```php
$x = '0';
$y = 0;

if ($x != $y) {
    // 不會執行
    // The expression will always pass
}
```

#### 調整

強制比較型別和值

```php
$x = '0';
$y = 0;

if ($x !== $y) {
    // 會執行
    // This ensures the comparison is accurate by type and value
}

```

#### 舉例 (二)

```php
$arr = null;
$str = '';

if ($arr != $str) {
    // 不會執行
    // The expression will always pass
}
```

#### 調整

強制比較型別和值

```php
$arr = null;
$str = '';

if ($arr !== $str) {
    // 會執行
    // This ensures the comparison is accurate by type and value
}
```

### 使用判斷語法糖來簡化語法

#### 舉例

```php
if (isset($_GET['name'])) {
    $name = $_GET['name'];
} else {
    $name = 'nobody';
}
```

#### 調整

```php
$name = $_GET['name'] ?? 'nobody';
```

### 函式用法

需使用 type hinting 限定參數值，避免內部還需要判斷

#### 舉例

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

#### 調整

```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

減少函式參數，最好少於 2 個

#### 舉例

```php
class Questionnaire
{
    public function __construct(
        string $firstName,
        string $lastName,
        string $patronymic,
        string $region,
        string $district,
        string $city,
        string $phone,
        string $email
    ) {
        // ...
    }
}
```

#### 調整

```php
class Name
{
    private $firstName;

    private $lastName;

    private $patronymic;

    public function __construct(string $firstName, string $lastName, string $patronymic)
    {
        $this->firstName = $firstName;
        $this->lastName = $lastName;
        $this->patronymic = $patronymic;
    }
}

class City
{
    private $region;

    private $district;

    private $city;

    public function __construct(string $region, string $district, string $city)
    {
        $this->region = $region;
        $this->district = $district;
        $this->city = $city;
    }
}

class Contact
{
    private $phone;

    private $email;

    public function __construct(string $phone, string $email)
    {
        $this->phone = $phone;
        $this->email = $email;
    }
}

class Questionnaire
{
    public function __construct(Name $name, City $city, Contact $contact)
    {
        // ...
    }
}
```

function 名稱要表達目的性

#### 舉例

```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// What is this? A handle for the message? Are we writing to a file now?
$message->handle();
```

#### 調整

```php
class Email
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);

$message->send();
```

function 內部盡量減少抽象層(abstraction)

程式碼的問題在於抽象層次混亂和函式責任不明確

#### 舉例

```php

tokenize 的細節過多，目的不夠明確
function tokenize(string $code): ~~array~~
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

似乎是為了進行 lexical analysis，但其實它在做的是把 token 轉換為 AST 節點
function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterPHPAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
    }
}
```

#### 調整

```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterPHPAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
        }
    }
}
```

函式內不要使用 boolean 作為區分，函式功能違反單一職責原則
#### 舉例

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/' . $name);
    } else {
        touch($name);
    }
}
```

#### 調整

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/' . $name);
}
```

函式內不僅僅有回傳，還有改變外部的值，這將更難維護
#### 舉例

```php
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Rootimes\' blog';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}
splitIntoFirstAndLastName();

var_dump($name);
// ['Rootimes', 'blog'];
```

#### 調整

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Rootimes blog';
$newName = splitIntoFirstAndLastName($name);

var_dump($name);
// 'Rootimes blog';

var_dump($newName);
// ['Rootimes', 'blog'];
```

避免使用全域函式， php 本身沒有 namespace 會導致衝突
#### 舉例

```php
function config(): array
{
    return [
        'foo' => 'bar',
    ];
}
```

#### 調整
利用類別封裝

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        // null coalescing operator
        return $this->configuration[$key] ?? null;
    }
}

# 使用 class 

$configuration = new Configuration([
    'foo' => 'bar',
]);
```

避免使用 singleton pattern

1. 隱藏了應用的依賴關係
2. 不僅負責自身的邏輯，還負責自己的創建和生命週期管理
#### 舉例

```php
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): self
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```

#### 調整

明確創建 DBConnection 的實例

```php
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

    // ...
}
```
```php
$connection = new DBConnection($dsn);
```
### 物件與資料結構

#### 使用物件封裝

#### 舉例

```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

#### 調整

```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

$bankAccount->withdraw($shoesPrice);

$balance = $bankAccount->getBalance();
```

## 參考資料
[piotrplenik/clean-code-php](https://github.com/piotrplenik/clean-code-php)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論