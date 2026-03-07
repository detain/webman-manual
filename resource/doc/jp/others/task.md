# 遅い業務処理

ときに、遅い業務を処理する必要があります。遅い業務が webman の他のリクエスト処理に影響しないようにするため、状況に応じて異なる処理方法を使用できます。

## 方法1：メッセージキューの使用
[Redisキュー](../queue/redis.md) [STOMPキュー](../queue/stomp.md)を参照してください

#### 利点
突発的な大量の業務処理リクエストに対応できます

#### 欠点
クライアントに直接結果を返すことはできません。結果をプッシュする必要がある場合は、[webman/push](https://www.workerman.net/plugin/2)で処理結果をプッシュするなど、他のサービスと連携する必要があります。

## 方法2：新しいHTTPポートの追加

新しいHTTPポートを追加して遅いリクエストを処理します。これらの遅いリクエストは、このポートにアクセスすることで特定のプロセスグループで処理され、処理後に結果が直接クライアントに返されます。

#### 利点
データを直接クライアントに返すことができます

#### 欠点
突発的な大量のリクエストに対応できません

#### 実施手順
`config/process.php` に以下の設定を追加します。
```php
return [
    // ... 他の設定は省略 ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // プロセス数
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // リクエストクラスの設定
            'logger' => \support\Log::channel('default'), // ログインスタンス
            'appPath' => app_path(), // appディレクトリの場所
            'publicPath' => public_path() // publicディレクトリの場所
        ]
    ]
];
```

これにより、遅いインターフェースは `http://127.0.0.1:8686/` のプロセスグループを通じて処理され、他のプロセスの業務処理に影響しません。

フロントエンドがポートの違いを認識しないようにするため、nginx に 8686 ポートへのプロキシを追加できます。遅いインターフェースのリクエストパスが `/task` で始まると仮定すると、nginx の設定は以下のようになります：
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# 新しい 8686 の upstream を追加
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /task で始まるリクエストは 8686 ポートへ、必要に応じて /task を必要なプレフィックスに変更してください
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # その他のリクエストは元の 8787 ポートへ
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

これにより、クライアントが `ドメイン.com/task/xxx` にアクセスすると、別の 8686 ポートで処理され、8787 ポートのリクエスト処理に影響しません。

## 方法3：HTTP Chunked による非同期セグメント送信

#### 利点
データを直接クライアントに返すことができます

**workerman/http-client のインストール**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // 空の chunk を送信してレスポンスの終了を示す
        });
        // 先に HTTP ヘッダーを送信し、その後のデータは非同期で送信
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **ヒント**
> 本例では `workerman/http-client` クライアントで HTTP 結果を非同期取得してデータを返しています。[AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html) などの他の非同期クライアントも使用できます。
