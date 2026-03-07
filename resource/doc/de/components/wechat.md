# WeChat SDK

## Projekt-URL

https://github.com/w7corp/easywechat
  
## Installation
 
```php
composer require w7corp/easywechat
```
  
## Verwendung

**config/wechat.php**

```php
<?php
return [
    /**
     * Kontobasisdaten. Bitte von der WeChat-Offiziellplattform/Open-Platform beziehen
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - MUSS im Kompatibilitäts- und Sicherheitsmodus ausgefüllt werden!!!

    /**
     * Stable Access Token verwenden
     * Standard: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = verwenden, false = nicht verwenden
     */
    'use_stable_access_token' => false,

    /**
     * OAuth-Konfiguration
     *
     * scopes: Offiziellplattform (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: Callback-URL nach OAuth-Autorisierung
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * HTTP-Anfragekonfiguration (Timeout usw.). Verfügbare Parameter siehe:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Nur bei Bedarf, die Standard-URL zu überschreiben (z.B. aus dem Ausland). Verschiedene URIs für verschiedene Module.

        'retry' => true, // Standard-Wiederholungskonfiguration verwenden
        //  'retry' => [
        //      // Nur bei folgenden Statuscodes wiederholen
        //      'status_codes' => [429, 500]
        //       // Maximale Wiederholungsanzahl
        //      'max_retries' => 3,
        //      // Anfrageintervall (Millisekunden)
        //      'delay' => 1000,
        //      // Falls gesetzt, wird die Wartezeit bei jeder Wiederholung mit diesem Faktor multipliziert
        //      // (z.B. Erstes Mal: 1000ms; Zweites Mal: 3 * 1000ms; usw.)
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

// Callback-URL für Autorisierungsereignisse: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//Serveranfrage muss ersetzt werden
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Weitere Informationen

Besuchen Sie https://easywechat.com/6.x/official-account/examples.html

