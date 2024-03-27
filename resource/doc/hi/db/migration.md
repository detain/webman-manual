# माइग्रेशन डेटाबेस माइग्रेशन उपकरण Phinx

## विवरण

Phinx विकासको लागि डेटाबेस सोधपुरी र राख्न हल्ला गर्न सजिलो बनाउँछ। यो मानव रचित SQL प्रणाली हटाउँछ, र डाटाबेस माइग्रेसन प्रबन्ध गर्न शक्तिशाली PHP API प्रयोग गर्दछ। विकासकहरूले आफूलाई व्यावसायिक प्रबन्ध चलान बनाउन सक्छन्। Phinx विभिन्न डेटाबेसहरूबीच डेटाबेस माइग्रेसन गर्न सजिलो बनाउँछ। यसले जाँच गर्छ कुन माइग्रेसन स्क्रिप्टहरू व्यावस्थित भएका छन्, जसलाई विकासकहरूले डाटाबेसको स्थिति को हिचकिचलाई नभएकोले बढावा गर्न सक्छन्।

## परियोजना पता

https://github.com/cakephp/phinx

## स्थापना

```php
composer require robmorgan/phinx
```

## आधिकारिक चिनी दस्तावेज पता

थप उपयोगको लागि, कृपया आधिकारिक चिनी दस्तावेज हेर्नुहोस्। यहाँले कम्तिमा webman मा कसरी कन्फिगर गर्ने भन्न बाँकी छ।

https://tsy12321.gitbooks.io/phinx-doc/content/

## माइग्रेसन फाइल डिरेक्टरी संरचना

``` 
.
├── app                           अनुप्रयोग डिरेक्टरी
│   ├── controller                नियन्त्रक डिरेक्टरी
│   │   └── Index.php             नियन्त्रक
│   ├── model                     मोडेल डिरेक्टरी
......
├── database                      डाटाबेस फाइलहरू
│   ├── migrations                माइग्रेसन फाइलहरू
│   │   └── 20180426073606_create_user_table.php
│   ├── seeds                     परीक्षण डाटा
│   │   └── UserSeeder.php
......
```

## phinx.php कन्फिगरेसन

परियोजना मूल डायरेक्टरीमा phinx.php फाइल बनाउनुहोस्

```php
<?php
return [
    "paths" => [
        "migrations" => "database/migrations",
        "seeds"      => "database/seeds"
    ],
    "environments" => [
        "default_migration_table" => "phinxlog",
        "default_database"        => "dev",
        "default_environment"     => "dev",
        "dev" => [
            "adapter" => "DB_CONNECTION",
            "host"    => "DB_HOST",
            "name"    => "DB_DATABASE",
            "user"    => "DB_USERNAME",
            "pass"    => "DB_PASSWORD",
            "port"    => "DB_PORT",
            "charset" => "utf8"
        ]
    ]
];
```

## उपयोग सुझाव

जब एक प्लसकोड सम्मिलन गरिएको छ भने माइग्रेसन फाइलहरूलाई अधिक परिवर्तन गर्न अनुमति दिइने, समस्या आउनु अघि नयाँ परिवर्तन वा हटाउने कार्य गर्नुपर्छ।

#### डाटा तालिका सिर्जना कार्य फाइल नाम नियम

`{समय (स्वत: सिर्जना)}_create_{तालिका नाम अंग्रेजी लगाउनु}`

#### डाटा तालिका संशोधन कार्य फाइल नाम नियम

`{समय (स्वत: सिर्जना)}_modify_{तालिका नाम अंग्रेजी लगाउनु+निर्दिष्ट संशोधन अंग्रेजी लगाउनु}`

### डाटा तालिका हटाउन कार्य फाइल नाम नियम

`{समय (स्वत: सिर्जना)}_delete_{तालिका नाम अंग्रेजी लगाउनु+निर्दिष्ट संशोधन अंग्रेजी लगाउनु}`

### तालिका डाटा भर्ने कार्य फाइल नाम नियम

`{समय (स्वत: सिर्जना)}_fill_{तालिका नाम अंग्रेजी लगाउनु+निर्दिष्ट संशोधन अंग्रेजी लगाउनु}`