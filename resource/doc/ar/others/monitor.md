# مراقبة العملية
يأتي webman مزوداً بعملية مراقبة مدمجة تدعم وظيفتين:
1. مراقبة تحديثات الملفات وإعادة تحميل كود الأعمال الجديد تلقائياً (تُستخدم عادةً أثناء التطوير).
2. مراقبة استهلاك الذاكرة لجميع العمليات؛ إذا كانت عملية ما على وشك تجاوز حد `memory_limit` في `php.ini`، يتم إعادة تشغيلها تلقائياً بشكل آمن (دون التأثير على الأعمال).

## تكوين المراقبة
تكوين `monitor` في `config/process.php`:
```php

global $argv;

return [
    // كشف تحديث الملف وإعادة التحميل التلقائي
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // مراقبة هذه المجلدات
            'monitorDir' => array_merge([    // المجلدات التي يجب مراقبة ملفاتها
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // الملفات ذات هذه الامتدادات ستُراقَب
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // تفعيل مراقبة الملفات
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // تفعيل مراقبة الذاكرة
            ]
        ]
    ]
];
```
`monitorDir` يحدد المجلدات التي تُراقَب للتحديثات (يفضل ألا يكون عدد الملفات كبيراً).
`monitorExtensions` يحدد امتدادات الملفات التي تُراقَب في مجلدات `monitorDir`.
عندما تكون `options.enable_file_monitor` بقيمة `true`، تُفعّل مراقبة تحديثات الملفات (تكون مفعّلة افتراضياً في وضع التصحيح على Linux).
عندما تكون `options.enable_memory_monitor` بقيمة `true`، تُفعّل مراقبة الذاكرة (غير مدعومة على Windows).

> **تلميح**
> على Windows، تُفعّل مراقبة تحديثات الملفات فقط عند تشغيل `windows.bat` أو `php windows.php`.
