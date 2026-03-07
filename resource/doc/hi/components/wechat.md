# WeChat SDK

## परियोजना URL

https://github.com/w7corp/easywechat
  
## स्थापना
 
```php
composer require w7corp/easywechat
```
  
## उपयोग

**config/wechat.php**

```php
<?php
return [
    /**
     * खाता मूल जानकारी। WeChat आधिकारिक प्लेटफॉर्म / ओपन प्लेटफॉर्म से प्राप्त करें
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - अनुकूलता व सुरक्षा मोड में अवश्य भरें!!!

    /**
     * Stable Access Token उपयोग करना है या नहीं
     * डिफ़ॉल्ट: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = उपयोग करें, false = उपयोग न करें
     */
    'use_stable_access_token' => false,

    /**
     * OAuth कॉन्फ़िगरेशन
     *
     * scopes: आधिकारिक प्लेटफॉर्म (snsapi_userinfo / snsapi_base), ओपन प्लेटफॉर्म: snsapi_login
     * redirect_url: OAuth प्राधिकरण के बाद कॉलबैक URL
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * HTTP अनुरोध कॉन्फ़िगरेशन (टाइमआउट आदि)। उपलब्ध पैरामीटर देखें:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // केवल तब उपयोग करें जब डिफ़ॉल्ट URL ओवरराइड करना हो (जैसे विदेश से)। विभिन्न मॉड्यूल के लिए अलग-अलग URI कॉन्फ़िगर करें।

        'retry' => true, // डिफ़ॉल्ट पुनःप्रयास कॉन्फ़िगरेशन उपयोग करें
        //  'retry' => [
        //      // निम्नलिखित स्टेटस कोड पर ही पुनःप्रयास करें
        //      'status_codes' => [429, 500]
        //       // अधिकतम पुनःप्रयास संख्या
        //      'max_retries' => 3,
        //      // अनुरोध अंतराल (मिलीसेकंड)
        //      'delay' => 1000,
        //      // यदि सेट किया जाए, तो प्रत्येक पुनःप्रयास का प्रतीक्षा समय इस गुणक से गुणा होगा
        //      // (जैसे पहला: 1000ms; दूसरा: 3 * 1000ms; आदि)
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

// प्राधिकरण इवेंट कॉलबैक URL: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//सर्वर अनुरोध प्रतिस्थापित करना अनिवार्य है
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## अधिक जानकारी

https://easywechat.com/6.x/official-account/examples.html पर जाएँ

