# WeChat SDK

## 프로젝트 URL

https://github.com/w7corp/easywechat
  
## 설치
 
```php
composer require w7corp/easywechat
```
  
## 사용

**config/wechat.php**

```php
<?php
return [
    /**
     * 계정 기본 정보. 위챗 공식 플랫폼/오픈 플랫폼에서 획득하세요
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey, 호환 및 보안 모드에서는 반드시 입력하세요!!!

    /**
     * Stable Access Token 사용 여부
     * 기본값: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = 사용, false = 미사용
     */
    'use_stable_access_token' => false,

    /**
     * OAuth 설정
     *
     * scopes: 공식 플랫폼(snsapi_userinfo / snsapi_base), 오픈 플랫폼: snsapi_login
     * redirect_url: OAuth 인증 완료 후 콜백 URL
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * API 요청 관련 설정(타임아웃 등). 사용 가능한 매개변수 참고:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // 해외에서 기본 URL을 덮어쓰고 싶을 때만 사용. 모듈별로 다른 uri 설정

        'retry' => true, // 기본 재시도 설정 사용
        //  'retry' => [
        //      // 다음 상태 코드에서만 재시도
        //      'status_codes' => [429, 500]
        //       // 최대 재시도 횟수
        //      'max_retries' => 3,
        //      // 요청 간격 (밀리초)
        //      'delay' => 1000,
        //      // 설정 시, 각 재시도의 대기 시간에 이 계수를 곱함
        //      // (예: 첫 번째: 1000ms; 두 번째: 3 * 1000ms; 등)
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

// 인증 이벤트 콜백 URL: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//서버 요청 교체 필수
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## 더 많은 정보

https://easywechat.com/6.x/official-account/examples.html 방문

