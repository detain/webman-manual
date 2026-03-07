# डेटाबेस

चूंकि अधिकांश प्लगइन [webman-admin](https://www.workerman.net/plugin/82) इंस्टॉल करते हैं, इसलिए `webman-admin` के डेटाबेस कॉन्फ़िगरेशन को सीधे पुनः उपयोग करने की सलाह दी जाती है।

जिन मॉडल्स का बेस क्लास `plugin\admin\app\model\Base` है, वे स्वचालित रूप से webman-admin के डेटाबेस कॉन्फ़िगरेशन का उपयोग करेंगे।
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

आप `plugin.admin.mysql` के माध्यम से webman-admin डेटाबेस तक पहुंच भी सकते हैं, उदाहरण के लिए:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## अपना डेटाबेस उपयोग करना

प्लगइन अपना स्वयं का डेटाबेस उपयोग करना भी चुन सकते हैं। उदाहरण के लिए, `plugin/foo/config/database.php` की सामग्री:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql कनेक्शन का नाम है
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'डेटाबेस',
            'username'    => 'उपयोगकर्ता_नाम',
            'password'    => 'पासवर्ड',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin कनेक्शन का नाम है
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'डेटाबेस',
            'username'    => 'उपयोगकर्ता_नाम',
            'password'    => 'पासवर्ड',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

संदर्भ प्रारूप है `Db::connection('plugin.{प्लगइन}.{कनेक्शन_नाम}');`, उदाहरण के लिए:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

मुख्य प्रोजेक्ट के डेटाबेस का उपयोग करने के लिए, सीधे इसे कॉल करें:

```php
use support\Db;
Db::table('user')->first();
// मान लें कि मुख्य प्रोजेक्ट में admin कनेक्शन भी कॉन्फ़िगर है
Db::connection('admin')->table('admin')->first();
```

#### Model के लिए डेटाबेस कॉन्फ़िगर करना

आप Model के लिए Base क्लास बना सकते हैं और प्लगइन के स्वयं के डेटाबेस कनेक्शन का उपयोग करने के लिए `$connection` प्रॉपर्टी सेट कर सकते हैं:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

इस तरह प्लगइन में सभी मॉडल्स जो Base से इनहैरिट करते हैं, स्वचालित रूप से प्लगइन के स्वयं के डेटाबेस का उपयोग करेंगे।

## स्वचालित डेटाबेस इम्पोर्ट

`php webman app-plugin:create foo` चलाने पर foo प्लगइन बनाया जाता है, जिसमें `plugin/foo/api/Install.php` और `plugin/foo/install.sql` शामिल होते हैं।

> **सुझाव**
> यदि install.sql फ़ाइल जनरेट नहीं होती है, तो `webman/console` अपग्रेड का प्रयास करें: `composer require webman/console ^1.3.6`

#### प्लगइन इंस्टॉल करते समय डेटाबेस इम्पोर्ट करना

जब प्लगइन इंस्टॉल होता है, Install.php में `install` मेथड चलता है, जो `install.sql` में SQL स्टेटमेंट्स को स्वचालित रूप से निष्पादित करता है, इस प्रकार डेटाबेस टेबल इम्पोर्ट होते हैं।

`install.sql` की सामग्री टेबल निर्माण और ऐतिहासिक स्कीमा परिवर्तन होनी चाहिए। प्रत्येक स्टेटमेंट `;` से समाप्त होना चाहिए, उदाहरण के लिए:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'प्राथमिक कुंजी',
  `order_id` varchar(50) NOT NULL COMMENT 'ऑर्डर ID',
  `user_id` int NOT NULL COMMENT 'उपयोगकर्ता ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'भुगतान राशि',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='ऑर्डर';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'प्राथमिक कुंजी',
  `name` varchar(50) NOT NULL COMMENT 'नाम',
  `price` int NOT NULL COMMENT 'कीमत',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='वस्तुएं';
```

**डेटाबेस कनेक्शन बदलना**

डिफ़ॉल्ट रूप से `install.sql` webman-admin के डेटाबेस में इम्पोर्ट होता है। दूसरे डेटाबेस में इम्पोर्ट करने के लिए, Install.php में `$connection` प्रॉपर्टी संशोधित करें:

```php
<?php

class Install
{
    // प्लगइन के स्वयं के डेटाबेस को निर्दिष्ट करें
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**टेस्टिंग**

प्लगइन इंस्टॉल करने के लिए `php webman app-plugin:install foo` चलाएं। फिर डेटाबेस जांचें — `foo_orders` और `foo_goods` टेबल बन चुके होंगे।

#### प्लगइन अपग्रेड के दौरान टेबल स्ट्रक्चर बदलना

जब प्लगइन अपग्रेड को नई टेबल या स्कीमा परिवर्तन की आवश्यकता होती है, तो संबंधित SQL स्टेटमेंट्स को `install.sql` के अंत में जोड़ें। प्रत्येक स्टेटमेंट `;` से समाप्त होना चाहिए। उदाहरण के लिए, `foo_user` टेबल जोड़ना और `foo_orders` टेबल में `status` कॉलम जोड़ना:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'प्राथमिक कुंजी',
  `order_id` varchar(50) NOT NULL COMMENT 'ऑर्डर ID',
  `user_id` int NOT NULL COMMENT 'उपयोगकर्ता ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'भुगतान राशि',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='ऑर्डर';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'प्राथमिक कुंजी',
 `name` varchar(50) NOT NULL COMMENT 'नाम',
 `price` int NOT NULL COMMENT 'कीमत',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='वस्तुएं';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'प्राथमिक कुंजी',
 `name` varchar(50) NOT NULL COMMENT 'नाम'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='उपयोगकर्ता';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'स्थिति';
```

अपग्रेड के दौरान Install.php में `update` मेथड चलता है, जो `install.sql` में स्टेटमेंट्स को निष्पादित करता है। नए निष्पादित होते हैं; पहले से लागू किए गए छोड़ दिए जाते हैं, इस प्रकार अपग्रेड के दौरान डेटाबेस परिवर्तन सही ढंग से लागू होते हैं।

#### प्लगइन अनइंस्टॉल करते समय डेटाबेस हटाना

जब प्लगइन अनइंस्टॉल होता है, Install.php में `uninstall` मेथड कॉल होता है। यह `install.sql` में CREATE TABLE स्टेटमेंट्स का स्वचालित रूप से विश्लेषण करता है और उन टेबल्स को हटा देता है।

यदि आप केवल अपना `uninstall.sql` निष्पादित करना चाहते हैं, स्वचालित टेबल हटाना नहीं, तो `plugin/{प्लगइन_नाम}/uninstall.sql` बनाएं। उस स्थिति में `uninstall` मेथड केवल उस फ़ाइल में स्टेटमेंट्स निष्पादित करेगा।
