# Redis
Redis এর ব্যবহার ডাটাবেসের অনুরূপ, উদাহরণস্বরূপ `plugin/foo/config/redis.php`
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
ব্যবহারের সময়
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

একইভাবে, যদি মূল প্রকল্পের Redis কনফিগারেশন পুনরায় ব্যবহার করতে চান
```php
use support\Redis;
Redis::get('key');
// ধরে নিন মূল প্রকল্পে cache সংযোগও কনফিগার করা হয়েছে
Redis::connection('cache')->get('key');
```
