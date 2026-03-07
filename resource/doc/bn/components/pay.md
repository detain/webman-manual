# পেমেন্ট SDK


## প্রজেক্ট ঠিকানা

 https://github.com/yansongda/pay

## ইনস্টলেশন

```php
composer require yansongda/pay -vvv
```

## ব্যবহার

**আলিপে**

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
        // এনক্রিপশন পদ্ধতি: **RSA2**
        'private_key' => 'MIIEpAIBAAKCAQEAs6+F2leOgOrvj9jTeDhb5q46GewOjqLBlGSs/bVL4Z3fMr3p+Q1Tux/6uogeVi/eHd84xvQdfpZ87A1SfoWnEGH5z15yorccxSOwWUI+q8gz51IWqjgZxhWKe31BxNZ+prnQpyeMBtE25fXp5nQZ/pftgePyUUvUZRcAUisswntobDQKbwx28VCXw5XB2A+lvYEvxmMv/QexYjwKK4M54j435TuC3UctZbnuynSPpOmCu45ZhEYXd4YMsGMdZE5/077ZU1aU7wx/gk07PiHImEOCDkzqsFo0Buc/knGcdOiUDvm2hn2y1XvwjyFOThsqCsQYi4JmwZdRa8kvOf57nwIDAQABAoIBAQCw5QCqln4VTrTvcW+msB1ReX57nJgsNfDLbV2dG8mLYQemBa9833DqDK6iynTLNq69y88ylose33o2TVtEccGp8Dqluv6yUAED14G6LexS43KtrXPgugAtsXE253ZDGUNwUggnN1i0MW2RcMqHdQ9ORDWvJUCeZj/AEafgPN8AyiLrZeL07jJz/uaRfAuNqkImCVIarKUX3HBCjl9TpuoMjcMhz/MsOmQ0agtCatO1eoH1sqv5Odvxb1i59c8Hvq/mGEXyRuoiDo05SE6IyXYXr84/Nf2xvVNHNQA6kTckj8shSi+HGM4mO1Y4Pbb7XcnxNkT0Inn6oJMSiy56P+CpAoGBAO1O+5FE1ZuVGuLb48cY+0lHCD+nhSBd66B5FrxgPYCkFOQWR7pWyfNDBlmO3SSooQ8TQXA25blrkDxzOAEGX57EPiipXr/hy5e+WNoukpy09rsO1TMsvC+v0FXLvZ+TIAkqfnYBgaT56ku7yZ8aFGMwdCPL7WJYAwUIcZX8wZ3dAoGBAMHWplAqhe4bfkGOEEpfs6VvEQxCqYMYVyR65K0rI1LiDZn6Ij8fdVtwMjGKFSZZTspmsqnbbuCE/VTyDzF4NpAxdm3cBtZACv1Lpu2Om+aTzhK2PI6WTDVTKAJBYegXaahBCqVbSxieR62IWtmOMjggTtAKWZ1P5LQcRwdkaB2rAoGAWnAPT318Kp7YcDx8whOzMGnxqtCc24jvk2iSUZgb2Dqv+3zCOTF6JUsV0Guxu5bISoZ8GdfSFKf5gBAo97sGFeuUBMsHYPkcLehM1FmLZk1Q+ljcx3P1A/ds3kWXLolTXCrlpvNMBSN5NwOKAyhdPK/qkvnUrfX8sJ5XK2H4J8ECgYAGIZ0HIiE0Y+g9eJnpUFelXvsCEUW9YNK4065SD/BBGedmPHRC3OLgbo8X5A9BNEf6vP7fwpIiRfKhcjqqzOuk6fueA/yvYD04v+Da2MzzoS8+hkcqF3T3pta4I4tORRdRfCUzD80zTSZlRc/h286Y2eTETd+By1onnFFe2X01mwKBgQDaxo4PBcLL2OyVT5DoXiIdTCJ8KNZL9+kV1aiBuOWxnRgkDjPngslzNa1bK+klGgJNYDbQqohKNn1HeFX3mYNfCUpuSnD2Yag53Dd/1DLO+NxzwvTu4D6DCUnMMMBVaF42ig31Bs0jI3JQZVqeeFzSET8fkoFopJf3G6UXlrIEAQ==',
        // পাবলিক কী সার্টিফিকেট মোড ব্যবহারের জন্য, নিচের দুটি প্যারামিটার কনফিগার করুন এবং ali_public_key কে .crt দিয়ে শেষ হওয়া আলিপে পাবলিক কী সার্টিফিকেট পথে পরিবর্তন করুন, যেমন (./cert/alipayCertPublicKey_RSA2.crt)
        // 'app_cert_public_key' => './cert/appCertPublicKey.crt', // অ্যাপ্লিকেশন পাবলিক কী সার্টিফিকেট পথ
        // 'alipay_root_cert' => './cert/alipayRootCert.crt', // আলিপে রুট সার্টিফিকেট পথ
        'log' => [ // ঐচ্ছিক
            'file' => './logs/alipay.log',
            'level' => 'info', // প্রোডাকশনে info এবং ডেভলপমেন্টে debug ব্যবহারের পরামর্শ
            'type' => 'single', // ঐচ্ছিক, daily নির্বাচনযোগ্য
            'max_file' => 30, // ঐচ্ছিক, type daily হলে কার্যকর, ডিফল্ট ৩০ দিন
        ],
        'http' => [ // ঐচ্ছিক
            'timeout' => 5.0,
            'connect_timeout' => 5.0,
            // আরও কনফিগার অপশন [Guzzle](https://guzzle-cn.readthedocs.io/zh_CN/latest/request-options.html) দেখুন
        ],
        'mode' => 'dev', // ঐচ্ছিক, এই প্যারামিটার সেট করলে স্যান্ডবক্স মোডে প্রবেশ করবে
    ];

    public function index()
    {
        $order = [
            'out_trade_no' => time(),
            'total_amount' => '1',
            'subject' => 'test subject - পরীক্ষা',
        ];

        $alipay = Pay::alipay($this->config)->web($order);

        return $alipay->send();// Laravel ফ্রেমওয়ার্কে সরাসরি `return $alipay` ব্যবহার করুন
    }

    public function return()
    {
        $data = Pay::alipay($this->config)->verify(); // হ্যাঁ, সিগনেচার ভেরিফিকেশন এতই সহজ!

        // অর্ডার নম্বর: $data->out_trade_no
        // আলিপে ট্রানজেকশন নম্বর: $data->trade_no
        // অর্ডার মোট পরিমাণ: $data->total_amount
    }

    public function notify()
    {
        $alipay = Pay::alipay($this->config);
    
        try{
            $data = $alipay->verify(); // হ্যাঁ, সিগনেচার ভেরিফিকেশন এতই সহজ!

            // trade_status এবং অন্যান্য লজিক নিজে যাচাই করুন। আলিপে বিজনেস নোটিফিকেশনে কেবল TRADE_SUCCESS বা TRADE_FINISHED হলে ক্রেতার পেমেন্ট সফল বলে বিবেচিত হয়।
            // 1. মারচেন্ট সিস্টেমে তৈরি অর্ডার নম্বরের সাথে নোটিফিকেশন ডেটার out_trade_no মিলে কিনা যাচাই করুন;
            // 2. total_amount সত্যিই সেই অর্ডারের প্রকৃত পরিমাণ কিনা যাচাই করুন;
            // 3. নোটিফিকেশনের seller_id (বা seller_email) out_trade_no এর সাথে মিলে কিনা যাচাই করুন;
            // 4. app_id মারচেন্ট নিজের কিনা যাচাই করুন।
            // 5. অন্যান্য বিজনেস লজিক

            Log::debug('Alipay notify', $data->all());
        } catch (\Exception $e) {
            // $e->getMessage();
        }

        return $alipay->success()->send();// Laravel ফ্রেমওয়ার্কে সরাসরি `return $alipay->success()` ব্যবহার করুন
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
        'app_id' => 'wxb3fxxxxxxxxxxx', // অফিসিয়াল অ্যাকাউন্ট APPID
        'miniapp_id' => 'wxb3fxxxxxxxxxxx', // মিনি প্রোগ্রাম APPID
        'mch_id' => '14577xxxx',
        'key' => 'mF2suE9sU6Mk1Cxxxxxxxxxxx',
        'notify_url' => 'http://yanda.net.cn/notify.php',
        'cert_client' => './cert/apiclient_cert.pem', // ঐচ্ছিক, রিফান্ড ইত্যাদিতে ব্যবহৃত
        'cert_key' => './cert/apiclient_key.pem',// ঐচ্ছিক, রিফান্ড ইত্যাদিতে ব্যবহৃত
        'log' => [ // ঐচ্ছিক
            'file' => './logs/wechat.log',
            'level' => 'info', // প্রোডাকশনে info এবং ডেভলপমেন্টে debug ব্যবহারের পরামর্শ
            'type' => 'single', // ঐচ্ছিক, daily নির্বাচনযোগ্য
            'max_file' => 30, // ঐচ্ছিক, type daily হলে কার্যকর, ডিফল্ট ৩০ দিন
        ],
        'http' => [ // ঐচ্ছিক
            'timeout' => 5.0,
            'connect_timeout' => 5.0,
            // আরও কনফিগার অপশন [Guzzle](https://guzzle-cn.readthedocs.io/zh_CN/latest/request-options.html) দেখুন
        ],
        'mode' => 'dev', // ঐচ্ছিক, dev/hk; `hk` হলে হংকং গেটওয়ে
    ];

    public function index()
    {
        $order = [
            'out_trade_no' => time(),
            'total_fee' => '1', // **ইউনিট: সেন্ট**
            'body' => 'test body - পরীক্ষা',
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
            $data = $pay->verify(); // হ্যাঁ, সিগনেচার ভেরিফিকেশন এতই সহজ!

            Log::debug('Wechat notify', $data->all());
        } catch (\Exception $e) {
            // $e->getMessage();
        }
        
        return $pay->success()->send();// Laravel ফ্রেমওয়ার্কে সরাসরি `return $pay->success()` ব্যবহার করুন
    }
}
```

## আরও তথ্য

ভিজিট করুন https://pay.yanda.net.cn/docs/2.x/overview
