# WeChat SDK

## URL del progetto

https://github.com/w7corp/easywechat
  
## Installazione
 
```php
composer require w7corp/easywechat
```
  
## Utilizzo

**config/wechat.php**

```php
<?php
return [
    /**
     * Informazioni base dell'account. Ottenere dalla Piattaforma Ufficiale WeChat / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - DEVE essere compilato in modalità compatibilità e sicurezza!!!

    /**
     * Se utilizzare Stable Access Token
     * Predefinito: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = utilizzare, false = non utilizzare
     */
    'use_stable_access_token' => false,

    /**
     * Configurazione OAuth
     *
     * scopes: Piattaforma Ufficiale (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL di callback dopo l'autorizzazione OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Configurazione richieste HTTP (timeout, ecc.). Parametri disponibili:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Usare solo per sovrascrivere l'URL predefinita (es. dall'estero). Configurare URI diversi per moduli diversi.

        'retry' => true, // Usare la configurazione di retry predefinita
        //  'retry' => [
        //      // Riprovare solo per i seguenti codici di stato
        //      'status_codes' => [429, 500]
        //       // Numero massimo di tentativi
        //      'max_retries' => 3,
        //      // Intervallo tra le richieste (millisecondi)
        //      'delay' => 1000,
        //      // Se impostato, il tempo di attesa di ogni retry sarà moltiplicato per questo fattore
        //      // (es. Primo: 1000ms; Secondo: 3 * 1000ms; ecc.)
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

// URL callback eventi autorizzazione: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//La richiesta del server deve essere sostituita
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Maggiori informazioni

Visita https://easywechat.com/6.x/official-account/examples.html

