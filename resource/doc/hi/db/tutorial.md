# डेटाबेस त्वरित प्रारंभ (Laravel डेटाबेस कंपोनेंट पर आधारित)

[webman/database](https://github.com/webman-php/database) [illuminate/database](https://github.com/illuminate/database) पर बना है और कनेक्शन पूल जोड़ता है, कोरूटीन और नॉन-कोरूटीन दोनों वातावरणों में काम करता है। उपयोग Laravel जैसा ही है।

आप [अन्य डेटाबेस कंपोनेंट का उपयोग](others.md) अनुभाग देखकर ThinkPHP या अन्य डेटाबेस इस्तेमाल कर सकते हैं।

## डेटाबेस स्थापना

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

स्थापना के बाद रिस्टार्ट ज़रूरी है (रिलोड काम नहीं करेगा)।

> **संकेत**
> webman/database Laravel के `illuminate/database` पर निर्भर है, इसलिए स्थापना के समय इसके डिपेंडेंसी अपने-आप इंस्टॉल हो जाते हैं।

> **ध्यान दें**
> अगर पेजिनेशन, डेटाबेस इवेंट या SQL लॉगिंग की ज़रूरत नहीं है, तो बस चलाएं:
> `composer require -W webman/database`

## डेटाबेस कॉन्फ़िगरेशन
`config/database.php`
```php

return [
    // डिफ़ॉल्ट डेटाबेस
    'default' => 'mysql',

    // कनेक्शन कॉन्फ़िगरेशन
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // swoole या swow चलाते समय ज़रूरी
            ],
            'pool' => [ // कनेक्शन पूल कॉन्फ़िगरेशन
                'max_connections' => 5, // अधिकतम कनेक्शन संख्या
                'min_connections' => 1, // न्यूनतम कनेक्शन संख्या
                'wait_timeout' => 3,    // पूल से कनेक्शन लेने की अधिकतम प्रतीक्षा; पार होने पर एक्सेप्शन। सिर्फ कोरूटीन वातावरण में
                'idle_timeout' => 60,   // पूल में कनेक्शन की अधिकतम निष्क्रियता; पार होने पर बंद कर min_connections तक लाया जाता है
                'heartbeat_interval' => 50, // पूल हार्टबीट अंतराल (सेकंड); 60 से कम सुझाया गया
            ],
        ],
    ],
];
```

`pool` कॉन्फ़िगरेशन के अलावा बाकी Laravel जैसा है।

## कनेक्शन पूल के बारे में
* हर प्रोसेस का अपना पूल है; पूल प्रोसेसों के बीच साझा नहीं होता।
* कोरूटीन बंद होने पर रिक्वेस्टें क्रम में चलती हैं, कोई समवर्तन नहीं, इसलिए पूल में अधिकतम एक कनेक्शन होता है।
* कोरूटीन चालू होने पर रिक्वेस्टें एक साथ चलती हैं; पूल ज़रूरत के हिसाब से कनेक्शन संख्या बदलता है, `max_connections` से ऊपर नहीं जाता, `min_connections` से नीचे नहीं आता।
* पूल में अधिकतम `max_connections` होने के कारण, जब डेटाबेस इस्तेमाल करने वाले कोरूटीन इससे ज़्यादा हो जाते हैं, तो कुछ कतार में `wait_timeout` सेकंड तक इंतज़ार करते हैं; पार होने पर एक्सेप्शन होता है।
* निष्क्रिय होने पर (कोरूटीन के साथ या बिना) कनेक्शन `idle_timeout` के बाद वापस ले लिए जाते हैं, `min_connections` तक (`min_connections` 0 हो सकता है)।

## डेटाबेस उपयोग उदाहरण
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

उपयोग Laravel जैसा है; `Db::table()` मेथड से डेटाबेस पर काम करें।
