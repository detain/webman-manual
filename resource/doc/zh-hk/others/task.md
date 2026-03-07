# 慢業務處理

有時候我們需要處理慢業務，為了避免慢業務影響 webman 的其他請求處理，這些業務根據情況不同可以使用不同的處理方案。

## 方案一：使用訊息佇列
參考 [Redis 佇列](../queue/redis.md) [Stomp 佇列](../queue/stomp.md)

#### 優點
可以應對突發海量業務處理請求

#### 缺點
無法直接返回結果給客戶端。如需推送結果需要配合其他服務，例如使用 [webman/push](https://www.workerman.net/plugin/2) 推送處理結果。

## 方案二：新增 HTTP 端口

新增 HTTP 端口處理慢請求，這些慢請求透過訪問這個端口進入特定的一組進程處理，處理後將結果直接返回給客戶端。

#### 優點
可以直接將資料返回給客戶端

#### 缺點
無法應對突發的海量請求

#### 實施步驟
在 `config/process.php` 裡增加如下配置。
```php
return [
    // ... 這裡省略了其他配置 ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // 進程數
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // request 類設置
            'logger' => \support\Log::channel('default'), // 日誌實例
            'appPath' => app_path(), // app 目錄位置
            'publicPath' => public_path() // public 目錄位置
        ]
    ]
];
```

這樣慢介面可以走 `http://127.0.0.1:8686/` 這組進程，不影響其他進程的業務處理。

為了讓前端無感知端口的區別，可以在 nginx 加一個到 8686 端口的代理。假設慢介面請求路徑都是以 `/task` 開頭，整個 nginx 配置類似如下：
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# 新增一個 8686 upstream
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # 以 /task 開頭的請求走 8686 端口，請按實際情況將 /task 更改為你需要的前綴
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # 其他請求走原 8787 端口
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

這樣客戶端訪問 `網域.com/task/xxx` 時將會走單獨的 8686 端口處理，不影響 8787 端口的請求處理。

## 方案三：利用 HTTP Chunked 非同步分段發送資料

#### 優點
可以直接將資料返回給客戶端

**安裝 workerman/http-client**

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
            $connection->send(new Chunk('')); // 發送空的 chunk 代表 response 結束
        });
        // 先發送一個 HTTP 頭，後續資料透過非同步發送
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **提示**
> 本例中使用了 `workerman/http-client` 客戶端非同步取得 HTTP 結果並返回資料，也可以使用其他非同步客戶端例如 [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html)。
