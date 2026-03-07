# SDK WeChat

## URL do projeto

https://github.com/w7corp/easywechat
  
## Instalação
 
```php
composer require w7corp/easywechat
```
  
## Uso

**config/wechat.php**

```php
<?php
return [
    /**
     * Informações básicas da conta. Obtenha na Plataforma Oficial WeChat / Open Platform
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey - DEVE ser preenchido nos modos de compatibilidade e segurança!!!

    /**
     * Se utilizar Stable Access Token
     * Padrão: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = utilizar, false = não utilizar
     */
    'use_stable_access_token' => false,

    /**
     * Configuração OAuth
     *
     * scopes: Plataforma Oficial (snsapi_userinfo / snsapi_base), Open Platform: snsapi_login
     * redirect_url: URL de callback após autorização OAuth
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * Configuração de requisições HTTP (timeout, etc.). Parâmetros disponíveis em:
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // Use apenas quando precisar sobrescrever a URL padrão (ex. do exterior). Configure URIs diferentes para módulos diferentes.

        'retry' => true, // Usar configuração de nova tentativa padrão
        //  'retry' => [
        //      // Nova tentativa apenas para os seguintes códigos de status
        //      'status_codes' => [429, 500]
        //       // Número máximo de tentativas
        //      'max_retries' => 3,
        //      // Intervalo entre requisições (milissegundos)
        //      'delay' => 1000,
        //      // Se definido, o tempo de espera de cada nova tentativa será multiplicado por este fator
        //      // (ex. Primeira: 1000ms; Segunda: 3 * 1000ms; etc.)
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

// URL de callback para eventos de autorização: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//A requisição do servidor deve ser substituída
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## Mais informações

Acesse https://easywechat.com/6.x/official-account/examples.html

