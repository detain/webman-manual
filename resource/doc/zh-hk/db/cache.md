# 緩存

[webman/cache](https://github.com/webman-php/cache) 是以 [symfony/cache](https://github.com/symfony/cache) 為基礎開發的緩存組件，相容協程與非協程環境，支援連接池。

## 安裝

```php
composer require -W webman/cache
```

## 示例
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## 配置檔位置
配置檔位於 `config/cache.php`，若不存在請手動建立。

## 配置檔內容
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` 支援四種驅動：**file**、**redis**、**array**、**apcu**。

#### file 檔案驅動
為預設驅動，不依賴其他組件，支援跨進程共用緩存數據，不支援多伺服器共用緩存數據。

#### array 記憶體驅動
記憶體儲存，效能最佳，但會佔用記憶體，不支援跨進程跨伺服器共用數據，進程重啟後失效，一般用於緩存數據量小的項目。

#### apcu 記憶體驅動
記憶體儲存，效能僅次於 array，支援跨進程共用緩存數據，不支援多伺服器共用緩存數據，進程重啟後失效，一般用於緩存數據量小的項目。

> 需安裝並啟用 [APCu 擴展](https://pecl.php.net/package/APCu)；不建議用於頻繁進行緩存寫入/刪除的場景，會導致明顯的效能下降。

#### redis 驅動
依賴 [webman/redis](./redis.md) 組件，支援跨進程跨伺服器共用緩存數據。

**stores.redis.connection**

`stores.redis.connection` 對應 `config/redis.php` 中的 key。使用 redis 時會複用 `webman/redis` 的配置，包括連接池配置。

**建議在 `config/redis.php` 中新增獨立的配置，例如 cache 類似如下**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== 新增
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

然後將 `stores.redis.connection` 設為 `cache`，`config/cache.php` 最終配置類似如下：

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## 切換儲存
可透過以下程式碼手動切換 store，以使用不同的儲存驅動，例如：

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **提示**
> Key 名稱受 [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) 限制，不得包含 `{}()/\@:` 中任一字元，此判斷截至 `symfony/cache` 7.2.4 可暫時透過 PHP ini 設定 `zend.assertions=-1` 略過。

## 使用其他 Cache 組件

[ThinkCache](https://github.com/webman-php/think-cache) 組件使用參考 [其他數據庫](others.md#ThinkCache)
