# WeChat SDK

## URL dự án

https://github.com/w7corp/easywechat
  
## Cài đặt
 
```php
composer require w7corp/easywechat
```
  
## Sử dụng

**config/wechat.php**

```php
<?php
return [
    /**
     * Thông tin cơ bản tài khoản. Lấy từ WeChat Official Platform / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - BẮT BUỘC điền trong chế độ tương thích và bảo mật!!!

    /**
     * Có sử dụng Stable Access Token hay không
     * Mặc định: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = sử dụng, false = không sử dụng
     */
    'use_stable_access_token' => false,

    /**
     * Cấu hình OAuth
     *
     * scopes: Nền tảng chính thức (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL callback sau khi ủy quyền OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Cấu hình yêu cầu HTTP (thời gian chờ, v.v.). Tham số có sẵn xem tại:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Chỉ dùng khi cần ghi đè URL mặc định (ví dụ từ nước ngoài). Cấu hình URI khác nhau cho từng module.

        'retry' => true, // Sử dụng cấu hình thử lại mặc định
        //  'retry' => [
        //      // Thử lại chỉ với các mã trạng thái sau
        //      'status_codes' => [429, 500]
        //       // Số lần thử lại tối đa
        //      'max_retries' => 3,
        //      // Khoảng cách giữa các yêu cầu (miligiây)
        //      'delay' => 1000,
        //      // Nếu thiết lập, thời gian chờ mỗi lần thử lại sẽ nhân với hệ số này
        //      // (vd. Lần 1: 1000ms; Lần 2: 3 * 1000ms; v.v.)
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

// URL callback sự kiện ủy quyền: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//Phải thay thế yêu cầu máy chủ
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Thêm thông tin

Truy cập https://easywechat.com/6.x/official-account/examples.html

