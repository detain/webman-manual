# SDK الدفع


## عنوان المشروع

 https://github.com/yansongda/pay

## التثبيت

```php
composer require yansongda/pay -vvv
```

## الاستخدام

**Alipay**

```php
<?php
namespace App\Http\Controllers;

use Yansongda\Pay\Pay;
use Yansongda\Pay\Log;

class PayController
{
    protected $config = [
        'app_id' => '2016082000295641',
        'notify_url' => 'http://yansongda.cn/notify.php',
        'return_url' => 'http://yansongda.cn/return.php',
        'ali_public_key' => 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuWJKrQ6SWvS6niI+4vEVZiYfjkCfLQfoFI2nCp9ZLDS42QtiL4Ccyx8scgc3nhVwmVRte8f57TFvGhvJD0upT4O5O/lRxmTjechXAorirVdAODpOu0mFfQV9y/T9o9hHnU+VmO5spoVb3umqpq6D/Pt8p25Yk852/w01VTIczrXC4QlrbOEe3sr1E9auoC7rgYjjCO6lZUIDjX/oBmNXZxhRDrYx4Yf5X7y8FRBFvygIE2FgxV4Yw+SL3QAa2m5MLcbusJpxOml9YVQfP8iSurx41PvvXUMo49JG3BDVernaCYXQCoUJv9fJwbnfZd7J5YByC+5KM4sblJTq7bXZWQIDAQAB',
        // طريقة التشفير: **RSA2**
        'private_key' => 'MIIEpAIBAAKCAQEAs6+F2leOgOrvj9jTeDhb5q46GewOjqLBlGSs/bVL4Z3fMr3p+Q1Tux/6uogeVi/eHd84xvQdfpZ87A1SfoWnEGH5z15yorccxSOwWUI+q8gz51IWqjgZxhWKe31BxNZ+prnQpyeMBtE25fXp5nQZ/pftgePyUUvUZRcAUisswntobDQKbwx28VCXw5XB2A+lvYEvxmMv/QexYjwKK4M54j435TuC3UctZbnuynSPpOmCu45ZhEYXd4YMsGMdZE5/077ZU1aU7wx/gk07PiHImEOCDkzqsFo0Buc/knGcdOiUDvm2hn2y1XvwjyFOThsqCsQYi4JmwZdRa8kvOf57nwIDAQABAoIBAQCw5QCqln4VTrTvcW+msB1ReX57nJgsNfDLbV2dG8mLYQemBa9833DqDK6iynTLNq69y88ylose33o2TVtEccGp8Dqluv6yUAED14G6LexS43KtrXPgugAtsXE253ZDGUNwUggnN1i0MW2RcMqHdQ9ORDWvJUCeZj/AEafgPN8AyiLrZeL07jJz/uaRfAuNqkImCVIarKUX3HBCjl9TpuoMjcMhz/MsOmQ0agtCatO1eoH1sqv5Odvxb1i59c8Hvq/mGEXyRuoiDo05SE6IyXYXr84/Nf2xvVNHNQA6kTckj8shSi+HGM4mO1Y4Pbb7XcnxNkT0Inn6oJMSiy56P+CpAoGBAO1O+5FE1ZuVGuLb48cY+0lHCD+nhSBd66B5FrxgPYCkFOQWR7pWyfNDBlmO3SSooQ8TQXA25blrkDxzOAEGX57EPiipXr/hy5e+WNoukpy09rsO1TMsvC+v0FXLvZ+TIAkqfnYBgaT56ku7yZ8aFGMwdCPL7WJYAwUIcZX8wZ3dAoGBAMHWplAqhe4bfkGOEEpfs6VvEQxCqYMYVyR65K0rI1LiDZn6Ij8fdVtwMjGKFSZZTspmsqnbbuCE/VTyDzF4NpAxdm3cBtZACv1Lpu2Om+aTzhK2PI6WTDVTKAJBYegXaahBCqVbSxieR62IWtmOMjggTtAKWZ1P5LQcRwdkaB2rAoGAWnAPT318Kp7YcDx8whOzMGnxqtCc24jvk2iSUZgb2Dqv+3zCOTF6JUsV0Guxu5bISoZ8GdfSFKf5gBAo97sGFeuUBMsHYPkcLehM1FmLZk1Q+ljcx3P1A/ds3kWXLolTXCrlpvNMBSN5NwOKAyhdPK/qkvnUrfX8sJ5XK2H4J8ECgYAGIZ0HIiE0Y+g9eJnpUFelXvsCEUW9YNK4065SD/BBGedmPHRC3OLgbo8X5A9BNEf6vP7fwpIiRfKhcjqqzOuk6fueA/yvYD04v+Da2MzzoS8+hkcqF3T3pta4I4tORRdRfCUzD80zTSZlRc/h286Y2eTETd+By1onnFFe2X01mwKBgQDaxo4PBcLL2OyVT5DoXiIdTCJ8KNZL9+kV1aiBuOWxnRgkDjPngslzNa1bK+klGgJNYDbQqohKNn1HeFX3mYNfCUpuSnD2Yag53Dd/1DLO+NxzwvTu4D6DCUnMMMBVaF42ig31Bs0jI3JQZVqeeFzSET8fkoFopJf3G6UXlrIEAQ==',
        // لاستخدام وضع الشهادة العمومية، قم بتكوين المعلمتين التاليتين وتعديل ali_public_key إلى مسار شهادة المفتاح العمومي Alipay المنتهية بـ .crt، مثل (./cert/alipayCertPublicKey_RSA2.crt)
        // 'app_cert_public_key' => './cert/appCertPublicKey.crt', // مسار شهادة المفتاح العمومي للتطبيق
        // 'alipay_root_cert' => './cert/alipayRootCert.crt', // مسار الشهادة الجذرية لـ Alipay
        'log' => [ // اختياري
            'file' => './logs/alipay.log',
            'level' => 'info', // يُوصى بتعيين المستوى إلى info في بيئة الإنتاج و debug في التطوير
            'type' => 'single', // اختياري، يمكن اختيار daily
            'max_file' => 30, // اختياري، فعال عندما يكون النوع daily، افتراضي 30 يوم
        ],
        'http' => [ // اختياري
            'timeout' => 5.0,
            'connect_timeout' => 5.0,
            // لمزيد من خيارات التكوين، راجع [Guzzle](https://guzzle-cn.readthedocs.io/zh_CN/latest/request-options.html)
        ],
        'mode' => 'dev', // اختياري، يعمل هذا الإعداد على تفعيل وضع الحماية
    ];

    public function index()
    {
        $order = [
            'out_trade_no' => time(),
            'total_amount' => '1',
            'subject' => 'test subject - 测试',
        ];

        $alipay = Pay::alipay($this->config)->web($order);

        return $alipay->send();// في إطار Laravel، استخدم `return $alipay` مباشرة
    }

    public function return()
    {
        $data = Pay::alipay($this->config)->verify(); // نعم، التحقق من التوقيع بهذه البساطة!

        // رقم الطلب: $data->out_trade_no
        // رقم معاملة Alipay: $data->trade_no
        // المبلغ الإجمالي للطلب: $data->total_amount
    }

    public function notify()
    {
        $alipay = Pay::alipay($this->config);
    
        try{
            $data = $alipay->verify(); // نعم، التحقق من التوقيع بهذه البساطة!

            // يُرجى التحقق من trade_status والمنطق الآخر. في إشعارات Alipay التجارية، تُعتبر الدفعة ناجحة فقط عند TRADE_SUCCESS أو TRADE_FINISHED.
            // 1. يجب على التاجر التحقق مما إذا كان out_trade_no في بيانات الإشعار يمثل رقم الطلب المُنشأ في نظامه؛
            // 2. التحقق مما إذا كان total_amount هو المبلغ الفعلي للطلب (أي المبلغ عند إنشاء الطلب)؛
            // 3. التحقق مما إذا كان seller_id (أو seller_email) في الإشعار يطابق الطرف المسؤول عن هذا الطلب (قد يمتلك التاجر عدة seller_id/seller_email)؛
            // 4. التحقق مما إذا كان app_id يمثل التاجر نفسه.
            // 5. منطق الأعمال الآخر

            Log::debug('Alipay notify', $data->all());
        } catch (\Exception $e) {
            // $e->getMessage();
        }

        return $alipay->success()->send();// في إطار Laravel، استخدم `return $alipay->success()` مباشرة
    }
}
```

**WeChat**

```php
<?php

namespace App\Http\Controllers;

use Yansongda\Pay\Pay;
use Yansongda\Pay\Log;

class PayController
{
    protected $config = [
        'appid' => 'wxb3fxxxxxxxxxxx', // APP APPID
        'app_id' => 'wxb3fxxxxxxxxxxx', // حساب عام APPID
        'miniapp_id' => 'wxb3fxxxxxxxxxxx', // تطبيق مصغر APPID
        'mch_id' => '14577xxxx',
        'key' => 'mF2suE9sU6Mk1Cxxxxxxxxxxx',
        'notify_url' => 'http://yanda.net.cn/notify.php',
        'cert_client' => './cert/apiclient_cert.pem', // اختياري، يُستخدم للاسترداد وغيره
        'cert_key' => './cert/apiclient_key.pem',// اختياري، يُستخدم للاسترداد وغيره
        'log' => [ // اختياري
            'file' => './logs/wechat.log',
            'level' => 'info', // يُوصى بتعيين المستوى إلى info في بيئة الإنتاج و debug في التطوير
            'type' => 'single', // اختياري، يمكن اختيار daily
            'max_file' => 30, // اختياري، فعال عندما يكون النوع daily، افتراضي 30 يوم
        ],
        'http' => [ // اختياري
            'timeout' => 5.0,
            'connect_timeout' => 5.0,
            // لمزيد من خيارات التكوين، راجع [Guzzle](https://guzzle-cn.readthedocs.io/zh_CN/latest/request-options.html)
        ],
        'mode' => 'dev', // اختياري، dev/hk؛ عند `hk` يكون بوابة هونغ كونغ
    ];

    public function index()
    {
        $order = [
            'out_trade_no' => time(),
            'total_fee' => '1', // **الوحدة: قرش**
            'body' => 'test body - 测试',
            'openid' => 'onkVf1FjWS5SBIixxxxxxx',
        ];

        $pay = Pay::wechat($this->config)->mp($order);

        // $pay->appId
        // $pay->timeStamp
        // $pay->nonceStr
        // $pay->package
        // $pay->signType
    }

    public function notify()
    {
        $pay = Pay::wechat($this->config);

        try{
            $data = $pay->verify(); // نعم، التحقق من التوقيع بهذه البساطة!

            Log::debug('Wechat notify', $data->all());
        } catch (\Exception $e) {
            // $e->getMessage();
        }
        
        return $pay->success()->send();// في إطار Laravel، استخدم `return $pay->success()` مباشرة
    }
}
```

## مزيد من المعلومات

زيارة https://pay.yanda.net.cn/docs/2.x/overview
