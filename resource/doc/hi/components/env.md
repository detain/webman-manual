# ENV कंपोनेंट vlucas/phpdotenv

## विवरण
`vlucas/phpdotenv` एक पर्यावरण चर लोडिंग कंपोनेंट है, जो विभिन्न वातावरणों (जैसे विकास, परीक्षण आदि) के कॉन्फ़िगरेशन को अलग करने के लिए उपयोग किया जाता है।

## परियोजना लिंक

https://github.com/vlucas/phpdotenv
  
## स्थापना
 
```php
composer require vlucas/phpdotenv
 ```
  
## उपयोग

### परियोजना रूट निर्देशिका में नई `.env` फ़ाइल बनाएं
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### कॉन्फ़िगरेशन फ़ाइल संशोधित करें
**config/database.php**
```php
return [
    // डिफ़ॉल्ट डेटाबेस
    'default' => 'mysql',

    // विभिन्न डेटाबेस कॉन्फ़िगरेशन
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **सुझाव**
> `.env` फ़ाइल को `.gitignore` सूची में जोड़ने की सिफारिश की जाती है ताकि उसे कोड रिपॉजिटरी में कमिट न करना पड़े। रिपॉजिटरी में `.env.example` कॉन्फ़िगरेशन नमूना फ़ाइल जोड़ें। परियोजना डिप्लॉय करते समय `.env.example` को `.env` के रूप में कॉपी करें और वर्तमान वातावरण के अनुसार `.env` में कॉन्फ़िगरेशन बदलें। इससे परियोजना विभिन्न वातावरणों में अलग-अलग कॉन्फ़िगरेशन लोड कर सकेगी।

> **ध्यान दें**
> `vlucas/phpdotenv` में PHP TS संस्करण (Thread Safe) में बग हो सकता है। कृपया NTS संस्करण (Non-Thread-Safe) का उपयोग करें। वर्तमान PHP संस्करण `php -v` चलाकर जांचा जा सकता है।

## अधिक जानकारी

https://github.com/vlucas/phpdotenv पर जाएं।
  
