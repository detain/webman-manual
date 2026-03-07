# İşlem izleme
webman, iki işlevi destekleyen yerleşik bir izleme süreciyle gelir:
1. Dosya güncellemelerini izler ve yeni iş kodunu otomatik olarak yeniden yükler (genellikle geliştirme sırasında kullanılır)
2. Tüm işlemlerin bellek kullanımını izler; bir işlem `php.ini` içindeki `memory_limit` sınırını aşmak üzereyse, o işlemi otomatik olarak güvenli şekilde yeniden başlatır (iş sürekliliğini etkilemez)

## İzleme yapılandırması
`config/process.php` dosyasındaki `monitor` yapılandırması:
```php

global $argv;

return [
    // Dosya güncellemesi algılama ve otomatik yeniden yükleme
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // Bu dizinleri izle
            'monitorDir' => array_merge([    // Hangi dizinlerin dosyaları izlenmeli
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Bu uzantılara sahip dosyalar izlenecek
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // Dosya izlemeyi etkinleştir
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // Bellek izlemeyi etkinleştir
            ]
        ]
    ]
];
```
`monitorDir` güncellemeler için hangi dizinlerin izleneceğini yapılandırır (izlenen dizinlerde çok fazla dosya olmamalıdır)
`monitorExtensions`, `monitorDir` dizinlerinde hangi dosya uzantılarının izleneceğini yapılandırır
`options.enable_file_monitor` `true` olduğunda dosya güncelleme izleme etkinleşir (Linux'ta debug modunda çalıştırıldığında varsayılan olarak etkindir)
`options.enable_memory_monitor` `true` olduğunda bellek izleme etkinleşir (Windows'ta desteklenmez)

> **İpucu**
> Windows'ta dosya güncelleme izlemesi yalnızca `windows.bat` veya `php windows.php` çalıştırıldığında etkinleşir
