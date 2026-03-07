# डिपेंडेंसी ऑटो इंजेक्शन

webman में डिपेंडेंसी ऑटो इंजेक्शन एक वैकल्पिक सुविधा है और डिफ़ॉल्ट रूप से बंद रहती है। अगर आपको डिपेंडेंसी ऑटो इंजेक्शन चाहिए तो [php-di](https://php-di.org/doc/getting-started.html) के इस्तेमाल की सिफारिश की जाती है। नीचे webman के साथ `php-di` के इस्तेमाल का तरीका बताया गया है।

## इंस्टॉलेशन

```
composer require php-di/php-di:^7.0
```

कॉन्फ़िगरेशन `config/container.php` को संशोधित करें। अंतिम कंटेंट इस प्रकार होना चाहिए:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> `config/container.php` फ़ाइल अंत में PSR-11 स्पेसिफिकेशन के अनुरूप एक कंटेनर इंस्टेंस रिटर्न करनी चाहिए। अगर आप `php-di` नहीं चाहते तो यहाँ दूसरा PSR-11 अनुरूप कंटेनर इंस्टेंस बनाकर रिटर्न कर सकते हैं। डिफ़ॉल्ट कॉन्फ़िगरेशन केवल webman की बेसिक कंटेनर क्षमता देता है।

## कंस्ट्रक्टर इंजेक्शन

`app/service/Mailer.php` बनाएँ (जरूरत हो तो फ़ोल्डर खुद बनाएँ) और निम्न कंटेंट रखें:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // ईमेल भेजने का कोड छोड़ा गया
    }
}
```

`app/controller/UserController.php` का कंटेंट:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

आम तौर पर `app\controller\UserController` का इंस्टेंस बनाने के लिए यह कोड चाहिए होगा:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

`php-di` के साथ डेवलपर्स को कंट्रोलर में `Mailer` मैन्युअल इंस्टेंस नहीं बनाना होता — webman खुद कर देता है। अगर `Mailer` इंस्टेंस बनाते समय दूसरी क्लास डिपेंडेंसी हों तो webman उन्हें भी इंस्टेंस और इंजेक्ट कर देगा। डेवलपर को कोई इनिशियलाइज़ेशन नहीं करनी पड़ती।

> **ध्यान**
> डिपेंडेंसी ऑटो इंजेक्शन सिर्फ उन इंस्टेंस के साथ काम करता है जो फ्रेमवर्क या `php-di` द्वारा बनाए गए हों। `new` से मैन्युअल बनाए इंस्टेंस पर नहीं। इंजेक्शन के लिए `new` की जगह `support\Container` इंटरफ़ेस इस्तेमाल करें, जैसे:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// new से बनाए इंस्टेंस पर डिपेंडेंसी इंजेक्शन काम नहीं करता
$user_service = new UserService;
// new से बनाए इंस्टेंस पर डिपेंडेंसी इंजेक्शन काम नहीं करता
$log_service = new LogService($path, $name);

// Container से बनाए इंस्टेंस पर डिपेंडेंसी इंजेक्शन काम करता है
$user_service = Container::get(UserService::class);
// Container से बनाए इंस्टेंस पर डिपेंडेंसी इंजेक्शन काम करता है
$log_service = Container::make(LogService::class, [$path, $name]);
```

## एट्रिब्यूट इंजेक्शन

कंस्ट्रक्टर डिपेंडेंसी इंजेक्शन के अलावा एट्रिब्यूट इंजेक्शन भी इस्तेमाल कर सकते हैं। पिछले उदाहरण को आगे बढ़ाते हुए `app\controller\UserController` को ऐसे बदलें:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

यह उदाहरण इंजेक्शन के लिए `#[Inject]` एट्रिब्यूट इस्तेमाल करता है और ऑब्जेक्ट टाइप के हिसाब से इंस्टेंस को मेंबर वेरिएबल में ऑटो इंजेक्ट करता है। असर कंस्ट्रक्टर इंजेक्शन जैसा ही है लेकिन कोड ज्यादा संक्षिप्त है।

> **ध्यान**
> webman वर्ज़न 1.4.6 से पहले कंट्रोलर पैरामीटर इंजेक्शन सपोर्ट नहीं करता। जैसे नीचे वाला कोड webman<=1.4.6 पर सपोर्टेड नहीं है:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // 1.4.6 से पहले कंट्रोलर पैरामीटर इंजेक्शन सपोर्ट नहीं
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

## कस्टम कंस्ट्रक्टर इंजेक्शन

कभी-कभी कंस्ट्रक्टर में पास होने वाले पैरामीटर क्लास इंस्टेंस नहीं होते बल्कि स्ट्रिंग, नंबर, ऐरे आदि होते हैं। जैसे Mailer कंस्ट्रक्टर को SMTP सर्वर IP और पोर्ट चाहिए:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // ईमेल भेजने का कोड छोड़ा गया
    }
}
```

इस स्थिति में पहले वाला कंस्ट्रक्टर ऑटो इंजेक्शन सीधे नहीं चल सकता क्योंकि `php-di` को `$smtp_host` और `$smtp_port` के वैल्यू नहीं पता। ऐसे में कस्टम इंजेक्शन इस्तेमाल करें।

`config/dependence.php` में (फाइल न हो तो बनाएँ) यह कोड जोड़ें:

```php
return [
    // ... दूसरी कॉन्फ़िगरेशन छोड़ी गई

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

जब डिपेंडेंसी इंजेक्शन को `app\service\Mailer` इंस्टेंस चाहिए होगा तो इस कॉन्फ़िगरेशन में बना इंस्टेंस ऑटो इस्तेमाल हो जाएगा।

ध्यान दें कि `config/dependence.php` में `Mailer` क्लास इंस्टेंस के लिए `new` इस्तेमाल हो रहा है। इस उदाहरण में कोई दिक्कत नहीं, लेकिन अगर `Mailer` क्लास दूसरी क्लास पर डिपेंड करती है या अंदर एट्रिब्यूट इंजेक्शन करती है तो `new` से इनिशियलाइज़ करने पर डिपेंडेंसी ऑटो इंजेक्शन नहीं होगा। समाधान कस्टम इंटरफ़ेस इंजेक्शन और `Container::get(क्लास नाम)` या `Container::make(क्लास नाम, [कंस्ट्रक्टर पैरामीटर्स])` से क्लास इनिशियलाइज़ करना है।

## कस्टम इंटरफ़ेस इंजेक्शन

असल प्रोजेक्ट में कंक्रीट क्लास की जगह इंटरफ़ेस पर प्रोग्राम करना बेहतर होता है। जैसे `app\controller\UserController` को `app\service\Mailer` की जगह `app\service\MailerInterface` पर डिपेंड करना चाहिए।

`MailerInterface` इंटरफ़ेस डिफाइन करें:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

`MailerInterface` का इम्प्लीमेंटेशन डिफाइन करें:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // ईमेल भेजने का कोड छोड़ा गया
    }
}
```

कंक्रीट इम्प्लीमेंटेशन की जगह `MailerInterface` इस्तेमाल करें:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Hello and welcome!');
        return response('ok');
    }
}
```

`config/dependence.php` में `MailerInterface` का इम्प्लीमेंटेशन डिफाइन करें:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

जब बिजनेस को `MailerInterface` चाहिए होगा तो ऑटो `Mailer` इम्प्लीमेंटेशन इस्तेमाल होगा।

> इंटरफ़ेस पर प्रोग्राम करने का फायदा यह है कि कम्पोनेंट बदलने पर बिजनेस कोड नहीं बदलना पड़ता — सिर्फ `config/dependence.php` में कंक्रीट इम्प्लीमेंटेशन। यूनिट टेस्ट के लिए भी बहुत उपयोगी।

## दूसरे कस्टम इंजेक्शन

क्लास डिपेंडेंसी के अलावा `config/dependence.php` में स्ट्रिंग, नंबर, ऐरे आदि भी डिफाइन किए जा सकते हैं।

मान लें `config/dependence.php` ऐसे डिफाइन है:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

तो `#[Inject]` से `smtp_host` और `smtp_port` को क्लास प्रॉपर्टी में इंजेक्ट कर सकते हैं:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // ईमेल भेजने का कोड छोड़ा गया
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // 192.168.1.11:25 आउटपुट होगा
    }
}
```

# लेज़ी लोडिंग

> लेज़ी लोडिंग एक डिज़ाइन पैटर्न है जो ऑब्जेक्ट के क्रिएशन या इनिशियलाइज़ेशन को तब तक टाल देता है जब तक वह सच में जरूरी न हो।

इस सुविधा के लिए एक अतिरिक्त डिपेंडेंसी चाहिए। नीचे वाला पैकेज `ocramius/proxy-manager` का फोर्क है; ओरिजिनल रिपॉजिटरी PHP 8 सपोर्ट नहीं करती।

```
composer require friendsofphp/proxy-manager-lts
```

इस्तेमाल:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass इंस्टेंस बना\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "प्रॉक्सी क्लास नाम: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

आउटपुट:

```
प्रॉक्सी क्लास नाम: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass इंस्टेंस बना
name: Lazy Loaded Object
```

यह उदाहरण दिखाता है कि `#[Injectable]` एट्रिब्यूट वाली क्लास इंजेक्ट होने पर पहले उसकी प्रॉक्सी क्लास बनती है। असली क्लास तभी इंस्टेंस होती है जब उसका कोई मेथड कॉल होता है।

# सर्कुलर डिपेंडेंसी

सर्कुलर डिपेंडेंसी तब होती है जब कई क्लास एक-दूसरे पर निर्भर होकर एक बंद डिपेंडेंसी लूप बनाती हैं।

- डायरेक्ट सर्कुलर डिपेंडेंसी
  - मॉड्यूल A मॉड्यूल B पर डिपेंड करता है, मॉड्यूल B मॉड्यूल A पर
  - A → B → A लूप बनता है

- इनडायरेक्ट सर्कुलर डिपेंडेंसी
  - कई मॉड्यूल डिपेंडेंसी साइकिल में शामिल
  - जैसे A → B → C → A

एट्रिब्यूट इंजेक्शन इस्तेमाल करने पर `php-di` सर्कुलर डिपेंडेंसी ऑटो डिटेक्ट करके एक्सेप्शन फेंकता है। जरूरत हो तो इस तरीके को अपनाएँ:

```php
class userController
{

    // यह कोड हटाएँ
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## अधिक जानकारी

कृपया [php-di डॉक्युमेंटेशन](https://php-di.org/doc/getting-started.html) देखें।
