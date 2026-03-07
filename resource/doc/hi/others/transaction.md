# ट्रांजैक्शन का सही उपयोग

webman में डेटाबेस ट्रांजैक्शन का उपयोग अन्य फ्रेमवर्क की तरह ही है। यहाँ ध्यान देने वाली बातें बताई गई हैं।

## कोड संरचना

कोड संरचना अन्य फ्रेमवर्क (जैसे Laravel, think-orm) के समान है:

```php
Db::beginTransaction();
try {
    // ..व्यापार तर्क छोड़ा गया...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**महत्वपूर्ण:** `\Exception` के बजाय **अवश्य** `\Throwable` का उपयोग करें, क्योंकि व्यापार तर्क के दौरान `Error` ट्रिगर हो सकता है जो `Exception` से विरासत में नहीं मिलता।

## डेटाबेस कनेक्शन

ट्रांजैक्शन के भीतर मॉडल पर काम करते समय, ध्यान दें कि मॉडल में कनेक्शन सेट है या नहीं। अगर मॉडल में कनेक्शन निर्दिष्ट है तो ट्रांजैक्शन शुरू करते समय उस कनेक्शन को निर्दिष्ट करना होगा, नहीं तो ट्रांजैक्शन काम नहीं करेगा (think-orm समान)। उदाहरण:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // यहाँ मॉडल के लिए कनेक्शन निर्दिष्ट है
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

जब मॉडल में कनेक्शन निर्दिष्ट हो, तो begin, commit और rollback सभी में कनेक्शन निर्दिष्ट करना होगा:

```php
Db::connection('mysql')->beginTransaction();
try {
    // व्यापार प्रसंस्करण
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## कमिट नहीं हुए ट्रांजैक्शन वाले अनुरोध ढूँढना
कभी-कभी व्यापार कोड में बग की वजह से किसी अनुरोध का ट्रांजैक्शन कमिट नहीं होता। उस कंट्रोलर मेथड को जल्दी ढूँढने के लिए जहाँ ट्रांजैक्शन कमिट नहीं हुआ, `webman/log` कंपोनेंट इंस्टॉल कर सकते हैं। यह कंपोनेंट प्रत्येक अनुरोध समाप्त होने के बाद अपने आप जाँचता है कि कोई कमिट नहीं हुआ ट्रांजैक्शन तो नहीं है, और लॉग में दर्ज करता है। लॉग की कीवर्ड है `Uncommitted transactions`

**webman/log इंस्टॉल करने का तरीका**

`composer require webman/log`

> **ध्यान दें**
> इंस्टॉल के बाद restart आवश्यक है, reload काम नहीं करेगा
