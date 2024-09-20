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

#### 舉例 : 拆分複雜判斷

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

#### 舉例 : 程式碼中註釋
```php
// 確定是否有任何 Join
if (count((array) $builder->getQuery()->joins) > 0)
```

#### 調整
```php
if ($this->hasJoins())
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

### DRY 原則 - 不要重覆自己

通過 SRP (單一職責原則)，先行簡化程式碼，再將部分封裝

重複使用程式碼，這並不是目的，它是一個整理結果

筆者自己認為，隨著 AI 發展

重複使用程式碼的部分可以在特定的類別或是命名空間內即可

未必要追求大規模的重用，高度的重用程式碼

有時帶來很高的耦合性，在開發上反而更加困難

#### 舉例 : Eloquent Scope
```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

#### 調整
```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

### 大量賦值，使用 ORM 方法

Laravel 為避免批量賦值導致非預期變更，提供了 $fillable 和 $guarded 的限制

如果使用手動賦值，不受此限

#### 舉例
```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;

// Add category to article
$article->category_id = $category->id;
$article->save();
```

#### 調整
```php
$category->article()->create($request->validated());
```

### 使用 Eager Loading 避免 N+1 問題

使用 with() 同時取得關聯模型

#### 舉例

假設 User 有 100，則會執行 101 次 DB 查詢

每個次取得 profile 都會執行一次

```php
$users = User::all();

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

#### 調整

僅執行 2 次查詢
```php
$users = User::with('profile')->get();

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

### 使用 config 和 enum 代替重複性文字

#### 舉例
```php
public function isNormal()
{
    return $article->type === 'normal';
}

return back()->with('message', 'Your article has been added!');
```

#### 調整
```php
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

```

## 參考資料
[Laravel best practices](https://github.com/alexeymezenin/laravel-best-practices)

## 免責聲明

以上內容算是個人的筆記，可能有誤或較多引用，期望能拋磚引玉

歡迎大家在底下補充與討論