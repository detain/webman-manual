# धीमा व्यवसायिक प्रसंस्करण

कभी-कभी हमें धीमे व्यवसाय को संसाधित करना पड़ता है। धीमे व्यवसाय द्वारा webman के अन्य अनुरोध प्रसंस्करण को प्रभावित न करने के लिए, स्थिति के अनुसार इन व्यवसायों के लिए विभिन्न समाधान उपयोग किए जा सकते हैं।

## विकल्प 1: संदेश कतार का उपयोग
[Redis कतार](../queue/redis.md) [Stomp कतार](../queue/stomp.md) देखें

#### लाभ
अचानक बड़े पैमाने पर प्रसंस्करण अनुरोधों का सामना कर सकता है

#### नुकसान
क्लाइंट को सीधे परिणाम वापस नहीं कर सकता। परिणाम पुश करने के लिए अन्य सेवाओं के साथ समन्वय आवश्यक है, जैसे [webman/push](https://www.workerman.net/plugin/2) का उपयोग कर प्रसंस्करण परिणाम पुश करना।

## विकल्प 2: नया HTTP पोर्ट जोड़ना

धीमे अनुरोधों को संसाधित करने के लिए नया HTTP पोर्ट जोड़ें। ये धीमे अनुरोध इस पोर्ट तक पहुंचकर एक विशिष्ट प्रक्रिया समूह द्वारा संसाधित होते हैं, प्रसंस्करण के बाद परिणाम सीधे क्लाइंट को वापस किए जाते हैं।

#### लाभ
डेटा सीधे क्लाइंट को वापस कर सकता है

#### नुकसान
अचानक बड़े पैमाने पर अनुरोधों का सामना नहीं कर सकता

#### क्रियान्वयन चरण
`config/process.php` में निम्नलिखित कॉन्फ़िगरेशन जोड़ें।
```php
return [
    // ... अन्य कॉन्फ़िगरेशन यहाँ छोड़े गए ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // प्रक्रियाओं की संख्या
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // अनुरोध वर्ग सेटिंग
            'logger' => \support\Log::channel('default'), // लॉगर इंस्टेंस
            'appPath' => app_path(), // app निर्देशिका स्थान
            'publicPath' => public_path() // public निर्देशिका स्थान
        ]
    ]
];
```

इस प्रकार धीमे इंटरफ़ेस `http://127.0.0.1:8686/` प्रक्रिया समूह के माध्यम से चल सकते हैं, अन्य प्रक्रियाओं के व्यवसायिक प्रसंस्करण को प्रभावित नहीं करते।

फ्रंटएंड को पोर्ट का अंतर महसूस न हो इसके लिए nginx में 8686 पोर्ट पर प्रॉक्सी जोड़ सकते हैं। यदि धीमे इंटरफ़ेस अनुरोध पथ सभी `/task` से शुरू होते हैं, तो nginx कॉन्फ़िगरेशन निम्नानुसार होगा:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# नया 8686 upstream जोड़ें
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /task से शुरू होने वाले अनुरोध 8686 पोर्ट पर जाएंगे, आवश्यकतानुसार /task बदलें
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # अन्य अनुरोध मूल 8787 पोर्ट पर जाएंगे
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

इस प्रकार जब क्लाइंट `domain.com/task/xxx` एक्सेस करेगा तो अलग 8686 पोर्ट द्वारा संसाधित होगा, 8787 पोर्ट के अनुरोध प्रसंस्करण को प्रभावित नहीं करेगा।

## विकल्प 3: HTTP Chunked का उपयोग कर अतुल्यकालिक खंडित डेटा भेजना

#### लाभ
डेटा सीधे क्लाइंट को वापस कर सकता है

**workerman/http-client इंस्टॉल करें**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // खाली chunk भेजकर प्रतिक्रिया समाप्ति दर्शाएं
        });
        // पहले HTTP हेडर भेजें, बाद का डेटा अतुल्यकालिक रूप से भेजा जाता है
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **नोट**
> यह उदाहरण `workerman/http-client` क्लाइंट का उपयोग कर HTTP परिणाम अतुल्यकालिक रूप से प्राप्त कर डेटा वापस करता है। [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html) जैसे अन्य अतुल्यकालिक क्लाइंट भी उपयोग किए जा सकते हैं।
