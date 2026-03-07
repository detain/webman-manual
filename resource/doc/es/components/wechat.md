# SDK de WeChat

## URL del proyecto

https://github.com/w7corp/easywechat
  
## Instalación
 
```php
composer require w7corp/easywechat
```
  
## Uso

**config/wechat.php**

```php
<?php
return [
    /**
     * Información básica de la cuenta. Obtener desde la Plataforma Oficial de WeChat / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - DEBE rellenarse en los modos compatibilidad y seguridad!!!

    /**
     * Si usar Stable Access Token
     * Por defecto: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = usar, false = no usar
     */
    'use_stable_access_token' => false,

    /**
     * Configuración OAuth
     *
     * scopes: Plataforma Oficial (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL de callback tras la autorización OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Configuración de solicitudes HTTP (timeout, etc.). Parámetros disponibles en:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Solo usar cuando necesite sobrescribir la URL por defecto (ej. desde el extranjero). Configurar URIs diferentes para distintos módulos.

        'retry' => true, // Usar configuración de reintento por defecto
        //  'retry' => [
        //      // Reintentar solo con los siguientes códigos de estado
        //      'status_codes' => [429, 500]
        //       // Número máximo de reintentos
        //      'max_retries' => 3,
        //      // Intervalo entre solicitudes (milisegundos)
        //      'delay' => 1000,
        //      // Si se define, el tiempo de espera de cada reintento se multiplicará por este factor
        //      // (ej. Primero: 1000ms; Segundo: 3 * 1000ms; etc.)
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

// URL de callback para eventos de autorización: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//Debe reemplazar la solicitud del servidor
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Más información

Visite https://easywechat.com/6.x/official-account/examples.html

