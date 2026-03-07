# मूलभूत प्लगइन बनाने और प्रकाशित करने की प्रक्रिया

## सिद्धांत
1. क्रॉस-ऑरिजिन प्लगइन उदाहरण के तौर पर: प्लगइन के तीन भाग होते हैं – क्रॉस-ऑरिजिन मिडलवेयर फ़ाइल, कॉन्फ़िग फ़ाइल `middleware.php`, और कमांड से स्वतः बनने वाला `Install.php`।
2. इन तीन फ़ाइलों को पैक करके Composer पर प्रकाशित करने के लिए कमांड इस्तेमाल होता है।
3. जब उपयोगकर्ता Composer से क्रॉस-ऑरिजिन प्लगइन इंस्टॉल करता है, तो `Install.php` मिडलवेयर और कॉन्फ़िग को `{मुख्य प्रोजेक्ट}/config/plugin` में कॉपी करता है ताकि webman लोड कर सके और क्रॉस-ऑरिजिन सक्रिय हो जाए।
4. जब उपयोगकर्ता Composer से प्लगइन हटाता है, तो `Install.php` संबंधित मिडलवेयर और कॉन्फ़िग फ़ाइलें हटा देता है और प्लगइन स्वतः अनइंस्टॉल हो जाता है।

## निर्देश
1. प्लगइन का नाम दो भागों में: `vendor` और `प्लगइन का नाम`, जैसे `webman/push`, Composer पैकेज के नाम के अनुरूप।
2. प्लगइन कॉन्फ़िग फ़ाइलें `config/plugin/vendor/प्लगइन का नाम/` के तहत रखी जाती हैं (कंसोल कमांड कॉन्फ़िग डायरेक्टरी खुद बनाता है)। प्लगइन को कॉन्फ़िग की ज़रूरत न हो तो ऑटो-बनाई डायरेक्टरी हटानी होगी।
3. प्लगइन कॉन्फ़िग डायरेक्टरी केवल समर्थन करती है: `app.php` (मुख्य कॉन्फ़िग), `bootstrap.php` (प्रोसेस स्टार्ट), `route.php` (रूट), `middleware.php` (मिडलवेयर), `process.php` (कस्टम प्रोसेस), `database.php` (डेटाबेस), `redis.php` (Redis), `thinkorm.php` (thinkorm)। ये webman द्वारा स्वतः पहचाने जाते हैं।
4. कॉन्फ़िग एक्सेस: `config('plugin.vendor.प्लगइन का नाम.कॉन्फ़िग फ़ाइल.आइटम');`, जैसे `config('plugin.webman.push.app.app_key')`।
5. प्लगइन की अपनी डेटाबेस कॉन्फ़िग हो तो: `illuminate/database` के लिए `Db::connection('plugin.vendor.प्लगइन का नाम.कनेक्शन')`, `thinkorm` के लिए `Db::connect('plugin.vendor.प्लगइन का नाम.कनेक्शन')`।
6. प्लगइन को `app/` में बिजनेस फ़ाइलें रखनी हों तो मुख्य प्रोजेक्ट और दूसरे प्लगइन से टकराव से बचना होगा।
7. प्लगइन को मुख्य प्रोजेक्ट में फ़ाइलें/फोल्डर्स कॉपी करने से बचना चाहिए। जैसे क्रॉस-ऑरिजिन प्लगइन में सिर्फ कॉन्फ़िग कॉपी होती है; मिडलवेयर फ़ाइलें `vendor/webman/cros/src` में रहती हैं।
8. प्लगइन नेमस्पेस के लिए PascalCase सुझाया जाता है, जैसे `Webman/Console`।

## उदाहरण

**`webman/console` कमांड लाइन इंस्टॉल करें**

`composer require webman/console`

### प्लगइन बनाएं

मान लें बनाने वाले प्लगइन का नाम `foo/admin` है (Composer पर प्रकाशित करने वाला प्रोजेक्ट नाम, छोटे अक्षर में)। चलाएं:

`php webman plugin:create --name=foo/admin`

इससे `vendor/foo/admin` (प्लगइन फ़ाइलें) और `config/plugin/foo/admin` (कॉन्फ़िग) बनेंगे।

> नोट
> `config/plugin/foo/admin` समर्थन करता है: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`। webman जैसा फॉर्मेट, ऑटो मर्ज।
> एक्सेस में `plugin` प्रीफिक्स इस्तेमाल करें, जैसे `config('plugin.foo.admin.app')`।


### प्लगइन एक्सपोर्ट करें

डेवलपमेंट पूरा होने के बाद चलाएं:

`php webman plugin:export --name=foo/admin`

एक्सपोर्ट

> स्पष्टीकरण
> एक्सपोर्ट पर `config/plugin/foo/admin` कॉपी होकर `vendor/foo/admin/src` में जाती है और `Install.php` बनता है। इंस्टॉल/अनइंस्टॉल के समय `Install.php` चलता है।
> डिफ़ॉल्ट इंस्टॉल: `vendor/foo/admin/src` का कॉन्फ़िग प्रोजेक्ट के `config/plugin` में कॉपी होता है।
> डिफ़ॉल्ट अनइंस्टॉल: प्रोजेक्ट के `config/plugin` से संबंधित कॉन्फ़िग फ़ाइलें हट जाती हैं।
> इंस्टॉल/अनइंस्टॉल पर कस्टम लॉजिक के लिए `Install.php` में बदलाव कर सकते हैं।

### प्लगइन सबमिट करें
* मान लें आपके पास [GitHub](https://github.com) और [Packagist](https://packagist.org) अकाउंट हैं।
* [GitHub](https://github.com) पर `admin` रिपॉजिटरी बनाएं और कोड पुश करें, जैसे `https://github.com/आपका-यूज़रनेम/admin`।
* `https://github.com/आपका-यूज़रनेम/admin/releases/new` पर जाकर रिलीज़ बनाएं, जैसे `v1.0.0`।
* [Packagist](https://packagist.org) पर `Submit` क्लिक करके `https://github.com/आपका-यूज़रनेम/admin` भेजें, प्लगइन प्रकाशित हो जाएगा।

> **सुझाव**
> Packagist पर नाम कॉन्फ्लिक्ट हो तो दूसरा vendor चुनें, जैसे `foo/admin` को `myfoo/admin` कर दें।

अपडेट पर: कोड GitHub पर पुश करें, `https://github.com/आपका-यूज़रनेम/admin/releases/new` पर नई रिलीज़ बनाएं, फिर `https://packagist.org/packages/foo/admin` पर `Update` क्लिक करें।

## प्लगइन में कमांड जोड़ें
कुछ प्लगइन को कस्टम कमांड चाहिए। जैसे `webman/redis-queue` इंस्टॉल करने पर प्रोजेक्ट में `redis-queue:consumer` कमांड मिलता है। `php webman redis-queue:consumer send-mail` चलाने पर तेज़ी से `SendMail.php` कंज्यूमर क्लास बनती है, डेवलपमेंट आसान हो जाती है।

`foo/admin` प्लगइन में `foo-admin:add` कमांड जोड़ने के लिए:

### कमांड बनाएं

**फ़ाइल `vendor/foo/admin/src/FooAdminAddCommand.php` बनाएं**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'कमांड का विवरण';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **नोट**
> प्लगइनों के कमांड टकराव से बचने के लिए `vendor-plugin:कमांड` फॉर्मेट इस्तेमाल करें। जैसे `foo/admin` के सभी कमांड को `foo-admin:` प्रीफिक्स होना चाहिए, जैसे `foo-admin:add`।

### कॉन्फ़िग जोड़ें
**`config/plugin/foo/admin/command.php` बनाएं**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // ज़रूरत के अनुसार और जोड़ें...
];
```

> **सुझाव**
> `command.php` प्लगइन के कस्टम कमांड रजिस्टर करता है। हर एंट्री एक कमांड क्लास है। `webman/console` इन्हें खुद लोड करता है। अधिक जानकारी [कंसोल कमांड](console.md) में।

### एक्सपोर्ट चलाएं
`php webman plugin:export --name=foo/admin` चलाकर प्लगइन एक्सपोर्ट करें और Packagist पर प्रकाशित करें। `foo/admin` इंस्टॉल के बाद `foo-admin:add` कमांड उपलब्ध होगा। `php webman foo-admin:add jerry` चलाने पर `Admin add jerry` प्रिंट होगा।
