# WeChat SDK

## Proje URL'si

https://github.com/w7corp/easywechat
  
## Kurulum
 
```php
composer require w7corp/easywechat
```
  
## Kullanım

**config/wechat.php**

```php
<?php
return [
    /**
     * Hesap temel bilgileri. WeChat Resmi Platform / Open Platform'dan alın
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - Uyumluluk ve güvenlik modlarında mutlaka doldurulmalı!!!

    /**
     * Stable Access Token kullanılsın mı
     * Varsayılan: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = kullan, false = kullanma
     */
    'use_stable_access_token' => false,

    /**
     * OAuth yapılandırması
     *
     * scopes: Resmi Platform (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: OAuth yetkilendirme sonrası geri çağırma URL'si
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * HTTP isteği yapılandırması (zaman aşımı vb.). Kullanılabilir parametreler:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Yalnızca varsayılan URL'yi geçersiz kılmak istediğinizde (örn. yurtdışından) kullanın. Farklı modüller için farklı URI'ler yapılandırın.

        'retry' => true, // Varsayılan yeniden deneme yapılandırmasını kullan
        //  'retry' => [
        //      // Yalnızca aşağıdaki durum kodlarında yeniden dene
        //      'status_codes' => [429, 500]
        //       // Maksimum yeniden deneme sayısı
        //      'max_retries' => 3,
        //      // İstek aralığı (milisaniye)
        //      'delay' => 1000,
        //      // Ayarlandıysa, her yeniden denemenin bekleme süresi bu çarpanla çarpılır
        //      // (örn. İlk: 1000ms; İkinci: 3 * 1000ms; vb.)
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

// Yetkilendirme olayı geri çağırma URL'si: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//Sunucu isteği mutlaka değiştirilmelidir
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Daha fazla bilgi

https://easywechat.com/6.x/official-account/examples.html adresini ziyaret edin

