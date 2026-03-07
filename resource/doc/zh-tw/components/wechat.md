# 微信SDK

## 專案位址

https://github.com/w7corp/easywechat
  
## 安裝
 
```php
composer require w7corp/easywechat
```
  
## 使用

**config/wechat.php**

```php
<?php
return [
    /**
     * 帳號基本資訊，請從微信公眾平台/開放平台取得
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey，相容與安全模式下請務必填寫！！！

    /**
     * 是否使用 Stable Access Token
     * 預設 false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true 使用 false 不使用
     */
    'use_stable_access_token' => false,

    /**
     * OAuth 設定
     *
     * scopes：公眾平台（snsapi_userinfo / snsapi_base），開放平台：snsapi_login
     * redirect_url：OAuth 授權完成後的回調頁面位址
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * 介面請求相關設定，逾時時間等，具體可用參數請參考：
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // 若您在國外想要覆蓋預設的 url 時才使用，可根據不同模組配置不同的 uri

        'retry' => true, // 使用預設重試設定
        //  'retry' => [
        //      // 僅以下狀態碼重試
        //      'status_codes' => [429, 500]
        //       // 最大重試次數
        //      'max_retries' => 3,
        //      // 請求間隔（毫秒）
        //      'delay' => 1000,
        //      // 若設定，每次重試的等待時間都會乘以此係數
        //      //（例如：首次 1000ms；第二次：3 * 1000ms；以此類推）
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

// 授權事件回調位址：http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//必須替換服務端請求
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## 更多內容

造訪 https://easywechat.com/6.x/official-account/examples.html

