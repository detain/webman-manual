# webman स्थिर फ़ाइल प्रसंस्करण

webman स्थिर फ़ाइल पहुंच का समर्थन करता है, स्थिर फ़ाइलें सभी `public` निर्देशिका में रखी जाती हैं। उदाहरण के लिए, `http://127.0.0.8787/upload/avatar.png` की पहुंच वास्तव में `{मुख्य परियोजना निर्देशिका}/public/upload/avatar.png` की पहुंच है।

> **ध्यान दें**
> `/app/xx/फ़ाइलनाम` से शुरू होने वाली स्थिर फ़ाइल पहुंच वास्तव में ऐप प्लगइन की `public` निर्देशिका तक पहुंच है। अर्थात्, `{मुख्य परियोजना निर्देशिका}/public/app/` के नीचे की निर्देशिकाओं तक पहुंच समर्थित नहीं है।
> अधिक जानकारी के लिए [ऐप प्लगइन](./plugin/app.md) देखें।

## स्थिर फ़ाइल समर्थन बंद करें

यदि स्थिर फ़ाइल समर्थन की आवश्यकता नहीं है, तो `config/static.php` खोलें और `enable` विकल्प को false पर बदलें। बंद करने के बाद सभी स्थिर फ़ाइल पहुंच 404 वापस करेगा।

## स्थिर फ़ाइल निर्देशिका बदलें

webman डिफ़ॉल्ट रूप से स्थिर फ़ाइल निर्देशिका के रूप में `public` निर्देशिका का उपयोग करता है। बदलने के लिए `support/helpers.php` में `public_path()` सहायक फ़ंक्शन संपादित करें।

## स्थिर फ़ाइल मध्यवर्ती

webman `app/middleware/StaticFile.php` में स्थित एक स्थिर फ़ाइल मध्यवर्ती के साथ आता है।
कभी-कभी स्थिर फ़ाइलों पर कुछ प्रक्रिया करने की आवश्यकता होती है, जैसे स्थिर फ़ाइल में क्रॉस-ऑरिजिन HTTP हेडर जोड़ना या बिंदु (`.`) से शुरू होने वाली फ़ाइलों तक पहुंच प्रतिबंधित करना—इस मध्यवर्ती का उपयोग किया जा सकता है।

`app/middleware/StaticFile.php` की सामग्री निम्नलिखित के समान है:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // बिंदु से शुरू होने वाली छिपी फ़ाइलों तक पहुंच प्रतिबंधित करें
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // क्रॉस-ऑरिजिन हेडर जोड़ें
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
इस मध्यवर्ती की आवश्यकता होने पर, `config/static.php` में `middleware` विकल्प सक्षम करें।
