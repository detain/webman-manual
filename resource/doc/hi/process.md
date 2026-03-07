# कस्टम प्रोसेस

webman में आप workerman की तरह कस्टम लिसनर या प्रोसेस बना सकते हैं।

> **ध्यान दें**
> Windows उपयोगकर्ताओं को कस्टम प्रोसेस चलाने के लिए webman शुरू करने हेतु `php windows.php` का उपयोग करना होगा।

## कस्टम HTTP सेवा
कभी-कभी आपको विशेष आवश्यकता हो सकती है जिसके लिए webman HTTP सेवा का मुख्य कोड बदलना पड़ सकता है। ऐसे में कस्टम प्रोसेस का उपयोग करें।

उदाहरण के लिए नया `app\Server.php` बनाएँ।

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // यहाँ Webman\App के मेथड ओवरराइट करें
}
```

`config/process.php` में निम्न कॉन्फ़िगरेशन जोड़ें।

```php
use Workerman\Worker;

return [
    // ... अन्य कॉन्फ़िगरेशन छोड़े गए ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // प्रोसेस संख्या
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // अनुरोध क्लास सेट करें
            'logger' => \support\Log::channel('default'), // लॉग इंस्टेंस
            'appPath' => app_path(), // app निर्देशिका स्थान
            'publicPath' => public_path() // public निर्देशिका स्थान
        ]
    ]
];
```

> **सुझाव**
> webman के अंतर्निहित HTTP प्रोसेस को बंद करने के लिए बस `config/server.php` में `listen=>''` सेट करें।

## कस्टम WebSocket लिसनर उदाहरण

नया `app/Pusher.php` बनाएँ।

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> ध्यान दें: सभी onXXX मेथड public होने चाहिए।

`config/process.php` में निम्न कॉन्फ़िगरेशन जोड़ें।

```php
return [
    // ... अन्य प्रोसेस कॉन्फ़िगरेशन छोड़े गए ...
    
    // websocket_test प्रोसेस का नाम है
    'websocket_test' => [
        // यहाँ प्रोसेस क्लास निर्दिष्ट करें, ऊपर परिभाषित Pusher क्लास
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## कस्टम नॉन-लिसनिंग प्रोसेस उदाहरण

नया `app/TaskTest.php` बनाएँ।

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // हर 10 सेकंड में डेटाबेस जाँचें कि नए उपयोगकर्ता ने पंजीकरण किया है या नहीं
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

`config/process.php` में निम्न कॉन्फ़िगरेशन जोड़ें।

```php
return [
    // ... अन्य प्रोसेस कॉन्फ़िगरेशन छोड़े गए ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> ध्यान दें: listen छोड़ने पर कोई पोर्ट नहीं सुनेगा, count छोड़ने पर प्रोसेस संख्या डिफ़ॉल्ट 1 होगी।

## कॉन्फ़िगरेशन फ़ाइल विवरण

प्रोसेस का पूर्ण कॉन्फ़िगरेशन इस प्रकार परिभाषित है:

```php
return [
    // ... 
    
    // websocket_test प्रोसेस का नाम है
    'websocket_test' => [
        // यहाँ प्रोसेस क्लास निर्दिष्ट करें
        'handler' => app\Pusher::class,
        // लिसनिंग प्रोटोकॉल, IP और पोर्ट (वैकल्पिक)
        'listen'  => 'websocket://0.0.0.0:8888',
        // प्रोसेस संख्या (वैकल्पिक, डिफ़ॉल्ट 1)
        'count'   => 2,
        // प्रोसेस चलाने वाला उपयोगकर्ता (वैकल्पिक, डिफ़ॉल्ट वर्तमान उपयोगकर्ता)
        'user'    => '',
        // प्रोसेस चलाने वाला उपयोगकर्ता समूह (वैकल्पिक, डिफ़ॉल्ट वर्तमान समूह)
        'group'   => '',
        // वर्तमान प्रोसेस reload समर्थन करता है या नहीं (वैकल्पिक, डिफ़ॉल्ट true)
        'reloadable' => true,
        // reusePort सक्षम करें
        'reusePort'  => true,
        // transport (वैकल्पिक, SSL चाहिए तो ssl सेट करें, डिफ़ॉल्ट tcp)
        'transport'  => 'tcp',
        // context (वैकल्पिक, transport ssl होने पर सर्टिफिकेट पथ पास करें)
        'context'    => [], 
        // प्रोसेस क्लास कंस्ट्रक्टर पैरामीटर (वैकल्पिक)
        'constructor' => [],
        // यह प्रोसेस सक्षम है या नहीं
        'enable' => true
    ],
];
```

## सारांश

webman का कस्टम प्रोसेस वास्तव में workerman का सरल इनकैप्सुलेशन है। यह कॉन्फ़िगरेशन और बिजनेस लॉजिक अलग करता है और workerman के `onXXX` कॉलबैक को क्लास मेथड के माध्यम से लागू करता है। अन्य उपयोग workerman के साथ पूरी तरह समान है।
