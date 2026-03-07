# WeChat SDK

## প্রজেক্ট URL

https://github.com/w7corp/easywechat
  
## ইনস্টলেশন
 
```php
composer require w7corp/easywechat
```
  
## ব্যবহার

**config/wechat.php**

```php
<?php
return [
    /**
     * অ্যাকাউন্টের মৌলিক তথ্য। WeChat অফিসিয়াল প্ল্যাটফর্ম / ওপেন প্ল্যাটফর্ম থেকে সংগ্রহ করুন
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - কম্প্যাটিবিলিটি ও সিকিউরিটি মোডে অবশ্যই পূরণ করুন!!!

    /**
     * Stable Access Token ব্যবহার করবেন কিনা
     * ডিফল্ট: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = ব্যবহার করুন, false = ব্যবহার করবেন না
     */
    'use_stable_access_token' => false,

    /**
     * OAuth কনফিগারেশন
     *
     * scopes: অফিসিয়াল প্ল্যাটফর্ম (snsapi_userinfo / snsapi_base), ওপেন প্ল্যাটফর্ম: snsapi_login
     * redirect_url: OAuth অনুমোদনের পর কলব্যাক URL
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * HTTP অনুরোধ কনফিগারেশন (টাইমআউট ইত্যাদি)। উপলব্ধ প্যারামিটার দেখুন:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // ডিফল্ট URL ওভাররাইড করতে চাইলে (যেমন বিদেশ থেকে) শুধুমাত্র ব্যবহার করুন। বিভিন্ন মডিউলের জন্য বিভিন্ন URI কনফিগার করুন।

        'retry' => true, // ডিফল্ট রিট্রাই কনফিগারেশন ব্যবহার করুন
        //  'retry' => [
        //      // শুধুমাত্র নিম্নলিখিত স্ট্যাটাস কোডে রিট্রাই করুন
        //      'status_codes' => [429, 500]
        //       // সর্বোচ্চ রিট্রাই সংখ্যা
        //      'max_retries' => 3,
        //      // অনুরোধের ব্যবধান (মিলিসেকেন্ড)
        //      'delay' => 1000,
        //      // সেট করা থাকলে, প্রতিটি রিট্রাইয়ের অপেক্ষার সময় এই ফ্যাক্টর দ্বারা গুণিত হবে
        //      // (যেমন প্রথম: 1000ms; দ্বিতীয়: 3 * 1000ms; ইত্যাদি)
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

// অনুমোদন ইভেন্ট কলব্যাক URL: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//সার্ভার রিকোয়েস্ট অবশ্যই প্রতিস্থাপন করতে হবে
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## আরও তথ্য

দেখুন https://easywechat.com/6.x/official-account/examples.html

