# phar पैकिंग

phar PHP में JAR के समान एक पैकेज फ़ाइल है। आप अपनी webman परियोजना को एक ही phar फ़ाइल में पैक करने के लिए phar का उपयोग कर सकते हैं, ताकि इसे आसानी से डिप्लॉय किया जा सके।

**यहां [fuzqing](https://github.com/fuzqing) के PR का बहुत आभार है।**

> **ध्यान दें**
> `php.ini` में phar कॉन्फ़िगरेशन विकल्प को बंद करने की आवश्यकता है, अर्थात `phar.readonly = 0` सेट करें।

## कमांड लाइन उपकरण स्थापित करें
`composer require webman/console`

## पैकिंग
webman परियोजना रूट डायरेक्टरी में `php webman build:phar` कमांड चलाएं। यह `build` डायरेक्टरी में एक `webman.phar` फ़ाइल उत्पन्न करेगा।

> पैकिंग संबंधित सेटिंग `config/plugin/webman/console/app.php` में है।

## स्टार्ट और स्टॉप संबंधित कमांड
**स्टार्ट**
`php webman.phar start` या `php webman.phar start -d`

**स्टॉप**
`php webman.phar stop`

**स्टेटस देखें**
`php webman.phar status`

**कनेक्शन स्टेटस देखें**
`php webman.phar connections`

**रीस्टार्ट**
`php webman.phar restart` या `php webman.phar restart -d`

## विवरण
* पैक किए गए परियोजना reload का समर्थन नहीं करती; कोड अपडेट के लिए रीस्टार्ट आवश्यक है।

* पैक के आकार और मेमोरी उपयोग को कम करने के लिए, `config/plugin/webman/console/app.php` में `exclude_pattern` और `exclude_files` विकल्प सेट करके अप्रयोजनीय फ़ाइलों को बाहर किया जा सकता है।

* webman.phar चलाने के बाद webman.phar के फ़ोल्डर में `runtime` फ़ोल्डर उत्पन्न होगा, जिसमें लॉग आदि अस्थायी फ़ाइलें रखी जाएंगी।

* अगर आपके परियोजना में .env फ़ाइल का उपयोग है, तो .env फ़ाइल को webman.phar फ़ोल्डर में रखना चाहिए।

* कभी भी उपयोगकर्ता द्वारा अपलोड की गई फ़ाइलों को phar पैकेज के अंदर संग्रहीत न करें, क्योंकि `phar://` प्रोटोकॉल से उपयोगकर्ता फ़ाइलों पर काम करना बहुत खतरनाक है (phar deserialization कमजोरी)। उपयोगकर्ता अपलोड फ़ाइलें phar पैकेज के बाहर डिस्क पर अलग से संग्रहीत होनी चाहिए। नीचे देखें।

* अगर आपके बिजनेस में public फ़ोल्डर में फ़ाइल अपलोड करने की आवश्यकता है, तो आपको public फ़ोल्डर को अलग से webman.phar फ़ोल्डर में रखना होगा, इसके बाद `config/app.php` को कॉन्फ़िगर करने की आवश्यकता होगी।
```php
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
`public_path($सापेक्ष_पथ)` हेल्पर फ़ंक्शन का इस्तेमाल करके वास्तविक public फ़ोल्डर की स्थिति पता कर सकता है।

* webman.phar विंडोज़ पर कस्टम प्रोसेस का समर्थन नहीं करता है।
