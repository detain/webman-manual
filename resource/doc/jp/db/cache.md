# キャッシュ

[webman/cache](https://github.com/webman-php/cache)は[symfony/cache](https://github.com/symfony/cache)をベースとしたキャッシュコンポーネントで、コルーチン環境と非コルーチン環境の両方に対応し、接続プールをサポートしています。

## インストール

```php
composer require -W webman/cache
```

## 例
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

## 設定ファイルの場所
設定ファイルは `config/cache.php` にあります。存在しない場合は手動作成してください。

## 設定ファイルの内容
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
`stores.driver` は **file**、**redis**、**array**、**apcu** の4種類のドライバをサポートしています。

#### file ドライバ
デフォルトのドライバです。外部コンポーネントに依存しません。プロセス間でのキャッシュ共有をサポートします。複数サーバー間での共有はサポートしません。

#### array ドライバ
メモリストレージで最高のパフォーマンスを発揮しますが、メモリを消費します。プロセス間・サーバー間での共有をサポートしません。プロセス再起動後にデータは失われます。キャッシュデータ量が少ないプロジェクトに適しています。

#### apcu ドライバ
メモリストレージです。パフォーマンスは array に次ぎます。プロセス間でのキャッシュ共有をサポートします。複数サーバー間での共有はサポートしません。プロセス再起動後にデータは失われます。キャッシュデータ量が少ないプロジェクトに適しています。

> [APCu 拡張機能](https://pecl.php.net/package/APCu)のインストールと有効化が必要です。キャッシュの書き込み・削除が頻繁なシナリオには不向きで、顕著なパフォーマンス低下の原因になることがあります。

#### redis ドライバ
[webman/redis](./redis.md) コンポーネントに依存します。プロセス間・サーバー間でのキャッシュ共有をサポートします。

**stores.redis.connection**

`stores.redis.connection` は `config/redis.php` で定義されたキーに対応します。Redis を使用する際は、接続プール設定を含む `webman/redis` の設定を再利用します。

**`config/redis.php` にキャッシュ用の専用設定を追加することを推奨します。例：**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== 新規追加
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

続いて `stores.redis.connection` を `cache` に設定します。最終的な `config/cache.php` は以下のようになります：

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

## ストアの切り替え
次のように手動でストアを切り替え、異なるドライバを使用できます：

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **ヒント**
> キャッシュキー名は [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) により制限され、`{}()/\@:` のいずれの文字も含めることはできません。`symfony/cache` 7.2.4 以降、PHP ini の `zend.assertions=-1` によりこのチェックを一時的にスキップできます。

## 他のキャッシュコンポーネントの使用

[ThinkCache](https://github.com/webman-php/think-cache) コンポーネントについては、[その他のデータベース](others.md#ThinkCache) を参照してください。
