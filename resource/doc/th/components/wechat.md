# WeChat SDK

## URL โปรเจกต์

https://github.com/w7corp/easywechat
  
## การติดตั้ง
 
```php
composer require w7corp/easywechat
```
  
## การใช้งาน

**config/wechat.php**

```php
<?php
return [
    /**
     * ข้อมูลพื้นฐานบัญชี ดึงจาก WeChat Official Platform / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - ต้องกรอกในโหมดความเข้ากันได้และความปลอดภัย!!!

    /**
     * ใช้ Stable Access Token หรือไม่
     * ค่าเริ่มต้น: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = ใช้, false = ไม่ใช้
     */
    'use_stable_access_token' => false,

    /**
     * การตั้งค่า OAuth
     *
     * scopes: แพลตฟอร์มอย่างเป็นทางการ (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL สำหรับเรียกกลับหลังการอนุญาต OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * การตั้งค่าการร้องขอ HTTP (หมดเวลา ฯลฯ) ดูพารามิเตอร์ที่ใช้ได้ที่:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // ใช้เฉพาะเมื่อต้องการเขียนทับ URL เริ่มต้น (เช่น จากต่างประเทศ) ตั้งค่า URI ต่างกันสำหรับแต่ละโมดูล

        'retry' => true, // ใช้การตั้งค่าลองใหม่เริ่มต้น
        //  'retry' => [
        //      // ลองใหม่เฉพาะสำหรับรหัสสถานะต่อไปนี้
        //      'status_codes' => [429, 500]
        //       // จำนวนครั้งสูงสุดในการลองใหม่
        //      'max_retries' => 3,
        //      // ช่วงเวลาระหว่างการร้องขอ (มิลลิวินาที)
        //      'delay' => 1000,
        //      // ถ้ากำหนด เวลารอของแต่ละครั้งจะคูณด้วยตัวคูณนี้
        //      // (เช่น ครั้งแรก: 1000ms; ครั้งที่สอง: 3 * 1000ms; ฯลฯ)
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

// URL สำหรับเรียกกลับเหตุการณ์อนุญาต: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//ต้องแทนที่คำขอเซิร์ฟเวอร์
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## ข้อมูลเพิ่มเติม

เข้าชม https://easywechat.com/6.x/official-account/examples.html

