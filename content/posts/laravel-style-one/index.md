+++
title = 'Laravel Style'
date = 2024-09-19T23:04:43+08:00
draft = false
tags = [
    "laravel",
]
categories = [
    "coding-style",
]
description = "Laravel 的 coding style 整理"
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

### 降低 Controller 複雜度

筆者認為這些技巧可以按情形調整，避免過度設計問題

一些本身很簡單的的功能(行數本身少)，仍然僵硬的套用這些原則

最終反而維護困難(程式碼散落在專案各處)

以下拆分目的主要是示範

#### 舉例 : Service

將商業邏輯移到 Service 層當中，Controller 只保留選擇使用 Service 和取得參數方法

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }

    ...
}
```

#### 調整

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ...
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

#### 舉例 : Query

使用 Query Builder 或是 Raw SQL 時，將這部分程式放置在 Model 當中，也可自訂一個 Repository 層

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

#### 調整

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

#### 舉例 : Validate

需要驗證的資料，驗證方法可以移到 RequestForm 類別內

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ...
}
```

#### 調整
```php
public function store(PostRequest $request)
{
    ...
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

#### 舉例 : Resource
```php
public function index(Request $request)
{
    ...

    $message = $this->postService->getPosts();

    return response()->json($message, 200);
}
```

#### 調整 
```php
public function index(Request $request)
{
    ...

    $message = $this->postService->getPosts(); # model collection

    return new PostResource::collection($message);
}

public function show(Request $request)
{
    ...

    $message = $this->postService->getPosts(); # model instances

    return new PostResource($message);
}
```

## 參考資料
[laravel best practices](https://github.com/alexeymezenin/laravel-best-practices)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論