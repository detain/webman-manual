# Stomp कतार

Stomp एक सरल (स्ट्रीमिंग) पाठ-उन्मुख संदेश प्रोटोकॉल है जो एक इंटरऑपरेबल कनेक्शन प्रारूप प्रदान करता है, जिससे STOMP क्लाइंट किसी भी STOMP संदेश ब्रोकर (Broker) के साथ बातचीत कर सकते हैं। [workerman/stomp](https://github.com/walkor/stomp) Stomp क्लाइंट को लागू करता है और मुख्यतः RabbitMQ, Apollo, ActiveMQ आदि संदेश कतार परिदृश्यों के लिए उपयोग किया जाता है।

## स्थापना
`composer require webman/stomp`

## कॉन्फ़िगरेशन
कॉन्फ़िगरेशन फ़ाइल `config/plugin/webman/stomp` के अंतर्गत है।

## संदेश भेजना
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // कतार
        $queue = 'examples';
        // डेटा (सरणी भेजते समय स्वयं सीरियलाइज़ करना आवश्यक है, जैसे json_encode, serialize आदि)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // भेजना निष्पादित करें
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> अन्य परियोजनाओं के साथ अनुकूलता के लिए Stomp घटक स्वचालित सीरियलाइज़ेशन और डीसीरियलाइज़ेशन प्रदान नहीं करता। सरणी डेटा भेजने पर स्वयं सीरियलाइज़ करना और उपभोग के समय स्वयं डीसीरियलाइज़ करना आवश्यक है।

## संदेश उपभोग
`app/queue/stomp/MyMailSend.php` नया बनाएं (क्लास नाम मनमाना हो सकता है, PSR-4 मानक के अनुसार)।
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // कतार का नाम
    public $queue = 'examples';

    // कनेक्शन नाम, stomp.php में कनेक्शन के अनुरूप
    public $connection = 'default';

    // मान client होने पर सर्वर को सफल उपभोग की सूचना देने के लिए $ack_resolver->ack() कॉल करना आवश्यक है
    // मान auto होने पर $ack_resolver->ack() कॉल करने की आवश्यकता नहीं है
    public $ack = 'auto';

    // उपभोग
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // यदि डेटा सरणी है तो स्वयं डीसीरियलाइज़ करना आवश्यक है
        var_export(json_decode($data, true)); // आउटपुट ['to' => 'tom@gmail.com', 'content' => 'hello']
        // सर्वर को सफल उपभोग की सूचना दें
        $ack_resolver->ack(); // ack auto होने पर यह कॉल छोड़ी जा सकती है
    }
}
```

# RabbitMQ में Stomp प्रोटोकॉल सक्षम करना
RabbitMQ डिफ़ॉल्ट रूप से Stomp प्रोटोकॉल सक्षम नहीं करता। सक्षम करने के लिए निम्न कमांड चलाएं:
```
rabbitmq-plugins enable rabbitmq_stomp
```
सक्षम करने के बाद Stomp का डिफ़ॉल्ट पोर्ट 61613 है।
