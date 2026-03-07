# طابور Stomp

Stomp هو بروتوكول رسائل نصية بسيط (تدفقي) يوفر صيغة اتصال قابلة للتشغيل البيني، مما يسمح لعملاء STOMP بالتفاعل مع أي وسيط رسائل STOMP (Broker). [workerman/stomp](https://github.com/walkor/stomp) ينفذ عميل Stomp، ويُستخدم بشكل أساسي في سيناريوهات طوابير الرسائل مثل RabbitMQ وApollo وActiveMQ وغيرها.

## التثبيت
`composer require webman/stomp`

## التكوين
ملف التكوين موجود في `config/plugin/webman/stomp`

## إرسال الرسائل
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // الطابور
        $queue = 'examples';
        // البيانات (عند تمرير مصفوفة، يجب التسلسل يدوياً، مثل استخدام json_encode أو serialize)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // تنفيذ الإرسال
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> لضمان التوافق مع المشاريع الأخرى، لا يوفر مكون Stomp التسلسل وإلغاء التسلسل التلقائي. إذا كانت البيانات المرسلة مصفوفة، يجب التسلسل يدوياً وإلغاء التسلسل عند الاستهلاك.

## استهلاك الرسائل
أنشئ ملفاً جديداً `app/queue/stomp/MyMailSend.php` (اسم الفئة يمكن أن يكون أي شيء طالما يلتزم بمعيار PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // اسم الطابور
    public $queue = 'examples';

    // اسم الاتصال، يقابل الاتصال في stomp.php
    public $connection = 'default';

    // عندما تكون القيمة client، يجب استدعاء $ack_resolver->ack() لإبلاغ الخادم بالاستهلاك الناجح
    // عندما تكون القيمة auto، لا حاجة لاستدعاء $ack_resolver->ack()
    public $ack = 'auto';

    // الاستهلاك
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // إذا كانت البيانات مصفوفة، يجب إلغاء التسلسل يدوياً
        var_export(json_decode($data, true)); // يخرج ['to' => 'tom@gmail.com', 'content' => 'hello']
        // إبلاغ الخادم بالاستهلاك الناجح
        $ack_resolver->ack(); // يمكن حذف هذا الاستدعاء عندما يكون ack هو auto
    }
}
```

# تفعيل بروتوكول Stomp في RabbitMQ
RabbitMQ لا يفعل بروتوكول Stomp افتراضياً. نفّذ الأمر التالي للتفعيل:
```
rabbitmq-plugins enable rabbitmq_stomp
```
بعد التفعيل، المنفذ الافتراضي لـ Stomp هو 61613.
