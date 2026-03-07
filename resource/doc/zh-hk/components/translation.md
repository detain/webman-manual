# 多語言

多語言功能使用 [symfony/translation](https://github.com/symfony/translation) 元件。

## 安裝
```
composer require symfony/translation
```

## 建立語言包
webman 預設將語言包放在 `resource/translations` 目錄下（如不存在請自行建立），如需變更目錄，請在 `config/translation.php` 中設定。
每種語言對應其中一個子資料夾，語言定義預設放在 `messages.php` 裡。範例如下：
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

所有語言檔都會回傳一個陣列，例如：
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## 設定

`config/translation.php`

```php
return [
    // 預設語言
    'locale' => 'zh_CN',
    // 回退語言，無法在目前語言中找到翻譯時，會嘗試使用回退語言中的翻譯
    'fallback_locale' => ['zh_CN', 'en'],
    // 語言檔案存放的資料夾
    'path' => base_path() . '/resource/translations',
];
```

## 翻譯

使用 `trans()` 方法進行翻譯。

建立語言檔 `resource/translations/zh_CN/messages.php` 如下：
```php
return [
    'hello' => '你好 世界!',
];
```

建立檔案 `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

存取 `http://127.0.0.1:8787/user/get` 將回傳「你好 世界!」

## 變更預設語言

使用 `locale()` 方法切換語言。

新增語言檔 `resource/translations/en/messages.php` 如下：
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 切換語言
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
存取 `http://127.0.0.1:8787/user/get` 將回傳「hello world!」

你也可以使用 `trans()` 函數的第四個參數來暫時切換語言，例如上面的例子與下面這段是等價的：
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 第四個參數切換語言
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## 為每個請求明確設定語言
translation 為單例，代表所有請求共用此實例，若某個請求使用 `locale()` 設定了預設語言，會影響此程序後續所有請求。因此應為每個請求明確設定語言。例如使用以下中介軟體：

建立檔案 `app/middleware/Lang.php`（如目錄不存在請自行建立）如下：
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

在 `config/middleware.php` 中加入全域中介軟體如下：
```php
return [
    // 全域中介軟體
    '' => [
        // ... 此處省略其他中介軟體
        app\middleware\Lang::class,
    ]
];
```


## 使用佔位符
有時，一則訊息包含需要被翻譯的變數，例如
```php
trans('hello ' . $name);
```
遇到這種情況時，我們使用佔位符來處理。

變更 `resource/translations/zh_CN/messages.php` 如下：
```php
return [
    'hello' => '你好 %name%!',
];
```
翻譯時，透過第二個參數傳入佔位符對應的值：
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## 處理複數
部分語言會因數量不同而呈現不同句式，例如 `There is %count% apple`，當 `%count%` 為 1 時句式正確，大於 1 時則錯誤。

遇到這種情況時，我們使用**管道符**（`|`）列出複數形式。

於語言檔 `resource/translations/en/messages.php` 新增 `apple_count` 如下：
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

我們還可以指定數字範圍，建立更複雜的複數規則：
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## 指定語言檔

語言檔預設名稱為 `messages.php`，實際上你可以建立其他名稱的語言檔。

建立語言檔 `resource/translations/zh_CN/admin.php` 如下：
```php
return [
    'hello_admin' => '你好 管理員!',
];
```

透過 `trans()` 的第三個參數指定語言檔（省略 `.php` 副檔名）。
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理員!
```

## 更多資訊
請參考 [symfony/translation 手冊](https://symfony.com/doc/current/translation.html)
