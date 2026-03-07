# Redis कतार

Redis पर आधारित संदेश कतार, विलंबित संदेश प्रसंस्करण का समर्थन करती है।

## स्थापना
`composer require webman/redis-queue`

## विन्यास फ़ाइल
Redis विन्यास फ़ाइल `{मुख्य-प्रोजेक्ट}/config/plugin/webman/redis-queue/redis.php` में स्वचालित रूप से जनरेट होती है, सामग्री कुछ इस प्रकार होती है:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // पासवर्ड, वैकल्पिक
            'db' => 0,            // डेटाबेस
            'max_attempts'  => 5, // सेवन विफलता के बाद पुनः प्रयास संख्या
            'retry_seconds' => 5, // पुनः प्रयास अंतराल, सेकंड में
        ]
    ],
];
```

### सेवन विफलता पुनः प्रयास
यदि सेवन विफल होता है (एक्सेप्शन होती है), तो संदेश विलंबित कतार में डाला जाता है और अगले पुनः प्रयास की प्रतीक्षा करता है। पुनः प्रयास संख्या `max_attempts` द्वारा नियंत्रित है, अंतराल `retry_seconds` और `max_attempts` द्वारा संयुक्त रूप से। उदाहरण: यदि `max_attempts` 5 और `retry_seconds` 10 है, तो पहला पुनः प्रयास अंतराल `1*10` सेकंड, दूसरा `2*10` सेकंड, तीसरा `3*10` सेकंड, 5 पुनः प्रयास तक। यदि पुनः प्रयास संख्या `max_attempts` सेट से अधिक हो जाती है, तो संदेश `{redis-queue}-failed` कुंजी वाली विफलता कतार में डाला जाता है।

## संदेश वितरण (तुल्यकालिक)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // कतार नाम
        $queue = 'send-mail';
        // डेटा, सीधे ऐरे पास कर सकते हैं, सीरियलाइजेशन की ज़रूरत नहीं
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // संदेश भेजें
        Redis::send($queue, $data);
        // विलंबित संदेश भेजें, 60 सेकंड बाद प्रसंस्करण होगा
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
सफल वितरण पर `Redis::send()` true लौटाता है, नहीं तो false या एक्सेप्शन।

> **सुझाव**
> विलंबित कतार सेवन समय में विचलन हो सकता है। उदाहरण: जब सेवन गति उत्पादन गति से धीमी हो, कतार अटक सकती है और सेवन विलंबित हो सकता है। निवारण: अधिक उपभोक्ता प्रक्रियाएँ चलाएं।

## संदेश वितरण (अतुल्यकालिक)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // कतार नाम
        $queue = 'send-mail';
        // डेटा, सीधे ऐरे पास कर सकते हैं, सीरियलाइजेशन की ज़रूरत नहीं
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // संदेश भेजें
        Client::send($queue, $data);
        // विलंबित संदेश भेजें, 60 सेकंड बाद प्रसंस्करण होगा
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` कोई मान नहीं लौटाता। यह अतुल्यकालिक पुश है और Redis तक 100% संदेश वितरण की गारंटी नहीं देता।

> **सुझाव**
> `Client::send()` का सिद्धांत स्थानीय मेमोरी में मेमोरी कतार बनाना और संदेशों को Redis में अतुल्यकालिक सिंक करना है (सिंक तेज़ है, लगभग प्रति सेकंड 10,000 संदेश)। यदि प्रक्रिया रीस्टार्ट हो और स्थानीय मेमोरी कतार का डेटा पूरी तरह सिंक न हुआ हो, तो संदेश हानि हो सकती है। `Client::send()` अतुल्यकालिक वितरण गैर-महत्वपूर्ण संदेशों के लिए उपयुक्त है।

> **सुझाव**
> `Client::send()` अतुल्यकालिक है और केवल Workerman रनटाइम में इस्तेमाल हो सकता है। कमांड लाइन स्क्रिप्ट के लिए तुल्यकालिक इंटरफ़ेस `Redis::send()` का उपयोग करें।

## अन्य प्रोजेक्ट से संदेश भेजना
कभी-कभी दूसरे प्रोजेक्ट से संदेश भेजने होते हैं और `webman\redis-queue` इस्तेमाल नहीं कर सकते। ऐसे मामले में निम्न फ़ंक्शन से कतार में संदेश भेज सकते हैं।

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

यहाँ पैरामीटर `$redis` Redis इंस्टेंस है। उदाहरण: redis एक्सटेंशन उपयोग इस प्रकार है:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## सेवन
उपभोक्ता प्रक्रिया विन्यास फ़ाइल `{मुख्य-प्रोजेक्ट}/config/plugin/webman/redis-queue/process.php` में है। उपभोक्ता निर्देशिका `{मुख्य-प्रोजेक्ट}/app/queue/redis/` के अंतर्गत है।

कमांड `php webman redis-queue:consumer my-send-mail` चलाने पर फ़ाइल `{मुख्य-प्रोजेक्ट}/app/queue/redis/MyMailSend.php` जनरेट होगी।

> **सुझाव**
> इस कमांड के लिए [कंसोल](../plugin/console.md) प्लगइन इंस्टॉल करना ज़रूरी है। इंस्टॉल न करना चाहें तो निम्न जैसा मैन्युअल बना सकते हैं:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // सेवन करने वाली कतार का नाम
    public $queue = 'send-mail';

    // कनेक्शन नाम, plugin/webman/redis-queue/redis.php में कनेक्शन से संगत
    public $connection = 'default';

    // सेवन
    public function consume($data)
    {
        // डिसीरियलाइज़ेशन की ज़रूरत नहीं
        var_export($data); // आउटपुट ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // सेवन विफलता कॉलबैक
    /* 
    $package = [
        'id' => 1357277951, // संदेश ID
        'time' => 1709170510, // संदेश समय
        'delay' => 0, // विलंब समय
        'attempts' => 2, // सेवन संख्या
        'queue' => 'send-mail', // कतार नाम
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // संदेश सामग्री
        'max_attempts' => 5, // अधिकतम पुनः प्रयास
        'error' => 'त्रुटि संदेश' // त्रुटि संदेश
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // डिसीरियलाइज़ेशन की ज़रूरत नहीं
        var_export($package); 
    }
}
```

> **ध्यान दें**
> सेवन के दौरान कोई एक्सेप्शन या Error न फेंकने पर सेवन सफल माना जाता है; नहीं तो विफल और संदेश पुनः प्रयास कतार में जाता है। redis-queue में ack मैकेनिज़्म नहीं है; इसे ऑटो ack समझ सकते हैं (कोई एक्सेप्शन या Error न होने पर)। वर्तमान संदेश सफलतापूर्वक सेवन नहीं हुआ चिह्नित करने के लिए मैन्युअल एक्सेप्शन फेंक कर पुनः प्रयास कतार में भेज सकते हैं। व्यवहार में यह ack मैकेनिज़्म से अलग नहीं है।

> **सुझाव**
> उपभोक्ता बहु-सर्वर और बहु-प्रक्रिया का समर्थन करते हैं, और एक ही संदेश दो बार सेवन नहीं होगा। सेवन किए संदेश कतार से अपने आप हटा दिए जाते हैं; मैन्युअल हटाने की ज़रूरत नहीं।

> **सुझाव**
> उपभोक्ता प्रक्रियाएं एक साथ कई अलग-अलग कतारें सेवन कर सकती हैं। नई कतार जोड़ने के लिए `process.php` में विन्यास बदलने की ज़रूरत नहीं। नई कतार उपभोक्ता जोड़ने पर बस `app/queue/redis` के अंतर्गत संगत `Consumer` क्लास जोड़ें और `$queue` प्रॉपर्टी से सेवन करने वाली कतार का नाम निर्दिष्ट करें।

> **सुझाव**
> Windows उपयोगकर्ताओं को webman स्टार्ट करने के लिए `php windows.php` चलाना होगा, नहीं तो उपभोक्ता प्रक्रिया स्टार्ट नहीं होगी।

> **सुझाव**
> onConsumeFailure कॉलबैक हर सेवन विफलता पर ट्रिगर होता है। विफलता के बाद की लॉजिक यहाँ संभाल सकते हैं। (इस सुविधा के लिए `webman/redis-queue>=1.3.2` और `workerman/redis-queue>=1.2.1` ज़रूरी है)

## अलग-अलग कतारों के लिए अलग उपभोक्ता प्रक्रियाएं
डिफ़ॉल्ट में सभी उपभोक्ता एक ही प्रक्रिया साझा करते हैं। कभी-कभी कुछ कतारों का सेवन अलग करना होता है—जैसे धीमे सेवन वाले व्यवसाय को एक प्रक्रिया समूह में, तेज़ सेवन वाले को दूसरे में। इसके लिए उपभोक्ताओं को दो निर्देशिकाओं में बाँट सकते हैं, जैसे `app_path() . '/queue/redis/fast'` और `app_path() . '/queue/redis/slow'` (उपभोक्ता क्लास का नेमस्पेस भी अपडेट करना होगा)। विन्यास निम्नानुसार:
```php
return [
    ...दूसरा विन्यास छोड़ा गया...
    
    'redis_consumer_fast'  => [ // कुंजी कस्टम है, प्रारूप प्रतिबंध नहीं, यहाँ redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // उपभोक्ता क्लास निर्देशिका
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // कुंजी कस्टम है, प्रारूप प्रतिबंध नहीं, यहाँ redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // उपभोक्ता क्लास निर्देशिका
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

इस तरह तेज़ व्यवसाय उपभोक्ता `queue/redis/fast` निर्देशिका में जाते हैं और धीमे उपभोक्ता `queue/redis/slow` में, कतारों को उपभोक्ता प्रक्रियाएं निर्दिष्ट करने का लक्ष्य हासिल हो जाता है।

## एकाधिक Redis विन्यास
#### विन्यास
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // पासवर्ड, स्ट्रिंग प्रकार, वैकल्पिक
            'db' => 0,            // डेटाबेस
            'max_attempts'  => 5, // सेवन विफलता के बाद पुनः प्रयास
            'retry_seconds' => 5, // पुनः प्रयास अंतराल, सेकंड में
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // पासवर्ड, स्ट्रिंग प्रकार, वैकल्पिक
            'db' => 0,            // डेटाबेस
            'max_attempts'  => 5, // सेवन विफलता के बाद पुनः प्रयास
            'retry_seconds' => 5, // पुनः प्रयास अंतराल, सेकंड में
        ]
    ],
];
```

ध्यान दें: विन्यास में `other` कुंजी वाला अतिरिक्त Redis विन्यास जोड़ा गया है।

#### एकाधिक Redis में संदेश भेजना

```php
// कुंजी `default` वाली कतार में संदेश भेजें
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// समान
Client::send($queue, $data);
Redis::send($queue, $data);

// कुंजी `other` वाली कतार में संदेश भेजें
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### एकाधिक Redis से सेवन
विन्यास में कुंजी `other` वाली कतार से संदेश सेवन:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // सेवन करने वाली कतार का नाम
    public $queue = 'send-mail';

    // === विन्यास में कुंजी 'other' वाली कतार से सेवन के लिए यहाँ 'other' सेट करें ===
    public $connection = 'other';

    // सेवन
    public function consume($data)
    {
        // डिसीरियलाइज़ेशन की ज़रूरत नहीं
        var_export($data);
    }
}
```

## अक्सर पूछे जाने वाले प्रश्न

**त्रुटि `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` क्यों आती है?**

यह त्रुटि केवल अतुल्यकालिक वितरण इंटरफ़ेस `Client::send()` में होती है। अतुल्यकालिक वितरण पहले संदेश स्थानीय मेमोरी में सहेजता है, फिर प्रक्रिया आइडल होने पर Redis को भेजता है। अगर Redis संदेश उत्पादन गति से धीरे प्राप्त करता है, या प्रक्रिया अन्य काम में व्यस्त रहती है और मेमोरी से Redis तक संदेश सिंक करने का पर्याप्त समय नहीं मिलता, तो संदेश अटक सकते हैं। 600 सेकंड से अधिक संदेश अटकने पर यह त्रुटि ट्रिगर होती है।

समाधान: संदेश वितरण के लिए तुल्यकालिक वितरण इंटरफ़ेस `Redis::send()` का उपयोग करें।
