# Redis
استخدام Redis يشبه استخدام قاعدة البيانات، مثلًا `plugin/foo/config/redis.php`
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 1,
    ],
];
```
عند الاستخدام
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

وبالمثل، إذا أردت إعادة استخدام إعداد Redis للمشروع الرئيسي
```php
use support\Redis;
Redis::get('key');
// بافتراض أن المشروع الرئيسي أضاف أيضًا اتصال cache
Redis::connection('cache')->get('key');
```
