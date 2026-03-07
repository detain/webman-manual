# Stomp কিউ

Stomp একটি সরল (স্ট্রিমিং) পাঠ্য-ভিত্তিক বার্তা প্রোটোকল যা একটি আন্তঃপরিচালনযোগ্য সংযোগ বিন্যাস প্রদান করে, যাতে STOMP ক্লায়েন্ট যেকোনো STOMP বার্তা ব্রোকার (Broker) এর সাথে যোগাযোগ করতে পারে। [workerman/stomp](https://github.com/walkor/stomp) Stomp ক্লায়েন্ট বাস্তবায়ন করে এবং প্রধানত RabbitMQ, Apollo, ActiveMQ ইত্যাদি বার্তা কিউ দৃশ্যপটে ব্যবহৃত হয়।

## ইনস্টলেশন
`composer require webman/stomp`

## কনফিগারেশন
কনফিগারেশন ফাইল `config/plugin/webman/stomp` এর অধীনে রয়েছে।

## বার্তা প্রেরণ
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // কিউ
        $queue = 'examples';
        // ডেটা (অ্যারে পাঠাতে হলে নিজে সিরিয়ালাইজ করতে হবে, যেমন json_encode, serialize ইত্যাদি ব্যবহার করে)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // প্রেরণ সম্পাদন করুন
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> অন্যান্য প্রকল্পের সাথে সামঞ্জস্যের জন্য Stomp উপাদান স্বয়ংক্রিয় সিরিয়ালাইজেশন এবং ডিসিরিয়ালাইজেশন প্রদান করে না। অ্যারে ডেটা পাঠালে নিজে সিরিয়ালাইজ করতে হবে এবং গ্রহণ করার সময় নিজে ডিসিরিয়ালাইজ করতে হবে।

## বার্তা গ্রহণ
`app/queue/stomp/MyMailSend.php` নতুন তৈরি করুন (ক্লাসের নাম যেকোনো হতে পারে, PSR-4 মান অনুসরণ করলে)।
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // কিউর নাম
    public $queue = 'examples';

    // সংযোগের নাম, stomp.php এ সংযোগের সাথে মিলে যায়
    public $connection = 'default';

    // মান client হলে সফলভাবে গ্রহণ হয়েছে জানাতে $ack_resolver->ack() কল করতে হবে
    // মান auto হলে $ack_resolver->ack() কল করার প্রয়োজন নেই
    public $ack = 'auto';

    // গ্রহণ
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // ডেটা অ্যারে হলে নিজে ডিসিরিয়ালাইজ করতে হবে
        var_export(json_decode($data, true)); // আউটপুট ['to' => 'tom@gmail.com', 'content' => 'hello']
        // সফলভাবে গ্রহণ হয়েছে সার্ভারকে জানান
        $ack_resolver->ack(); // ack auto হলে এই কল বাদ দেওয়া যেতে পারে
    }
}
```

# RabbitMQ এ Stomp প্রোটোকল সক্ষম করা
RabbitMQ ডিফল্টভাবে Stomp প্রোটোকল সক্ষম করে না। সক্ষম করতে নিম্নলিখিত কমান্ড চালান:
```
rabbitmq-plugins enable rabbitmq_stomp
```
সক্ষম করার পরে Stomp এর ডিফল্ট পোর্ট 61613।
