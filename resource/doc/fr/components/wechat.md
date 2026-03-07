# SDK WeChat

## URL du projet

https://github.com/w7corp/easywechat
  
## Installation
 
```php
composer require w7corp/easywechat
```
  
## Utilisation

**config/wechat.php**

```php
<?php
return [
    /**
     * Informations de base du compte. À obtenir depuis la Plateforme Officielle WeChat / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - DOIT être renseigné en mode compatibilité et sécurité !!!

    /**
     * Utiliser le Stable Access Token
     * Par défaut : false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = utiliser, false = ne pas utiliser
     */
    'use_stable_access_token' => false,

    /**
     * Configuration OAuth
     *
     * scopes : Plateforme Officielle (snsapi_userinfo / snsapi_base), Open Platform : snsapi_login
     * redirect_url : URL de callback après l'autorisation OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Configuration des requêtes HTTP (timeout, etc.). Paramètres disponibles :
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // À utiliser uniquement pour remplacer l'URL par défaut (ex. depuis l'étranger). Configurer des URI différentes pour chaque module.

        'retry' => true, // Utiliser la configuration de nouvelle tentative par défaut
        //  'retry' => [
        //      // Nouvelle tentative uniquement pour les codes d'état suivants
        //      'status_codes' => [429, 500]
        //       // Nombre maximum de nouvelles tentatives
        //      'max_retries' => 3,
        //      // Intervalle entre les requêtes (millisecondes)
        //      'delay' => 1000,
        //      // Si défini, le temps d'attente de chaque nouvelle tentative sera multiplié par ce facteur
        //      // (ex. Premier : 1000ms ; Deuxième : 3 * 1000ms ; etc.)
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

// URL de callback pour les événements d'autorisation : http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//La requête serveur doit être remplacée
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Plus d'informations

Consultez https://easywechat.com/6.x/official-account/examples.html

