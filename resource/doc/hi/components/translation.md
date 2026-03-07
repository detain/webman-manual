# बहुभाषी

बहुभाषी सपोर्ट [symfony/translation](https://github.com/symfony/translation) कंपोनेंट का उपयोग करता है।

## इंस्टॉलेशन
```
composer require symfony/translation
```

## भाषा पैकेज बनाना
webman डिफ़ॉल्ट रूप से भाषा पैकेज `resource/translations` फ़ोल्डर में रखता है (अगर नहीं है तो खुद बनाएं)। फ़ोल्डर बदलने के लिए `config/translation.php` में सेट करें।
हर भाषा का अपना सबफ़ोल्डर है, भाषा परिभाषाएं डिफ़ॉल्ट रूप से `messages.php` में रहती हैं। उदाहरण:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

सभी भाषा फाइलें एक ऐरे लौटाती हैं, उदाहरण:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## कॉन्फिगरेशन

`config/translation.php`

```php
return [
    // डिफ़ॉल्ट भाषा
    'locale' => 'zh_CN',
    // फॉलबैक भाषा: जब मौजूदा भाषा में अनुवाद न मिले तो फॉलबैक भाषा का अनुवाद चाहें
    'fallback_locale' => ['zh_CN', 'en'],
    // भाषा फाइलों का फ़ोल्डर
    'path' => base_path() . '/resource/translations',
];
```

## अनुवाद

अनुवाद के लिए `trans()` मेथड का उपयोग करें।

भाषा फाइल `resource/translations/zh_CN/messages.php` बनाएं:
```php
return [
    'hello' => '你好 世界!',
];
```

फाइल `app/controller/UserController.php` बनाएं:
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

`http://127.0.0.1:8787/user/get` पर जाने पर "你好 世界!" मिलेगा।

## डिफ़ॉल्ट भाषा बदलना

भाषा बदलने के लिए `locale()` मेथड का उपयोग करें।

भाषा फाइल `resource/translations/en/messages.php` जोड़ें:
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
        // भाषा बदलें
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
`http://127.0.0.1:8787/user/get` पर जाने पर "hello world!" मिलेगा।

`trans()` के चौथे पैरामीटर से अस्थायी तौर पर भाषा भी बदल सकते हैं। मिसाल के लिए ऊपर वाला और नीचे वाला दोनों बराबर हैं:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // चौथा पैरामीटर भाषा बदलता है
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## हर रिक्वेस्ट के लिए भाषा साफ़ तौर पर सेट करना
translation एक सिंगलटन है, यानी सभी रिक्वेस्ट एक ही इंस्टेंस शेयर करते हैं। अगर किसी रिक्वेस्ट में `locale()` से डिफ़ॉल्ट भाषा सेट की जाए तो उस प्रोसेस की बाकी सभी रिक्वेस्ट पर असर पड़ेगा। इसलिए हर रिक्वेस्ट के लिए भाषा साफ़ तौर पर सेट करनी चाहिए। मिसाल के लिए नीचे वाला मिडलवेयर इस्तेमाल करें:

फाइल `app/middleware/Lang.php` बनाएं (फ़ोल्डर नहीं है तो खुद बनाएं):
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

`config/middleware.php` में ग्लोबल मिडलवेयर जोड़ें:
```php
return [
    // ग्लोबल मिडलवेयर
    '' => [
        // ... दूसरे मिडलवेयर छोड़े गए
        app\middleware\Lang::class,
    ]
];
```


## प्लेसहोल्डर का उपयोग
कभी-कभी मैसेज में अनुवाद करने वाले वैरिएबल होते हैं, जैसे
```php
trans('hello ' . $name);
```
ऐसे में प्लेसहोल्डर इस्तेमाल करें।

`resource/translations/zh_CN/messages.php` अपडेट करें:
```php
return [
    'hello' => '你好 %name%!',
];
```
अनुवाद करते समय दूसरे पैरामीटर से प्लेसहोल्डर के वैल्यू पास करें:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## बहुवचन संभालना
कुछ भाषाओं में मात्रा के हिसाब से वाक्य बदलता है। जैसे `There is %count% apple` तभी सही है जब `%count%` 1 हो, 1 से ज़्यादा होने पर गलत।

ऐसे में **पाइप** (`|`) से बहुवचन रूप लिखें।

भाषा फाइल `resource/translations/en/messages.php` में `apple_count` जोड़ें:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

नंबर रेंज से जटिल बहुवचन नियम भी बना सकते हैं:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## भाषा फाइल निर्दिष्ट करना

डिफ़ॉल्ट फाइल का नाम `messages.php` है, लेकिन दूसरे नाम की फाइल भी बना सकते हैं।

भाषा फाइल `resource/translations/zh_CN/admin.php` बनाएं:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

`trans()` के तीसरे पैरामीटर से भाषा फाइल निर्दिष्ट करें (`.php` एक्सटेंशन हटाएं)। 
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## और जानकारी
[symfony/translation मैनुअल](https://symfony.com/doc/current/translation.html) देखें
