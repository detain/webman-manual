# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) [top-think/think-orm](https://github.com/top-think/think-orm) पर आधारित एक डेटाबेस घटक है। यह कनेक्शन पूल का समर्थन करता है और कॉरोटाइन तथा गैर-कॉरोटाइन दोनों वातावरण में काम करता है।

## इंस्टॉलेशन

`composer require -W webman/think-orm`

इंस्टॉलेशन के बाद restart (पुनः आरंभ) आवश्यक है (reload प्रभावी नहीं होता)।

## कॉन्फ़िगरेशन फ़ाइल

अपनी आवश्यकता के अनुसार कॉन्फ़िगरेशन फ़ाइल `config/think-orm.php` संशोधित करें।

## प्रलेखन

https://www.kancloud.cn/manual/think-orm

## उपयोग

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## मॉडल बनाना

think-orm मॉडल `support\think\Model` को विस्तारित करते हैं, नीचे दिखाए अनुसार:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

आप निम्न कमांड से भी think-orm मॉडल बना सकते हैं:

```
php webman make:model तालिका_नाम
```

> **सुझाव**
> इस कमांड के लिए `webman/console` आवश्यक है। इसे इससे इंस्टॉल करें: `composer require webman/console ^1.2.13`

> **ध्यान दें**
> यदि `make:model` को पता चलता है कि मुख्य प्रोजेक्ट `illuminate/database` का उपयोग कर रहा है, तो वह think-orm की बजाय Illuminate-आधारित मॉडल फ़ाइलें बनाएगा। उस स्थिति में, `tp` पैरामीटर जोड़कर think-orm मॉडल बनाएं: `php webman make:model तालिका_नाम tp` (काम न करे तो `webman/console` अपडेट करें)।
