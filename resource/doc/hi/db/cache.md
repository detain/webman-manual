# कैश

[webman/cache](https://github.com/webman-php/cache) [symfony/cache](https://github.com/symfony/cache) पर आधारित एक कैश कंपोनेंट है, जो कोरूटीन और गैर-कोरूटीन दोनों वातावरण के साथ संगत है और कनेक्शन पूल को सपोर्ट करता है।

## स्थापना

```php
composer require -W webman/cache
```

## उदाहरण
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

## कॉन्फ़िगरेशन फ़ाइल स्थान
कॉन्फ़िगरेशन फ़ाइल `config/cache.php` में है। मौजूद न हो तो मैन्युअल रूप से बनाएं।

## कॉन्फ़िगरेशन फ़ाइल सामग्री
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
`stores.driver` चार ड्राइवर सपोर्ट करता है: **file**, **redis**, **array** और **apcu**।

#### file ड्राइवर
डिफ़ॉल्ट ड्राइवर। बाहरी निर्भरता नहीं। प्रक्रियाओं के बीच कैश शेयरिंग सपोर्ट करता है। एक से अधिक सर्वर के बीच शेयरिंग सपोर्ट नहीं करता।

#### array ड्राइवर
मेमोरी स्टोरेज, सबसे अच्छा प्रदर्शन लेकिन मेमोरी इस्तेमाल करता है। प्रक्रियाओं या सर्वर के बीच शेयरिंग सपोर्ट नहीं करता। प्रक्रिया रीस्टार्ट के बाद डेटा खो जाता है। छोटे कैश वॉल्यूम वाले प्रोजेक्ट के लिए उपयुक्त।

#### apcu ड्राइवर
मेमोरी स्टोरेज। प्रदर्शन array के बाद दूसरा सबसे अच्छा। प्रक्रियाओं के बीच कैश शेयरिंग सपोर्ट करता है। एक से अधिक सर्वर के बीच शेयरिंग सपोर्ट नहीं करता। प्रक्रिया रीस्टार्ट के बाद डेटा खो जाता है। छोटे कैश वॉल्यूम वाले प्रोजेक्ट के लिए उपयुक्त।

> [APCu एक्सटेंशन](https://pecl.php.net/package/APCu) इंस्टॉल और इनेबल करना ज़रूरी है। बार-बार कैश लिखने/मिटाने वाले सिनारियो के लिए अनुशंसित नहीं, इससे प्रदर्शन साफ़ तौर पर कम हो सकता है।

#### redis ड्राइवर
[webman/redis](./redis.md) कंपोनेंट पर निर्भर। प्रक्रियाओं और सर्वर के बीच कैश शेयरिंग सपोर्ट करता है।

**stores.redis.connection**

`stores.redis.connection` वही है जो `config/redis.php` में परिभाषित key से मेल खाता है। Redis इस्तेमाल करने पर `webman/redis` कॉन्फ़िगरेशन फिर इस्तेमाल होता है, कनेक्शन पूल सेटिंग भी शामिल।

**`config/redis.php` में कैश के लिए अलग Redis कॉन्फ़िगरेशन जोड़ने की सलाह, उदाहरण:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== नया जोड़ें
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

फिर `stores.redis.connection` को `cache` पर सेट करें। अंतिम `config/cache.php` लगभग ऐसा दिखेगा:

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

## स्टोर बदलना
अलग ड्राइवर इस्तेमाल करने के लिए स्टोर मैन्युअल तौर पर बदला जा सकता है, उदाहरण:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **सुझाव**
> कैश key नाम [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) से सीमित हैं और `{}()/\@:` में कोई भी अक्षर नहीं रख सकते। `symfony/cache` 7.2.4 से PHP ini विकल्प `zend.assertions=-1` से यह चेक अस्थायी रूप से छोड़ा जा सकता है।

## अन्य कैश कंपोनेंट का उपयोग

[ThinkCache](https://github.com/webman-php/think-cache) कंपोनेंट के लिए [दूसरे डेटाबेस](others.md#ThinkCache) देखें।
