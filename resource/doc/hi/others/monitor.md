# प्रक्रिया मॉनिटरिंग
webman में एक अंतर्निहित मॉनिटर प्रक्रिया है जो दो सुविधाएँ प्रदान करती है:
1. फ़ाइल अपडेट की मॉनिटरिंग करती है और नया व्यावसायिक कोड स्वचालित रूप से रीलोड करती है (आमतौर पर विकास के दौरान उपयोग)।
2. सभी प्रक्रियाओं की मेमोरी खपत की मॉनिटरिंग करती है; यदि कोई प्रक्रिया `php.ini` में `memory_limit` सीमा के करीब पहुँच रही है, तो उस प्रक्रिया को स्वचालित रूप से सुरक्षित रूप से पुनः आरंभ करती है (व्यवसाय पर असर नहीं)।

## मॉनिटरिंग कॉन्फ़िगरेशन
`config/process.php` में `monitor` कॉन्फ़िगरेशन:
```php

global $argv;

return [
    // फ़ाइल अपडेट का पता लगाना और स्वचालित रीलोड
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // इन निर्देशिकाओं को मॉनिटर करें
            'monitorDir' => array_merge([    // किन निर्देशिकाओं की फ़ाइलों को मॉनिटर करना है
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // इन एक्सटेंशन वाली फ़ाइलें मॉनिटर की जाएँगी
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // फ़ाइल मॉनिटरिंग सक्षम करें
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // मेमोरी मॉनिटरिंग सक्षम करें
            ]
        ]
    ]
];
```
`monitorDir` अपडेट के लिए किन निर्देशिकाओं को मॉनिटर करना है यह कॉन्फ़िगर करता है (मॉनिटर की गई निर्देशिकाओं में फ़ाइलें अधिक न हों)।
`monitorExtensions` `monitorDir` निर्देशिकाओं में किन फ़ाइल एक्सटेंशन को मॉनिटर करना है यह कॉन्फ़िगर करता है।
`options.enable_file_monitor` जब `true` हो तो फ़ाइल अपडेट मॉनिटरिंग सक्षम होती है (Linux पर डिबग मोड में चलाने पर डिफ़ॉल्ट रूप से सक्षम)।
`options.enable_memory_monitor` जब `true` हो तो मेमोरी मॉनिटरिंग सक्षम होती है (Windows पर समर्थित नहीं है)।

> **सुझाव**
> Windows पर, फ़ाइल अपडेट मॉनिटरिंग केवल `windows.bat` या `php windows.php` चलाने पर सक्षम होती है।
