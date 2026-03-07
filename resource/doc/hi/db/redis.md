# रेडिस

[webman/redis](https://github.com/webman-php/redis) [illuminate/redis](https://github.com/illuminate/redis) को कनेक्शन पूल जोड़कर विस्तारित करता है और कोरूटीन तथा गैर-कोरूटीन दोनों वातावरणों में काम करता है। उपयोग लारावेल के समान है।

`illuminate/redis` उपयोग करने से पहले `php-cli` के लिए रेडिस एक्सटेंशन इंस्टॉल करना आवश्यक है।

## इंस्टॉलेशन

```php
composer require -W webman/redis illuminate/events
```

इंस्टॉलेशन के बाद रिस्टार्ट आवश्यक है (रिलोड काम नहीं करता)।

## कॉन्फ़िगरेशन

रेडिस कॉन्फ़िगरेशन फ़ाइल `config/redis.php` में है:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // कनेक्शन पूल सेटिंग
            'max_connections' => 10,     // पूल में अधिकतम कनेक्शन
            'min_connections' => 1,      // पूल में न्यूनतम कनेक्शन
            'wait_timeout' => 3,         // कनेक्शन हासिल करने की अधिकतम प्रतीक्षा (सेकंड)
            'idle_timeout' => 50,        // इसके बाद कनेक्शन min_connections तक रिलीज़ होते हैं
            'heartbeat_interval' => 50,  // हार्टबीट अंतराल (60 सेकंड से अधिक न हो)
        ],
    ]
];
```

## कनेक्शन पूल के बारे में

* प्रत्येक प्रोसेस का अपना पूल होता है; पूल प्रोसेस के बीच साझा नहीं होते।
* बिना कोरूटीन के निष्पादन अनुक्रमिक होता है, इसलिए अधिकतम एक कनेक्शन उपयोग होता है।
* कोरूटीन के साथ निष्पादन समवर्ती होता है और पूल `min_connections` से `max_connections` के बीच गतिशील रूप से समायोजित होता है।
* जब रेडिस इस्तेमाल करने वाले कोरूटीन की संख्या `max_connections` से अधिक हो, तो वे अधिकतम `wait_timeout` सेकंड प्रतीक्षा करते हैं; उसके बाद अपवाद फेंका जाता है।
* निष्क्रिय अवस्था में (कोरूटीन हो या न हो) कनेक्शन `idle_timeout` के बाद रिलीज़ होते हैं जब तक `min_connections` न रह जाए (0 मान्य है)।

## उदाहरण

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## रेडिस एपीआई

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

इसके समतुल्य:

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **ध्यान दें**
> `Redis::select($db)` एपीआई सावधानी से उपयोग करें। चूंकि वेबमैन मेमोरी में स्थायी फ्रेमवर्क है, एक अनुरोध में डेटाबेस बदलने से बाद के अनुरोध प्रभावित होते हैं। कई डेटाबेस के लिए प्रत्येक `$db` के लिए अलग रेडिस कनेक्शन कॉन्फ़िगर करने की सलाह दी जाती है।

## कई रेडिस कनेक्शन उपयोग करना

`config/redis.php` कॉन्फ़िगरेशन फ़ाइल में उदाहरण:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

डिफ़ॉल्ट रूप से `default` के अंतर्गत कॉन्फ़िगर कनेक्शन उपयोग होता है। कौन सा रेडिस कनेक्शन उपयोग करना है यह चुनने के लिए `Redis::connection()` मेथड का उपयोग करें:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## क्लस्टर कॉन्फ़िगरेशन

अगर आपके ऐप में रेडिस सर्वर क्लस्टर का उपयोग है, तो कॉन्फ़िगरेशन में `clusters` कुंजी के अंतर्गत परिभाषित करें:

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

डिफ़ॉल्ट रूप से क्लस्टर नोड पर क्लाइंट-साइड शार्डिंग कर सकता है, जिससे नोड पूल और बड़ी मेमोरी संभव होती है। ध्यान दें कि क्लाइंट शार्डिंग विफलता नहीं संभालता; इसलिए यह मुख्यतः दूसरी मुख्य डेटाबेस से कैश डेटा के लिए उपयुक्त है। रेडिस नेटिव क्लस्टर के लिए कॉन्फ़िगरेशन में `options` के अंतर्गत निर्दिष्ट करें:

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## पाइपलाइन कमांड

जब एक ऑपरेशन में सर्वर को कई कमांड भेजने हों तो पाइपलाइन का उपयोग करें। `pipeline` मेथड एक closure स्वीकार करता है; सभी कमांड एक ही ऑपरेशन में निष्पादित होते हैं:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
