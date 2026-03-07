# webman सत्र प्रबंधन

## उदाहरण
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

`$request->session();` के माध्यम से `Workerman\Protocols\Http\Session` इंस्टैंस प्राप्त करें और इसकी विधियों का उपयोग करके सत्र डेटा जोड़ें, संशोधित करें या हटाएं।

> **ध्यान दें**
> सत्र ऑब्जेक्ट के नष्ट होने पर सत्र डेटा स्वचालित रूप से सहेजा जाता है।
> सत्र ऑब्जेक्ट को वैश्विक चर में संग्रहीत करने से ऑब्जेक्ट नष्ट नहीं होता और स्वचालित रूप से सहेजा नहीं जाता। ऐसी स्थिति में डेटा सहेजने के लिए मैन्युअल रूप से `$session->save()` को कॉल करना होगा।

## सभी सत्र डेटा प्राप्त करें
```php
$session = $request->session();
$all = $session->all();
```
यह एक ऐरे लौटाता है। यदि कोई सत्र डेटा नहीं है, तो खाली ऐरे लौटाया जाता है।


## सत्र से किसी मान को प्राप्त करें
```php
$session = $request->session();
$name = $session->get('name');
```
डेटा मौजूद नहीं होने पर null लौटाता है।

आप get विधि के दूसरे पैरामीटर के रूप में डिफ़ॉल्ट मान भी भेज सकते हैं। सत्र ऐरे में संगत मान न मिलने पर डिफ़ॉल्ट मान लौटाया जाता है। उदाहरण:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## सत्र डेटा संग्रहीत करें
एक डेटा संग्रहीत करने के लिए set विधि का उपयोग करें।
```php
$session = $request->session();
$session->set('name', 'tom');
```
set कोई मान नहीं लौटाता। सत्र ऑब्जेक्ट नष्ट होने पर सत्र स्वचालित रूप से सहेजा जाता है।

अनेक मान संग्रहीत करने के लिए put विधि का उपयोग करें।
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
इसी प्रकार put भी कोई मान नहीं लौटाता।

## सत्र डेटा हटाएं
एक या अधिक सत्र डेटा हटाने के लिए `forget` विधि का उपयोग करें।
```php
$session = $request->session();
// एक आइटम हटाएं
$session->forget('name');
// अनेक आइटम हटाएं
$session->forget(['name', 'age']);
```

सिस्टम delete विधि भी प्रदान करता है। forget से भिन्न, delete केवल एक आइटम हटा सकता है।
```php
$session = $request->session();
// $session->forget('name'); के समान
$session->delete('name');
```


## सत्र से मान प्राप्त करें और हटाएं
```php
$session = $request->session();
$name = $session->pull('name');
```
निम्नलिखित कोड के समान प्रभाव:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
संगत सत्र मौजूद न होने पर null लौटाता है।


## सभी सत्र डेटा हटाएं
```php
$request->session()->flush();
```
कोई मान नहीं लौटाता। सत्र ऑब्जेक्ट नष्ट होने पर सत्र स्वचालित रूप से स्टोरेज से हटा दिया जाता है।


## सत्र मान की उपस्थिति जांचें
```php
$session = $request->session();
$has = $session->has('name');
```
सत्र मान मौजूद न होने या null होने पर false लौटाता है; अन्यथा true लौटाता है।

```php
$session = $request->session();
$has = $session->exists('name');
```
उपरोक्त कोड भी सत्र मान की उपस्थिति जांचता है। अंतर: मान null होने पर भी `exists` true लौटाता है।

## सहायक फ़ंक्शन session()

webman समान कार्यक्षमता के लिए सहायक फ़ंक्शन `session()` प्रदान करता है।
```php
// सत्र इंस्टैंस प्राप्त करें
$session = session();
// समतुल्य
$session = $request->session();

// मान प्राप्त करें
$value = session('key', 'default');
// समतुल्य
$value = session()->get('key', 'default');
// समतुल्य
$value = $request->session()->get('key', 'default');

// सत्र को मान सौंपें
session(['key1'=>'value1', 'key2' => 'value2']);
// समतुल्य
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// समतुल्य
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## कॉन्फ़िग फ़ाइल
सत्र कॉन्फ़िग फ़ाइल `config/session.php` में है। सामग्री निम्नलिखित के समान है:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class या RedisSessionHandler::class या RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handler FileSessionHandler::class होने पर मान 'file',
    // handler RedisSessionHandler::class होने पर मान 'redis'
    // handler RedisClusterSessionHandler::class होने पर मान 'redis_cluster' (Redis क्लस्टर)
    'type'    => 'file',

    // विभिन्न हैंडलर विभिन्न कॉन्फ़िगरेशन उपयोग करते हैं
    'config' => [
        // type 'file' के लिए कॉन्फ़िगरेशन
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // type 'redis' के लिए कॉन्फ़िगरेशन
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // session_id संग्रहीत करने वाली कुकी का नाम
    'auto_update_timestamp' => false,  // सत्र स्वचालित ताज़ा करें, डिफ़ॉल्ट: बंद
    'lifetime' => 7*24*60*60,          // सत्र समाप्ति समय
    'cookie_lifetime' => 365*24*60*60, // session_id कुकी समाप्ति समय
    'cookie_path' => '/',              // session_id कुकी पथ
    'domain' => '',                    // session_id कुकी डोमेन
    'http_only' => true,               // httpOnly सक्षम करें, डिफ़ॉल्ट: सक्षम
    'secure' => false,                 // केवल HTTPS पर सत्र सक्षम, डिफ़ॉल्ट: बंद
    'same_site' => '',                 // CSRF हमले और उपयोगकर्ता ट्रैकिंग रोकने के लिए, मान: strict/lax/none
    'gc_probability' => [1, 1000],     // सत्र गार्बेज संग्रहण संभावना
];
```

## सुरक्षा
सत्र में कक्षा इंस्टैंस को सीधे संग्रहीत करने की सिफारिश नहीं की जाती, खासकर अविश्वसनीय स्रोतों से। डिसीरियलाइजेशन सुरक्षा जोखिम पैदा कर सकता है।

