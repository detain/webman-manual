# WeChat SDK

## URL проекта

https://github.com/w7corp/easywechat
  
## Установка
 
```php
composer require w7corp/easywechat
```
  
## Использование

**config/wechat.php**

```php
<?php
return [
    /**
     * Основная информация об аккаунте. Получите на Платформе WeChat / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - обязательно заполните в режимах совместимости и безопасности!!!

    /**
     * Использовать ли Stable Access Token
     * По умолчанию: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = использовать, false = не использовать
     */
    'use_stable_access_token' => false,

    /**
     * Настройка OAuth
     *
     * scopes: Официальная платформа (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL обратного вызова после OAuth-авторизации
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Настройка HTTP-запросов (таймаут и т.д.). Доступные параметры см.:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Используйте только при необходимости переопределить URL по умолчанию (например, из-за рубежа). Настройте разные URI для разных модулей.

        'retry' => true, // Использовать стандартную конфигурацию повторов
        //  'retry' => [
        //      // Повторять только при следующих кодах состояния
        //      'status_codes' => [429, 500]
        //       // Максимальное количество повторов
        //      'max_retries' => 3,
        //      // Интервал между запросами (миллисекунды)
        //      'delay' => 1000,
        //      // Если задано, время ожидания при каждом повторе будет умножено на этот коэффициент
        //      // (например: первый: 1000ms; второй: 3 * 1000ms; и т.д.)
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

// URL обратного вызова для событий авторизации: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//Необходимо заменить серверный запрос
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Дополнительная информация

Посетите https://easywechat.com/6.x/official-account/examples.html

