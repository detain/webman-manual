# কাস্টম প্রসেস

webman-এ workerman-এর মতো কাস্টম লিসেনার বা প্রসেস তৈরি করা যায়।

> **লক্ষ্য করুন**
> Windows ব্যবহারকারীদের কাস্টম প্রসেস চালু করতে `php windows.php` দিয়ে webman চালু করতে হবে।

## কাস্টম HTTP সেবা
কখনও কখনও বিশেষ প্রয়োজন দেখা দিতে পারে যখন webman HTTP সেবার মূল কোড পরিবর্তন করতে হয়। সেক্ষেত্রে কাস্টম প্রসেস ব্যবহার করা যায়।

উদাহরণস্বরূপ নতুন `app\Server.php` তৈরি করুন।

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // এখানে Webman\App-এর মেথডগুলো ওভাররাইট করুন
}
```

`config/process.php`-এ নিচের কনফিগারেশন যুক্ত করুন।

```php
use Workerman\Worker;

return [
    // ... অন্যান্য কনফিগারেশন বাদ ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // প্রসেসের সংখ্যা
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // রিকোয়েস্ট ক্লাস সেট করুন
            'logger' => \support\Log::channel('default'), // লগ ইন্সট্যান্স
            'appPath' => app_path(), // app ফোল্ডারের অবস্থান
            'publicPath' => public_path() // public ফোল্ডারের অবস্থান
        ]
    ]
];
```

> **পরামর্শ**
> webman-এর নিজস্ব HTTP প্রসেস বন্ধ করতে চাইলে config/server.php-এ `listen=>''` সেট করুন।

## কাস্টম WebSocket লিসেনার উদাহরণ

নতুন `app/Pusher.php` তৈরি করুন।

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> লক্ষ্য রাখুন: সব onXXX মেথড public হতে হবে।

`config/process.php`-এ নিচের কনফিগারেশন যুক্ত করুন।

```php
return [
    // ... অন্যান্য প্রসেস কনফিগারেশন বাদ ...
    
    // websocket_test প্রসেসের নাম
    'websocket_test' => [
        // এখানে প্রসেস ক্লাস নির্দিষ্ট করুন, উপরে সংজ্ঞায়িত Pusher ক্লাস
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## কাস্টম অ-লিসেনার প্রসেস উদাহরণ

নতুন `app/TaskTest.php` তৈরি করুন।

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // প্রতি ১০ সেকেন্ডে ডাটাবেসে নতুন নিবন্ধিত ব্যবহারকারী আছে কিনা যাচাই করুন
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

`config/process.php`-এ নিচের কনফিগারেশন যুক্ত করুন।

```php
return [
    // ... অন্যান্য প্রসেস কনফিগারেশন বাদ ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> লক্ষ্য রাখুন: listen বাদ দিলে কোনো পোর্ট শোনা হবে না, count বাদ দিলে প্রসেস সংখ্যা ডিফল্ট ১ হবে।

## কনফিগারেশন ফাইলের ব্যাখ্যা

একটি প্রসেসের পূর্ণাঙ্গ কনফিগারেশন নিচের মতো সংজ্ঞায়িত হয়:

```php
return [
    // ... 
    
    // websocket_test প্রসেসের নাম
    'websocket_test' => [
        // এখানে প্রসেস ক্লাস নির্দিষ্ট করুন
        'handler' => app\Pusher::class,
        // শোনার প্রোটোকল, আইপি ও পোর্ট (ঐচ্ছিক)
        'listen'  => 'websocket://0.0.0.0:8888',
        // প্রসেসের সংখ্যা (ঐচ্ছিক, ডিফল্ট ১)
        'count'   => 2,
        // প্রসেস চালানোর ব্যবহারকারী (ঐচ্ছিক, ডিফল্ট বর্তমান ব্যবহারকারী)
        'user'    => '',
        // প্রসেস চালানোর ব্যবহারকারী গ্রুপ (ঐচ্ছিক, ডিফল্ট বর্তমান গ্রুপ)
        'group'   => '',
        // বর্তমান প্রসেস reload সমর্থন করে কিনা (ঐচ্ছিক, ডিফল্ট true)
        'reloadable' => true,
        // reusePort চালু করা
        'reusePort'  => true,
        // transport (ঐচ্ছিক, SSL চাইলে ssl সেট করুন, ডিফল্ট tcp)
        'transport'  => 'tcp',
        // context (ঐচ্ছিক, transport ssl হলে সার্টিফিকেট পাথ দিতে হবে)
        'context'    => [], 
        // প্রসেস ক্লাস কনস্ট্রাক্টর প্যারামিটার (ঐচ্ছিক)
        'constructor' => [],
        // এই প্রসেস সক্রিয় কিনা
        'enable' => true
    ],
];
```

## সংক্ষিপ্তসার

webman-এর কাস্টম প্রসেস মূলত workerman-এর একটি সাধারণ মোড়ক। এটি কনফিগারেশন ও ব্যবসাকে আলাদা রাখে এবং workerman-এর `onXXX` কলব্যাকগুলো ক্লাস মেথড দিয়ে বাস্তবায়ন করে। অন্য ব্যবহার workerman-এর সঙ্গে সম্পূর্ণ একই।
