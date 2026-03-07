# स्वचालित लोडिंग

## Composer से PSR-0 अनुरूप फ़ाइलें लोड करना
webman `PSR-4` स्वचालित लोडिंग विनिर्देश का पालन करता है। यदि आपके प्रोजेक्ट को PSR-0 अनुरूप लाइब्रेरी लोड करने की आवश्यकता है, तो इन चरणों का पालन करें:

- PSR-0 लाइब्रेरी रखने के लिए `extend` डायरेक्टरी बनाएँ
- `composer.json` संपादित करें और `autoload` के अंतर्गत यह जोड़ें:

```json
"psr-0" : {
    "": "extend/"
}
```
अंतिम परिणाम इस प्रकार दिखेगा:
![](../../assets/img/psr0.png)

- `composer dumpautoload` चलाएँ
- webman पुनः आरंभ करने के लिए `php start.php restart` चलाएँ (ध्यान दें: परिवर्तन लागू होने के लिए पुनः आरंभ आवश्यक है)

## Composer से कुछ फ़ाइलें लोड करना

- `composer.json` संपादित करें और `autoload.files` के अंतर्गत लोड की जाने वाली फ़ाइलें जोड़ें:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` चलाएँ
- webman पुनः आरंभ करने के लिए `php start.php restart` चलाएँ (ध्यान दें: परिवर्तन लागू होने के लिए पुनः आरंभ आवश्यक है)

> **ध्यान दें**
> composer.json में `autoload.files` से कॉन्फ़िगर फ़ाइलें webman शुरू होने से पहले लोड होती हैं। फ्रेमवर्क की `config/autoload.php` से लोड होने वाली फ़ाइलें webman शुरू होने के बाद लोड होती हैं।
> composer.json की `autoload.files` फ़ाइलों में बदलाव के बाद restart ज़रूरी है; reload से काम नहीं चलेगा। `config/autoload.php` से लोड होने वाली फ़ाइलें hot-reload सपोर्ट करती हैं; reload के बाद बदलाव लागू हो जाते हैं।

## फ्रेमवर्क से कुछ फ़ाइलें लोड करना
कुछ फ़ाइलें PSR मानक के अनुरूप नहीं हो सकतीं और स्वचालित लोड नहीं होतीं। आप `config/autoload.php` कॉन्फ़िगर करके इन्हें लोड कर सकते हैं, उदाहरण:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **ध्यान दें**
 > `autoload.php` में `support/Request.php` और `support/Response.php` लोड करने की सेटिंग है क्योंकि `vendor/workerman/webman-framework/src/support/` में भी ऐसे ही फ़ाइलें हैं। `autoload.php` से प्रोजेक्ट रूट की फ़ाइलें पहले लोड होती हैं, जिससे `vendor` फ़ाइलें बदले बिना इन दोनों को कस्टमाइज़ किया जा सकता है। कस्टमाइज़ की ज़रूरत न हो तो ये दो कॉन्फ़िगरेशन छोड़ सकते हैं।
