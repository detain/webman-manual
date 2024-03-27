# इवेंट इवेंट हैंडलिंग
`webman/event` एक सुगम इवेंट मैकेनिज़्म प्रदान करता है, जो कोड छेदना बिना कुछ बिजनेस लॉजिक निष्पादित करने की क्षमता प्रदान करता है, व्यापार मॉड्यूल के बीच छेद निष्पादित करने की क्षमता प्रदान करता है। एक सामान्य स्थिति का उदाहरण है जैसे कि एक नए उपयोगकर्ता पंजीकरण सफलतापूर्वक होता है, बस एक कस्टम इवेंट जैसे `user.register` प्रकार का प्रकाशित करने पर, हर एक मॉड्यूल ताकता है कि प्रेरित इवेंट को निष्पादित करने के लिए आवश्यक व्यवसायिक लॉजिक को निष्पादित करे।

## स्थापना
`composer require webman/event`

## इवेंट पर सदस्यता
इवेंट पर सदस्यता की स्थापना फ़ाइल `config/event.php` के माध्यम से होती है
```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...अन्य परिणाम संसाधित कार्य...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...अन्य परिणाम संसाधित कार्य...
    ]
];
```
**स्पष्टीकरण:**
- `user.register` `user.logout` आदि इवेंट नाम हैं, स्ट्रिंग प्रकार है, सुझाव दिया गया है कि शब्दों को लोअरकेस में रखा जाए और डॉट (`.`) से अलग किया जाए
- एक इवेंट कई इवेंट हैंडलिंग कार्यों के लिए संबंधित हो सकता है, कॉलिंग के अनुसार कॉन्फ़िगरेशन का अनुरोध किया जाता है

## इवेंट हैंडलिंग करने वाले फ़ंक्शन
इवेंट हैंडलिंग फ़ंक्शन कोई भी क्लास मेथड, फ़ंक्शन, बंद फ़ंक्शन आदि हो सकते हैं।
उदाहरण के तौर पर इवेंट हैंडलिंग क्लास `app/event/User.php` को बनाएं (डिरेक्टरी मौजूद नहीं हो तो कृपया स्वयं बनाएं)
```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## इवेंट प्रकाशित करना
`Event::emit($event_name, $data);` का उपयोग करके इवेंट प्रकाशित करें, उदाहरण के लिए
```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::emit('user.register', $user);
    }
}
```

> **सूचना**
> `Event::emit($event_name, $data);` पैरामीटर `$data` किसी भी डेटा हो सकता है, जैसे कि एरे, क्लास इंस्टेंस, स्ट्रिंग आदि 

## वाइल्डकार्ड इवेंट सुनना
वाइल्डकार्ड प्राप्तकर्ता सुनना आपको एक ही सुनने वाले पर अन्याय इवेंट का संबद्ध करने की अनुमति देता है, उदाहरण के लिए `config/event.php` में सेट करें
```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```
हम `event_data` के द्वारा इवेंट हैंडलिंग फ़ंक्शन दूसरे पैरामीटर `$event_data` को प्राप्त कर सकते हैं
```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // विशिष्ट इवेंट नाम, जैसे कि user.register user.logout आदि
        var_export($user);
    }
}
```

## इवेंट प्रकार को रोकना
जब हम इवेंट हैंडलिंग फ़ंक्शन में `false` लौटाते हैं, तो ऐसा इवेंट संपादक पर काम करना बंद हो जाएगा

## बंद फ़ंक्शन से इवेंट हैंडलिंग
इवेंट हैंडलिंग फ़ंक्शन क्लास मेथड हो सकती है, यहां की तौर पर

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## इवेंट और सुनने वाले को देखें
`php webman event:list` कमांड का प्रयोग करके परियोजना में संपूर्ण इवेंट और सुनने वाले को देखें

## ध्यान देने योग्य बिंदु
इवेंट इवेंट हैंडलिंग वास्तव में असिंक्रोनिज़ नहीं होती है, इवेंट सुंदर व्यापार को नहीं संभालने के लिए उपयुक्त नहीं होती है, धीमे व्यापार को संदेश कतरीनी सुनने के लिए उपयोग करना चाहिए, जैसे कि [webman/redis-queue](https://www.workerman.net/plugin/12)