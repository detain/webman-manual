# সাক্ষাৎকার
স্বাগতম, আমি আপনার জন্য কীভাবে সাহায্য করতে পারি? আপনি যদি অনুবাদ সম্পর্কে জিজ্ঞাসা থাকে তাহলে আমি আপনাকে কীভাবে সাহায্য করতে পারি তা জানাতে পারি।
## আউটপুট পেতে
কিছু লাইব্রেরি ফাইলের বিষয়বস্তুকে সরাসরি স্ট্যান্ডার্ড আউটপুটে প্রিন্ট করে, অর্থাৎ ডেটা টি টার্মিনালে প্রিন্ট করা হয়, যা ব্রাউজারে প্রেরণ করা হয় না, এই সময়ে আমরা `ob_start();` এবং `ob_get_clean();` এর মাধ্যমে ডেটা কে একটি ভেরিয়েবলে আবৃত্ত করতে হবে, তারপর আবার ডেটা টি ব্রাউজারে প্রেরণ করতে হবে, উদাহরণস্বরূপ:

```php
<?php

namespace app\controller;

use support\Request;

class ImageController
{
    public function get(Request $request)
    {
        // চিত্র তৈরি
        $im = imagecreatetruecolor(120, 20);
        $text_color = imagecolorallocate($im, 233, 14, 91);
        imagestring($im, 1, 5, 5,  'A Simple Text String', $text_color);

        // আউটপুট পেতে শুরু করুন
        ob_start();
        // চিত্র আউটপুট
        imagejpeg($im);
        // চিত্রের সামগ্রী পেতে
        $image = ob_get_clean();
        
        // চিত্র প্রেরণ করুন
        return response($image)->header('Content-Type', 'image/jpeg');
    }
}
```

## খণ্ডিত প্রতিক্রিয়া

কখনও কখনও আমরা খণ্ডে খণ্ডে প্রতিক্রিয়া পাঠাতে চাই। আপনি নীচের উদাহরণটি দেখতে পারেন।

```php
<?php

namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;
use Workerman\Timer;

class IndexController
{
    public function index(Request $request): Response
    {
        // সংযোগ পান
        $connection = $request->connection;
        // পর্যায়ক্রমে HTTP বডি পাঠান
        $timer = Timer::add(1, function () use ($connection, &$timer) {
            static $i = 0;
            if ($i++ < 10) {
                // HTTP বডি পাঠান
                $connection->send(new Chunk($i));
            } else {
                // মেমোরি লিক প্রতিরোধ করতে ব্যবহৃত না হওয়া টাইমার মুছুন
                Timer::del($timer);
                // ক্লায়েন্টকে প্রতিক্রিয়া সম্পূর্ণ হয়েছে জানাতে খালি Chunk পাঠান
                $connection->send(new Chunk(''));
            }
        });
        // প্রথমে Transfer-Encoding: chunked সহ HTTP হেডার আউটপুট করুন, তারপর HTTP বডি অ্যাসিঙ্ক্রোনাসভাবে পাঠান
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }

}
```

আপনি যদি বড় ভাষা মডেল কল করেন, নীচের উদাহরণটি দেখুন।

```
composer require webman/openai
```

```php
<?php
namespace app\controller;
use support\Request;

use Webman\Openai\Chat;
use Workerman\Protocols\Http\Chunk;

class ChatController
{
    public function completions(Request $request)
    {
        $connection = $request->connection;
        // যদি https://api.openai.com আপনার অঞ্চলে অ্যাক্সেসযোগ্য না হয়, তাহলে https://api.openai-proxy.com ব্যবহার করতে পারেন
        $chat = new Chat(['apikey' => 'sk-xx', 'api' => 'https://api.openai.com']);
        $chat->completions(
            [
                'model' => 'gpt-3.5-turbo',
                'stream' => true,
                'messages' => [['role' => 'user', 'content' => 'hello']],
            ], [
            'stream' => function($data) use ($connection) {
                // OpenAI API ডেটা ফেরত দিলে ব্রাউজারে ফরওয়ার্ড করুন
                $connection->send(new Chunk(json_encode($data, JSON_UNESCAPED_UNICODE) . "\n"));
            },
            'complete' => function($result, $response) use ($connection) {
                // প্রতিক্রিয়া সম্পূর্ণ হলে ত্রুটি পরীক্ষা করুন
                if (isset($result['error'])) {
                    $connection->send(new Chunk(json_encode($result, JSON_UNESCAPED_UNICODE) . "\n"));
                }
                // প্রতিক্রিয়া শেষ নির্দেশ করতে খালি chunk পাঠান
                $connection->send(new Chunk(''));
            },
        ]);
        // প্রথমে HTTP হেডার ফেরত দিন, ডেটা অ্যাসিঙ্ক্রোনাসভাবে ফেরত দেওয়া হবে
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```
