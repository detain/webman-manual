# ধীর ব্যবসায়িক প্রক্রিয়াকরণ

কখনও কখনও আমাদের ধীর ব্যবসায়িক প্রক্রিয়া পরিচালনা করতে হয়। ধীর ব্যবসা webman-এর অন্যান্য অনুরোধ প্রক্রিয়াকরণকে প্রভাবিত না করার জন্য, পরিস্থিতি অনুযায়ী এই ব্যবসাগুলির জন্য বিভিন্ন প্রক্রিয়াকরণ সমাধান ব্যবহার করা যেতে পারে।

## সমাধান ১: মেসেজ কিউ ব্যবহার
[Redis কিউ](../queue/redis.md) [Stomp কিউ](../queue/stomp.md) দেখুন

#### সুবিধা
হঠাৎ বৃহৎ ব্যবসায়িক প্রক্রিয়াকরণ অনুরোধ মোকাবেলা করতে পারে

#### অসুবিধা
ক্লায়েন্টকে সরাসরি ফলাফল ফেরত দেওয়া যায় না। ফলাফল পুশ করতে হলে অন্যান্য সেবার সাথে সমন্বয় প্রয়োজন, যেমন [webman/push](https://www.workerman.net/plugin/2) ব্যবহার করে প্রক্রিয়াকরণ ফলাফল পুশ করা।

## সমাধান ২: নতুন HTTP পোর্ট যোগ করা

ধীর অনুরোধ প্রক্রিয়া করার জন্য নতুন HTTP পোর্ট যোগ করুন। এই ধীর অনুরোধগুলি এই পোর্টে অ্যাক্সেসের মাধ্যমে নির্দিষ্ট প্রক্রিয়া দলের দ্বারা প্রক্রিয়া করা হয় এবং প্রক্রিয়াকরণের পর ফলাফল সরাসরি ক্লায়েন্টকে ফেরত দেওয়া হয়।

#### সুবিধা
ডেটা সরাসরি ক্লায়েন্টকে ফেরত দিতে পারে

#### অসুবিধা
হঠাৎ বৃহৎ অনুরোধ মোকাবেলা করতে পারে না

#### বাস্তবায়নের ধাপ
`config/process.php`-এ নিচের কনফিগারেশন যোগ করুন।
```php
return [
    // ... অন্যান্য কনফিগারেশন এখানে বাদ দেওয়া হয়েছে ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // প্রক্রিয়া সংখ্যা
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // রিকোয়েস্ট ক্লাস সেটিং
            'logger' => \support\Log::channel('default'), // লগার ইনস্ট্যান্স
            'appPath' => app_path(), // app ডাইরেক্টরি অবস্থান
            'publicPath' => public_path() // public ডাইরেক্টরি অবস্থান
        ]
    ]
];
```

এভাবে ধীর ইন্টারফেসগুলি `http://127.0.0.1:8686/` প্রক্রিয়া দলের মাধ্যমে যেতে পারে, অন্যান্য প্রক্রিয়ার ব্যবসায়িক প্রক্রিয়াকরণকে প্রভাবিত না করে।

ফ্রন্টএন্ড পোর্টের পার্থক্য অনুভব না করতে পারে, nginx-এ 8686 পোর্টে একটি প্রক্সি যোগ করা যেতে পারে। ধীর ইন্টারফেস অনুরোধ পাথগুলি যদি `/task` দিয়ে শুরু হয়, পুরো nginx কনফিগারেশন নিম্নরূপ হবে:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# নতুন 8686 upstream যোগ করুন
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /task দিয়ে শুরু হওয়া অনুরোধগুলি 8686 পোর্টে যাবে, প্রয়োজন অনুযায়ী /task পরিবর্তন করুন
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # অন্যান্য অনুরোধ মূল 8787 পোর্টে যাবে
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

এভাবে ক্লায়েন্ট `domain.com/task/xxx` অ্যাক্সেস করলে আলাদা 8686 পোর্ট দিয়ে প্রক্রিয়া হবে, 8787 পোর্টের অনুরোধ প্রক্রিয়াকরণকে প্রভাবিত করবে না।

## সমাধান ৩: HTTP Chunked ব্যবহার করে অ্যাসিঙ্ক্রোনাস সেগমেন্টেড ডেটা পাঠানো

#### সুবিধা
ডেটা সরাসরি ক্লায়েন্টকে ফেরত দিতে পারে

**workerman/http-client ইনস্টল করুন**

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
            $connection->send(new Chunk('')); // খালি chunk পাঠিয়ে রেসপন্স শেষ নির্দেশ করুন
        });
        // প্রথমে HTTP হেডার পাঠান, পরের ডেটা অ্যাসিঙ্ক্রোনাসে পাঠানো হয়
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **নোট**
> এই উদাহরণে `workerman/http-client` ক্লায়েন্ট অ্যাসিঙ্ক্রোনাসে HTTP ফলাফল নিয়ে ডেটা ফেরত দেয়। [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html) এর মত অন্যান্য অ্যাসিঙ্ক্রোনাস ক্লায়েন্টও ব্যবহার করা যেতে পারে।
