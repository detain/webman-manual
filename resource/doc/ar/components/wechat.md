# WeChat SDK

## رابط المشروع

https://github.com/w7corp/easywechat
  
## التثبيت
 
```php
composer require w7corp/easywechat
```
  
## الاستخدام

**config/wechat.php**

```php
<?php
return [
    /**
     * المعلومات الأساسية للحساب. احصل عليها من منصة وي تشات الرسمية / المنصة المفتوحة
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - يجب تعبئته في وضع التوافق والأمان!!!

    /**
     * استخدام Stable Access Token
     * الافتراضي: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = استخدام، false = عدم الاستخدام
     */
    'use_stable_access_token' => false,

    /**
     * إعدادات OAuth
     *
     * scopes: المنصة الرسمية (snsapi_userinfo / snsapi_base)، المنصة المفتوحة: snsapi_login
     * redirect_url: رابط استدعاء بعد تفويض OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * إعدادات طلبات HTTP (المهلة الزمنية، إلخ). المعلمات المتاحة:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // استخدم فقط عند الحاجة لتجاوز الرابط الافتراضي (مثلاً من الخارج). إعداد URIs مختلفة لكل وحدة.

        'retry' => true, // استخدام إعدادات إعادة المحاولة الافتراضية
        //  'retry' => [
        //      // إعادة المحاولة فقط لرموز الحالة التالية
        //      'status_codes' => [429, 500]
        //       // الحد الأقصى لعدد إعادة المحاولات
        //      'max_retries' => 3,
        //      // الفاصل الزمني بين الطلبات (ميلي ثانية)
        //      'delay' => 1000,
        //      // إذا تم التعيين، سيتم ضرب وقت انتظار كل إعادة محاولة بهذا العامل
        //      // (مثلاً. الأولى: 1000ms؛ الثانية: 3 * 1000ms؛ إلخ)
        //      'multiplier' => 3
        //  ],
    ],
];
```


**app\controller\OfficialAccount.php**

```php
<?php

namespace app\controller;

use EasyWeChat\OfficialAccount\Application;
use support\Request;
use Symfony\Component\HttpFoundation\HeaderBag;
use Symfony\Component\HttpFoundation\Request as SymfonyRequest;

// رابط استدعاء أحداث التفويض: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//يجب استبدال طلب الخادم
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## المزيد من المعلومات

قم بزيارة https://easywechat.com/6.x/official-account/examples.html

