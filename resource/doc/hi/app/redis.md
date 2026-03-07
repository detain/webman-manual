# Redis
Redis का उपयोग डेटाबेस के समान है, उदाहरण के लिए `plugin/foo/config/redis.php`
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
उपयोग करते समय
```php
use support\Redis;
Redis::connection('plugin.foo.default')->get('key');
Redis::connection('plugin.foo.cache')->get('key');
```

इसी तरह, यदि मुख्य परियोजना की Redis कॉन्फ़िगरेशन पुनः उपयोग करनी हो
```php
use support\Redis;
Redis::get('key');
// मान लें कि मुख्य परियोजना ने cache कनेक्शन भी कॉन्फ़िगर किया है
Redis::connection('cache')->get('key');
```
