# निर्देशिका संरचना

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

एक एप्लिकेशन प्लगइन में webman के समान निर्देशिका संरचना और कॉन्फ़िगरेशन फ़ाइलें होती हैं। व्यवहार में, विकास अनुभव सामान्य webman एप्लिकेशन विकसित करने के समान ही है।

प्लगइन निर्देशिका और नामकरण PSR-4 निर्दिष्टि का पालन करते हैं। चूँकि सभी प्लगइन `plugin` निर्देशिका में रखे जाते हैं, नेमस्पेस `plugin` से शुरू होते हैं, उदाहरण के लिए `plugin\foo\app\controller\UserController`।

## api निर्देशिका के बारे में

प्रत्येक प्लगइन में एक `api` निर्देशिका होती है। यदि आपका एप्लिकेशन अन्य एप्लिकेशन द्वारा कॉल किए जाने के लिए आंतरिक इंटरफ़ेस प्रदान करता है, तो उन इंटरफ़ेस को `api` निर्देशिका में रखें।

ध्यान दें: यहाँ उल्लिखित इंटरफ़ेस फ़ंक्शन कॉल इंटरफ़ेस हैं, नेटवर्क/HTTP इंटरफ़ेस नहीं।

उदाहरण के लिए, ईमेल प्लगइन `plugin/email/api/Email.php` में `Email::send()` इंटरफ़ेस प्रदान करता है ताकि अन्य एप्लिकेशन ईमेल भेजते समय कॉल कर सकें। इसके अलावा, `plugin/email/api/Install.php` webman-admin प्लगइन बाज़ार द्वारा इंस्टॉल या अनइंस्टॉल ऑपरेशन चलाने के लिए स्वचालित रूप से जनरेट होता है।
