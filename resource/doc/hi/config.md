# कॉन्फ़िगरेशन फाइलें

## स्थान
webman कॉन्फ़िगरेशन फाइलें `config/` निर्देशिका में स्थित हैं। आप अपने प्रोजेक्ट में संबंधित कॉन्फ़िगरेशन तक पहुँचने के लिए `config()` फ़ंक्शन का उपयोग कर सकते हैं।

## कॉन्फ़िगरेशन तक पहुँचना

सभी कॉन्फ़िगरेशन प्राप्त करें:
```php
config();
```

`config/app.php` में सभी कॉन्फ़िगरेशन प्राप्त करें:
```php
config('app');
```

`config/app.php` में `debug` कॉन्फ़िगरेशन प्राप्त करें:
```php
config('app.debug');
```

यदि कॉन्फ़िगरेशन एक सरणी है, तो आप नेस्टेड मानों तक पहुँचने के लिए `.` का उपयोग कर सकते हैं। उदाहरण:
```php
config('file.key1.key2');
```

## डिफ़ॉल्ट मान
```php
config($key, $default);
```
डिफ़ॉल्ट मान को दूसरे पैरामीटर के रूप में पास करें। यदि कॉन्फ़िगरेशन मौजूद नहीं है तो डिफ़ॉल्ट मान लौटाया जाएगा। यदि कॉन्फ़िगरेशन मौजूद नहीं है और कोई डिफ़ॉल्ट सेट नहीं है तो `null` लौटाया जाएगा।

## कस्टम कॉन्फ़िगरेशन
डेवलपर `config/` निर्देशिका में अपनी स्वयं की कॉन्फ़िगरेशन फाइलें जोड़ सकते हैं। उदाहरण:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**कॉन्फ़िगरेशन तक पहुँचते समय उपयोग**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## कॉन्फ़िगरेशन संशोधित करना
Webman डायनामिक कॉन्फ़िगरेशन परिवर्तनों का समर्थन नहीं करता है। सभी कॉन्फ़िगरेशन को संबंधित कॉन्फ़िगरेशन फाइलों में मैन्युअल रूप से संशोधित किया जाना चाहिए, फिर एप्लिकेशन को पुनः लोड या पुनः आरंभ करें।

> **ध्यान दें**
> सर्वर कॉन्फ़िगरेशन `config/server.php` और प्रोसेस कॉन्फ़िगरेशन `config/process.php` रीलोड का समर्थन नहीं करते हैं। परिवर्तन प्रभावी होने के लिए आपको पुनः आरंभ करना होगा।

## महत्वपूर्ण अनुस्मारक
यदि आप `config/` के तहत एक उपनिर्देशिका में कॉन्फ़िगरेशन फाइलें बनाते हैं, उदाहरण के लिए `config/order/status.php`, तो आपको `config/order/` निर्देशिका में निम्न सामग्री के साथ एक `app.php` फाइल की आवश्यकता है:
```php
<?php
return [
    'enable' => true,
];
```
`enable` को `true` पर सेट करने से फ्रेमवर्क को इस निर्देशिका से कॉन्फ़िगरेशन लोड करने के लिए बताया जाता है।
कॉन्फ़िगरेशन निर्देशिका संरचना इस प्रकार होनी चाहिए:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
फिर आप `status.php` से सरणी या विशिष्ट कुंजियों तक `config.order.status` के माध्यम से पहुँच सकते हैं।


## कॉन्फ़िगरेशन फाइल संदर्भ

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // सुनने का पोर्ट (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'transport' => 'tcp', // परिवहन प्रोटोकॉल (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'context' => [], // SSL आदि (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'name' => 'webman', // प्रोसेस नाम (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'count' => cpu_count() * 4, // प्रोसेस संख्या (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'user' => '', // उपयोगकर्ता (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'group' => '', // समूह (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'reusePort' => false, // पोर्ट पुनः उपयोग सक्षम करें (1.6.0 से हटाया गया, config/process.php में कॉन्फ़िगर किया गया)
    'event_loop' => '',  // इवेंट लूप क्लास, डिफ़ॉल्ट रूप से स्वतः चयनित
    'stop_timeout' => 2, // स्टॉप/रिस्टार्ट/रीलोड सिग्नल प्राप्त करने पर अधिकतम प्रतीक्षा समय, समय पर प्रोसेस निकलने पर जबरन बाहर निकलें
    'pid_file' => runtime_path() . '/webman.pid', // PID फाइल स्थान
    'status_file' => runtime_path() . '/webman.status', // स्थिति फाइल स्थान
    'stdout_file' => runtime_path() . '/logs/stdout.log', // stdout फाइल स्थान, webman शुरू होने के बाद सभी आउटपुट यहाँ लिखा जाता है
    'log_file' => runtime_path() . '/logs/workerman.log', // Workerman लॉग फाइल स्थान
    'max_package_size' => 10 * 1024 * 1024 // अधिकतम पैकेट आकार, 10M. अपलोड फाइल आकार इससे सीमित है
];
```

#### app.php
```php
return [
    'debug' => true,  // डिबग मोड, त्रुटियों पर स्टैक ट्रेस आदि सक्षम करता है। प्रोडक्शन में अक्षम किया जाना चाहिए
    'error_reporting' => E_ALL, // त्रुटि रिपोर्टिंग स्तर
    'default_timezone' => 'Asia/Shanghai', // डिफ़ॉल्ट समय क्षेत्र
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // सार्वजनिक निर्देशिका पथ
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // रनटाइम निर्देशिका पथ
    'controller_suffix' => 'Controller', // कंट्रोलर प्रत्यय
    'controller_reuse' => false, // कंट्रोलर पुनः उपयोग करें या नहीं
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman प्रोसेस कॉन्फ़िगरेशन
    'webman' => [ 
        'handler' => Http::class, // प्रोसेस हैंडलर क्लास
        'listen' => 'http://0.0.0.0:8787', // सुनने का पता
        'count' => cpu_count() * 4, // प्रोसेस संख्या, डिफ़ॉल्ट रूप से 4x CPU
        'user' => '', // प्रोसेस उपयोगकर्ता, कम विशेषाधिकार उपयोगकर्ता का उपयोग करें
        'group' => '', // प्रोसेस समूह, कम विशेषाधिकार समूह का उपयोग करें
        'reusePort' => false, // reusePort सक्षम करें, worker प्रोसेस में कनेक्शन वितरित करता है
        'eventLoop' => '', // इवेंट लूप क्लास, खाली होने पर server.event_loop उपयोग करता है
        'context' => [], // सुनने का संदर्भ, जैसे SSL
        'constructor' => [ // प्रोसेस हैंडलर के लिए कंस्ट्रक्टर पैरामीटर, यहाँ Http क्लास
            'requestClass' => Request::class, // कस्टम रिक्वेस्ट क्लास
            'logger' => Log::channel('default'), // लॉगर इंस्टेंस
            'appPath' => app_path(), // ऐप निर्देशिका पथ
            'publicPath' => public_path() // सार्वजनिक निर्देशिका पथ
        ]
    ],
    // फाइल परिवर्तन ऑटो रीलोड और मेमोरी लीक डिटेक्शन के लिए मॉनिटर प्रोसेस
    'monitor' => [
        'handler' => app\process\Monitor::class, // हैंडलर क्लास
        'reloadable' => false, // यह प्रोसेस रीलोड निष्पादित नहीं करता है
        'constructor' => [ // प्रोसेस हैंडलर के लिए कंस्ट्रक्टर पैरामीटर
            // देखने के लिए निर्देशिकाएं, बहुत अधिक न रखें क्योंकि यह डिटेक्शन धीमा करता है
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // परिवर्तन देखने के लिए फाइल एक्सटेंशन
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // अन्य विकल्प
            'options' => [
                // फाइल मॉनिटरिंग सक्षम करें, केवल Linux, डेमन मोड में फाइल मॉनिटरिंग डिफ़ॉल्ट रूप से अक्षम है
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // मेमोरी मॉनिटरिंग सक्षम करें, केवल Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11 डिपेंडेंसी इंजेक्शन कंटेनर इंस्टेंस लौटाएं
return new Webman\Container;
```

#### dependence.php
```php
// डिपेंडेंसी इंजेक्शन कंटेनर में सेवाओं और निर्भरताओं को कॉन्फ़िगर करें
return [];
```

#### route.php
```php

use support\Route;
// /test पथ के लिए रूट परिभाषित करें
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // डिफ़ॉल्ट व्यू हैंडलर क्लास
];
```

### autoload.php
```php
// फ्रेमवर्क ऑटोलोड फाइलों को कॉन्फ़िगर करें
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// कैश कॉन्फ़िगरेशन
return [
    'default' => 'file', // डिफ़ॉल्ट ड्राइवर
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // कैश फाइल स्टोरेज पथ
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Redis कनेक्शन नाम, redis.php में कॉन्फ़िगरेशन को संदर्भित करता है
        ],
        'array' => [
            'driver' => 'array' // मेमोरी में कैश, रीस्टार्ट पर साफ हो जाता है
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // डिफ़ॉल्ट डेटाबेस
 'default' => 'mysql',
 // डेटाबेस कनेक्शन कॉन्फ़िगरेशन
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // अपवाद हैंडलर क्लास सेट करें
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // हैंडलर
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // लॉग फाइल नाम
                    7, //$maxFiles // 7 दिन के लिए लॉग रखें
                    Monolog\Logger::DEBUG, // लॉग स्तर
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // फॉर्मैटर
                    'constructor' => [null, 'Y-m-d H:i:s', true], // फॉर्मैटर पैरामीटर
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // प्रकार
    'type' => 'file', // या redis या redis_cluster
     // हैंडलर
    'handler' => FileSessionHandler::class,
     // कॉन्फ़िगरेशन
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // स्टोरेज निर्देशिका
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // सत्र नाम
    'auto_update_timestamp' => false, // सत्र समाप्ति रोकने के लिए ऑटो टाइमस्टैम्प अपडेट
    'lifetime' => 7*24*60*60, // आयु
    'cookie_lifetime' => 365*24*60*60, // कुकी आयु
    'cookie_path' => '/', // कुकी पथ
    'domain' => '', // कुकी डोमेन
    'http_only' => true, // केवल HTTP
    'secure' => false, // केवल HTTPS
    'same_site' => '', // SameSite विशेषता
    'gc_probability' => [1, 1000], // सत्र गार्बेज कलेक्शन संभावना
];
```

#### middleware.php
```php
// मिडलवेयर कॉन्फ़िगर करें
return [];
```

#### static.php
```php
return [
    'enable' => true, // webman स्थिर फाइल सर्विंग सक्षम करें
    'middleware' => [ // स्थिर फाइल मिडलवेयर, कैश पॉलिसी, CORS आदि के लिए
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // डिफ़ॉल्ट भाषा
    'locale' => 'zh_CN',
    // फॉलबैक भाषाएं
    'fallback_locale' => ['zh_CN', 'en'],
    // अनुवाद फाइल स्थान
    'path' => base_path() . '/resource/translations',
];
```
