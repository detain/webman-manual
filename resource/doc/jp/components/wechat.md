# WeChat SDK

## プロジェクトURL

https://github.com/w7corp/easywechat
  
## インストール
 
```php
composer require w7corp/easywechat
```
  
## 使用方法

**config/wechat.php**

```php
<?php
return [
    /**
     * アカウント基本情報。微信公衆プラットフォーム/オープンプラットフォームで取得してください
     */
    'app_id'  => 'your-app-id',         // AppID
    'secret'  => 'your-app-secret',     // AppSecret
    'token'   => 'your-token',          // Token
    'aes_key' => '',                    // EncodingAESKey、互換・セキュリティモードでは必ず入力してください！！！

    /**
     * Stable Access Tokenを使用するかどうか
     * デフォルト: false
     * https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/getStableAccessToken.html
     * true = 使用する、false = 使用しない
     */
    'use_stable_access_token' => false,

    /**
     * OAuth設定
     *
     * scopes: 公衆プラットフォーム（snsapi_userinfo / snsapi_base）、オープンプラットフォーム: snsapi_login
     * redirect_url: OAuth認証完了後のコールバックURL
     */
    'oauth' => [
        'scopes'   => ['snsapi_userinfo'],
        'redirect_url' => '/examples/oauth_callback.php',
    ],

    /**
     * インターフェースリクエスト設定（タイムアウト等）。利用可能なパラメータは以下を参照：
     * https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php
     */
    'http' => [
        'timeout' => 5.0,
        // 'base_uri' => 'https://api.weixin.qq.com/', // 海外からデフォルトのURLを上書きしたい場合のみ使用。モジュールごとに異なるuriを設定

        'retry' => true, // デフォルトのリトライ設定を使用
        //  'retry' => [
        //      // 以下のステータスコードの場合のみリトライ
        //      'status_codes' => [429, 500]
        //       // 最大リトライ回数
        //      'max_retries' => 3,
        //      // リクエスト間隔（ミリ秒）
        //      'delay' => 1000,
        //      // 設定した場合、各リトライの待機時間にこの係数を掛けます
        //      // （例：1回目: 1000ms; 2回目: 3 * 1000ms; など）
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

// 認証イベントコールバックURL: http://easywechat.com/OfficialAccount/server

class OfficialAccount
{
    public function server(Request $request)
    {
        $config = config('wechat');
        $app = new Application($config);
        $symfony_request = new SymfonyRequest($request->get(), $request->post(), [], $request->cookie(), [], [], $request->rawBody());
        $symfony_request->headers = new HeaderBag($request->header());
        $app->setRequestFromSymfonyRequest($symfony_request);//サーバーリクエストの置き換えが必須
        $server = $app->getServer();
        $response = $server->serve();

        return response($response->getBody()->getContents(), $response->getStatusCode(), $response->getHeaders());
    }
}
```
  
  
## 詳細情報

https://easywechat.com/6.x/official-account/examples.html をご覧ください

