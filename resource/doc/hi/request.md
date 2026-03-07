# विवरण

## अनुरोध ऑब्जेक्ट प्राप्त करें
webman स्वत: ही कार्रवाई विधि के पहले पैरामीटर में अनुरोध ऑब्जेक्ट को स्वचालित रूप से इंजेक्ट करेगा, जैसे

**उदाहरण**
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        // get अनुरोध से नाम पैरामीटर प्राप्त करें, अगर नाम पैरामीटर पास नहीं किया गया है तो $default_name वापस दें
        $name = $request->get('name', $default_name);
        // ब्राउज़र को स्ट्रिंग वापस भेजें
        return response('hello ' . $name);
    }
}
```

`$request` ऑब्जेक्ट के माध्यम से हम किसी भी अनुरोध संबंधित डेटा को प्राप्त कर सकते हैं।

**कभी-कभी हमें अनुरोध के `$request` ऑब्जेक्ट को अनुरोधित होने के दौरान अन्य कक्षा में प्राप्त करना चाहिए, इस समय हमें केवल हेल्पर फंक्शन `request()` का उपयोग करना होगा**;

## अनुरोध पैरामीटर get प्राप्त  करें

**संपूर्ण get एरे प्राप्त करें**
```php
$request->get();
```
यदि अनुरोध में get पैरामीटर नहीं है तो एक खाली एरे देगा।

**get एरे का कोई विशेष मान प्राप्त  करें**
```php
$request->get('name');
```
अगर जो get एरे में यह मान शामिल नहीं है तो null लौटाएगा।

आप चाहें तो get विधि को दूसरी पैरामीटर के रूप में एक डिफ़ॉल्ट मान भी पास कर सकते हैं, अगर get एरे में विशेष मान नहीं मिला तो वह डिफ़ॉल्ट मान देगा। उदाहरण:
```php
$request->get('name', 'tom');
```

## अनुरोध पैरामीटर post प्राप्त  करें
**संपूर्ण post एरे प्राप्त करें**
```php
$request->post();
```
यदि अनुरोध में post पैरामीटर नहीं है तो एक खाली एरे देगा।

**पोस्ट एरे का कोई विशेष मान प्राप्त करें**
```php
$request->post('name');
```
अगर जो पोस्ट एरे में यह मान शामिल नहीं है तो null लौटाएगा।

get विधि की तरह, आप चाहें तो पोस्ट विधि को दूसरी पैरामीटर के रूप में एक डिफ़ॉल्ट मान भी पास कर सकते हैं, अगर पोस्ट एरे में विशेष मान नहीं मिला तो वह डिफ़ॉल्ट मान देगा। उदाहरण:
```php
$request->post('name', 'tom');
```

## हेल्पर फंक्शन input()
`$request->input()` के समान, `input()` हेल्पर फंक्शन सभी पैरामीटर प्राप्त कर सकता है। इसमें दो पैरामीटर होते हैं:
1. name: प्राप्त करने वाले पैरामीटर का नाम (यदि खाली है, तो सभी पैरामीटर की सरणी प्राप्त करता है)
2. default: डिफ़ॉल्ट मान (जब पहले तर्क से पैरामीटर नहीं मिलता है तो उपयोग किया जाता है)

**उदाहरण**
```php
// पैरामीटर name प्राप्त करें
$name = input('name');
// पैरामीटर name प्राप्त करें, यदि मौजूद नहीं है तो डिफ़ॉल्ट मान का उपयोग करें
$name = input('name', 'राज');
// सभी पैरामीटर प्राप्त करें
$all_params = input();
```

## मौलिक अनुरोध पोस्ट बॉडी प्राप्त  करें
```php
$post = $request->rawBody();
```
इस कार्य का php-fpm में `file_get_contents("php://input");` के ऑपरेशन के समान है। यह अनुरोध के मौलिक पोस्ट बॉडी को प्राप्त करने के लिए उपयोगी है। यह उपयोगी होता है गैर `application/x-www-form-urlencoded` प्रारूप के पोस्ट अनुरोध डेटा को प्राप्त करने में।


## हेडर प्राप्त  करें
**संपूर्ण हेडर एरे प्राप्त करें**
```php
$request->header();
```
अगर अनुरोध में हेडर पैरामीटर नहीं है तो एक खाली एरे देगा। सभी कुंजी छोटे लिखे गए होंगे।

**हेडर एरे का कोई विशेष मान प्राप्त  करें**
```php
$request->header('host');
```
अगर हेडर एरे में इस मान को नहीं मिला तो null देगा। सभी कुंजी छोटे लिखे गए होंगे।

get विधि की तरह, आप चाहें तो हेडर विधि को दूसरी पैरामीटर के रूप में एक डिफ़ॉल्ट मान भी पास कर सकते हैं, अगर हेडर एरे में विशेष मान नहीं मिला तो वह डिफ़ॉल्ट मान देगा। उदाहरण:
```php
$request->header('host', 'localhost');
```

## कुकी प्राप्त  करें
**संपूर्ण कुकी एरे प्राप्त करें**
```php
$request->cookie();
```
यदि अनुरोध में कुकी पैरामीटर नहीं है तो एक खाली एरे देगा।

**कुकी एरे का कोई विशेष मान प्राप्त  करें**
```php
$request->cookie('name');
```
अगर कुकी एरे में इस मान को नहीं मिला तो null देगा।

get विधि की तरह, आप चाहें तो कुकी विधि को दूसरी पैरामीटर के रूप में एक डिफ़ॉल्ट मान भी पास कर सकते हैं, अगर कुकी एरे में विशेष मान नहीं मिला तो वह डिफ़ॉल्ट मान देगा। उदाहरण:
```php
$request->cookie('name', 'tom');
```

## सभी इनपुट प्राप्त  करें
`post` `get` को शामिल करता है।
```php
$request->all();
```

## निर्दिष्ट इनपुट मान प्राप्त  करें
`post` `get` संग्रह से कोई मान प्राप्त  करें।
```php
$request->input('name', $default_value);
```

## कुछ इनपुट डेटा प्राप्त करें
`post` `get` संग्रह से कुछ डेटा प्राप्त करें।
```php
// उपयोगकर्ता नाम और पासवर्ड से बना संग्रह प्राप्त करें, यदि संबंधित कुंजी नहीं मिली है तो उपेक्षा करें
$only = $request->only(['username', 'password']);
// अवतार और उम्र को छोड़कर सभी इनपुट प्राप्त करें
$except = $request->except(['avatar', 'age']);
```

## कंट्रोलर पैरामीटर के माध्यम से इनपुट प्राप्त करें

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```
उपरोक्त लॉजिक इसके समतुल्य है:
```php
<?php
namespace app\controller;
use support\Request;
use support\Response;

class UserController
{
    public function create(Request $request): Response
    {
        $name = $request->input('name');
        $age = (int)$request->input('age', 18);
        return json(['name' => $name, 'age' => $age]);
    }
}
```
अधिक जानकारी के लिए [कंट्रोलर पैरामीटर बाइंडिंग](controller.md#कंट्रोलर-पैरामीटर-बाइंडिंग) देखें।

## फाइल अपलोड प्राप्त करें

> **नोट**
> फाइल अपलोड के लिए `multipart/form-data` प्रारूप के फॉर्म की आवश्यकता होती है।

**पूरी अपलोड फ़ाइल सरणी प्राप्त करें**
```php
$request->file();
```

फ़ॉर्म की तरह:
```html
<form method="post" action="http://127.0.0.1:8787/upload/files" enctype="multipart/form-data">
<input name="file1" multiple="multiple" type="file">
<input name="file2" multiple="multiple" type="file">
<input type="submit">
</form>
```

`$request->file()` का प्रारूप निम्नलिखित होता है:
```php
array (
    'file1' => object(webman\Http\UploadFile),
    'file2' => object(webman\Http\UploadFile)
)
```
यह `webman\Http\UploadFile` इंस्टेंस का एक सरणी है। `webman\Http\UploadFile` वर्करमैन का एक्सटेंशन `SplFileInfo` क्लास की संगत करती है और कुछ उपयोगी विधियों को प्रदान करती है।

```php
<?php
namespace app\controller;

use support\Request;

class UploadController
{
    public function files(Request $request)
    {
        foreach ($request->file() as $key => $spl_file) {
            var_export($spl_file->isValid()); // फ़ाइल क्या वैध है, जैसे true|false
            var_export($spl_file->getUploadExtension()); // अपलोड फ़ाइल एक्सटेंशन, जैसे 'jpg'
            var_export($spl_file->getUploadMimeType()); // अपलोड फ़ाइल MIME प्रकार, जैसे 'image/jpeg'
            var_export($spl_file->getUploadErrorCode()); // अपलोड त्रुटि कोड प्राप्त करें, जैसे UPLOAD_ERR_NO_TMP_DIR, UPLOAD_ERR_NO_FILE, UPLOAD_ERR_CANT_WRITE
            var_export($spl_file->getUploadName()); // अपलोड फ़ाइल नाम, जैसे 'my-test.jpg'
            var_export($spl_file->getSize()); // फ़ाइल का आकार प्राप्त करें, जैसे 13364, इकाई बाइट में
            var_export($spl_file->getPath()); // अपलोड की गई निर्देशिका प्राप्त करें, जैसे '/tmp'
            var_export($spl_file->getRealPath()); // अस्थायी फ़ाइल पथ प्राप्त करें, जैसे `/tmp/workerman.upload.SRliMu`
        }
        return response('ok');
    }
}
```

**ध्यान दें:**

- फ़ाइल अपलोड के बाद वह एक अस्थायी फ़ाइल के रूप में नामित किया जाएगा, जैसे `/tmp/workerman.upload.SRliMu`
- अपलोड फ़ाइल का आकार [defaultMaxPackageSize](http://doc.workerman.net/tcp-connection/default-max-package-size.html) की सीमा में रहेगा, डिफ़ॉल्ट 10M, 'config/server.php' फ़ाइल में 'max_package_size' को बदलकर डिफ़ॉल्ट मान को बदल सकते हैं।
- अनुरोध समाप्त होने के बाद अस्थायी फ़ाइल स्वचालित रूप से हटा दी जाएगी
- अगर अनुरोध में कोई फ़ाइल अपलोड नहीं होती है तो `$request->file()` एक खाली सरणी लौटाता है
- अपलोड की गई फ़ाइल `move_uploaded_file()` विधि का समर्थन नहीं करती है, कृपया `$file->move()` विधि का प्रयोग करें, नीचे दी गई उदाहरण देखें।

### विशेष फ़ाइल प्राप्त करें
```php
$request->file('avatar');
```
यदि फ़ाइल मौजूद हो तो यह अनुसार फ़ाइल की `webman\Http\UploadFile` इंस्टेंस लौटाता है, अन्यथा `null` लौटाता है।

**उदाहरण**
```php
<?php
namespace app\controller;

use support\Request;

class UploadController
{
    public function file(Request $request)
    {
        $file = $request->file('avatar');
        if ($file && $file->isValid()) {
            $file->move(public_path().'/files/myfile.'.$file->getUploadExtension());
            return json(['code' => 0, 'msg' => 'upload success']);
        }
        return json(['code' => 1, 'msg' => 'file not found']);
    }
}
```

## होस्ट प्राप्त करें
अनुरोध के होस्ट जानकारी प्राप्त करें।
```php
$request->host();
```
यदि अनुरोध का पता मानक 80 या 443 पोर्ट नहीं है, तो होस्ट जानकारी में पोर्ट शामिल हो सकता है, जैसे `example.com:8080`। पोर्ट की आवश्यकता न हो तो पहले पैरामीटर `true` को पास किया जा सकता है।

```php
$request->host(true);
```

## अनुरोध पद्धति प्राप्त करें
```php
 $request->method();
```
`GET`、`POST` ।`PUT` ।`DELETE`, `OPTIONS`, `HEAD` में से एक लौटाता है।

## अनुरोध यूआरआई प्राप्त करें

```php
$request->uri();
```
पथ और क्वेरी स्ट्रिंग सहित अनुरोध का URI लौटाता है।

## अनुरोध पथ प्राप्त करें

```php
$request->path();
```
अनुरोध का पथ भाग लौटाता है।

## अनुरोध क्वेरी स्ट्रिंग प्राप्त करें

```php
$request->queryString();
```
अनुरोध का क्वेरी स्ट्रिंग भाग लौटाता है।

## अनुरोध URL प्राप्त करें
`url()` विधि `Query` पैरामीटर के साथ URL लौटाती है नहीं करती है।
```php
$request->url();
```
यह `//www.workerman.net/workerman-chat` जैसा आउटपुट देगा।

`fullUrl()` विधि `Query` पैरामीटर के साथ URL लौटाती है।
```php
$request->fullUrl();
```
यह `//www.workerman.net/workerman-chat?type=download` जैसा आउटपुट देगा।

> **नोट:**
> `url()` और `fullUrl()` में प्रोटोकॉल भाग नहीं होते (http या https नहीं होता है)।
> क्योंकि ब्राउज़र में उपयोग के लिए `//example.com` इस तरह से शुरू होने वाले पते को स्वचालित रूप से वर्तमान स्थान की प्रोटोकॉल की पहचान करते हैं, स्वत: http या https से नियुक्त होकर अनुरोध भेजता है।

यदि आप एनजिंक्स प्रॉक्सी का उपयोग कर रहे हैं, तो कृपया एनजिंक्स कॉन्फ़िगरेशन में `proxy_set_header X-Forwarded-Proto $scheme;` जोड़ें, [एनजिंक्स प्रॉक्सी से संबंधित](others/nginx-proxy.md)।
ऐसा करने से आप ` $request->header('x-forwarded-proto');` का उपयोग करके http या https को पहचान सकते हैं, जैसे:
```php
echo $request->header('x-forwarded-proto'); // प्रिंट करें http या https
```

## अनुरोध HTTP संस्करण प्राप्त करें

```php
$request->protocolVersion();
```
स्ट्रिंग `1.1` या `1.0` को लौटाता है।

## अनुरोध सत्राधीश प्राप्त करें

```php
$request->sessionId();
```
अक्षरों और संख्याओं से बना स्ट्रिंग लौटाता है।
## प्राप्त करें अनुरोध क्लाइंट IP
```php
$request->getRemoteIp();
```

## प्राप्त करें अनुरोध क्लाइंट पोर्ट
```php
$request->getRemotePort();
```

## प्राप्त करें अनुरोध क्लाइंट वास्तविक IP
```php
$request->getRealIp($safe_mode=true);
```

परियोजना ने प्रॉक्सी (जैसे nginx) का उपयोग किया है तो ` $request->getRemoteIp() ` का उपयोग करके प्राप्त किया गया सामान्यतः प्रॉक्सी सर्वर IP (जैसे `127.0.0.1` `192.168.x.x`) वास्तविक ग्राहक IP नहीं होता है। इस समय आप ` $request->getRealIp() ` का उपयोग करके ग्राहक का वास्तविक IP प्राप्त करने के लिए कोशिश कर सकते हैं।

`$request->getRealIp()` वास्तविक IP प्राप्त करने के लिए HTTP हेडर के `x-forwarded-for`, `x-real-ip`, `client-ip`, `x-client-ip`, `via` फ़ील्ड से प्रयास करेगा।

> HTTP हेडर आसानी से जाली बनाया जा सकता है, इसलिए इस विधि से प्राप्त किए गए ग्राहक IP निश्चित रूप से 100% विश्वसनीय नहीं है, विशेष रूप से जब $safe_mode फॉल्स होता है। प्रॉक्सी के माध्यम से ग्राहक का वास्तविक IP प्राप्त करने का तुलनात्मक विश्वसनीय उपाय है, यहां तक कि यदि कोई जानकार तानिक प्रॉक्सी सर्वरों के IP के बारे में है और स्पष्ट रूप से पता है कि वास्तविक IP कोनसा HTTP हेडर ले जाता है। यदि $request->getRemoteIp() द्वारा वापस लौटाया गया IP निश्चित रूप से जानी पहचानी सुरक्षित प्रॉक्सी सर्वर की पुष्टि होती है, और फिर $request->header('वास्तविक IP लेने वाला HTTP हेडर') के माध्यम से वास्तविक IP प्राप्त कर सकते हैं।


## सर्वर IP प्राप्त करें
```php
$request->getLocalIp();
```

## सर्वर पोर्ट प्राप्त करें
```php
$request->getLocalPort();
```

## क्या यह एजेक्स अनुरोध है, इसे निर्धारित करें
```php
$request->isAjax();
```

## क्या यह पीजैक्स अनुरोध है, इसे निर्धारित करें
```php
$request->isPjax();
```

## क्या यह जेसन को वापस देना चाहता है, इसे निर्धारित करें
```php
$request->expectsJson();
```

## क्या ग्राहक जेसन को स्वीकार करता है, इसे निर्धारित करें
```php
$request->acceptJson();
```

## अनुरोध का प्लगइन नाम प्राप्त करें
प्लगइन के अलावा अनुरोध खाली स्ट्रिंग `''` लौटाते हैं।
```php
$request->plugin;
```

## अनुरोध का ऐप का नाम प्राप्त करें
एकल ऐप के समय हमेशा खाली स्ट्रिंग `''` लौटाएगा, [बहु-ऐप्लिकेशन](multiapp.md) के समय एप्लिकेशन का नाम लौटाएगा
```php
$request->app;
```

> क्योंकि बंद फ़ंक्शन किसी भी एप्लिकेशन का हिस्सा नहीं है, इसलिए बंद फ़ंक्शन से आने वाले अनुरोध $request->app हमेशा खाली स्ट्रिंग `''` लौटाता है
> बंद फ़ंक्शन देखें [रूट](route.md)

## अनुरोध का नियंत्रक का नाम प्राप्त करें
नियंत्रक के लिए संबंधित क्लास का नाम प्राप्त करें
```php
$request->controller;
```
वापस ब्लॉक की तरह `app\controller\IndexController` लौटेगा

> क्योंकि बंद फ़ंक्शन किसी भी नियंत्रक का हिस्सा नहीं है, इसलिए बंद फ़ंक्शन से आने वाले अनुरोध $request->controller हमेशा खाली स्ट्रिंग `''` लौटाता है
> बंद फ़ंक्शन देखें [रूट](route.md)


## अनुरोध का विधि का नाम प्राप्त करें
नियंत्रक की विधि का नाम प्राप्त करें
```php
$request->action;
```
वापस `index` जैसा लौटेगा

> क्योंकि बंद फ़ंक्शन किसी भी नियंत्रक का हिस्सा नहीं है, इसलिए बंद फ़ंक्शन से आने वाले अनुरोध $request->action हमेशा खाली स्ट्रिंग `''` लौटाता है
> बंद फ़ंक्शन देखें [रूट](route.md)

## पैरामीटर अधिलेखित करना

कभी-कभी हम अनुरोध पैरामीटर को अधिलेखित करना चाहते हैं, उदाहरण के लिए अनुरोध को फ़िल्टर करके अनुरोध वस्तु को फिर से असाइन करना। ऐसे मामलों में हम `setGet()`, `setPost()` और `setHeader()` विधियों का उपयोग कर सकते हैं।

#### GET पैरामीटर अधिलेखित करना
```php
$request->get(); // मान लें परिणाम ['name' => 'tom', 'age' => 18] है
$request->setGet(['name' => 'tom']);
$request->get(); // अंतिम परिणाम ['name' => 'tom'] है
```

> **नोट**
> उदाहरण के अनुसार, `setGet()` सभी GET पैरामीटर को अधिलेखित करता है। `setPost()` और `setHeader()` भी इसी प्रकार काम करते हैं।

#### POST पैरामीटर अधिलेखित करना
```php
$post = $request->post();
foreach ($post as $key => $value) {
    $post[$key] = htmlspecialchars($value);
}
$request->setPost($post);
$request->post(); // फ़िल्टर किए गए post पैरामीटर प्राप्त करें
```

#### HEADER पैरामीटर अधिलेखित करना
```php
$request->setHeader(['host' => 'example.com']);
$request->header('host'); // आउटपुट: example.com
```
