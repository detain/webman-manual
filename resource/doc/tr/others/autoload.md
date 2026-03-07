# Otomatik Yükleme

## Composer ile PSR-0 Uyumlu Dosyaları Yükleme
webman `PSR-4` otomatik yükleme standardına uyar. Projenizin PSR-0 uyumlu kütüphaneleri yüklemesi gerekiyorsa şu adımları izleyin:

- PSR-0 kütüphanelerini saklamak için `extend` dizini oluşturun
- `composer.json` dosyasını düzenleyip `autoload` altına şunları ekleyin:

```json
"psr-0" : {
    "": "extend/"
}
```
Sonuç aşağıdakine benzer olacaktır:
![](../../assets/img/psr0.png)

- `composer dumpautoload` komutunu çalıştırın
- webman'i yeniden başlatmak için `php start.php restart` çalıştırın (not: değişikliklerin geçerli olması için yeniden başlatma zorunludur)

## Composer ile Belirli Dosyaları Yükleme

- `composer.json` dosyasını düzenleyip `autoload.files` altına yüklenecek dosyaları ekleyin:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` komutunu çalıştırın
- webman'i yeniden başlatmak için `php start.php restart` çalıştırın (not: değişikliklerin geçerli olması için yeniden başlatma zorunludur)

> **Not**
> composer.json içindeki `autoload.files` ile tanımlanan dosyalar webman başlamadan önce yüklenir. Framework'ün `config/autoload.php` ile yüklenen dosyalar ise webman başladıktan sonra yüklenir.
> composer.json'deki `autoload.files` dosyalarında yapılan değişiklikler için restart gerekir; reload yetmez. Framework'ün `config/autoload.php` ile yüklenen dosyalar hot-reload destekler; değişiklikler reload ile geçerli olur.

## Framework ile Belirli Dosyaları Yükleme
Bazı dosyalar PSR standardına uymayabilir ve otomatik yüklenemez. `config/autoload.php` dosyasını yapılandırarak bu dosyaları yükleyebilirsiniz, örneğin:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Not**
 > `autoload.php` içinde `support/Request.php` ve `support/Response.php` yüklemesi ayarlanmıştır, çünkü `vendor/workerman/webman-framework/src/support/` dizininde de aynı isimde dosyalar vardır. `autoload.php` ile proje kök dizinindeki sürümler öncelikli yüklenir; böylece `vendor` içindekileri değiştirmeden bu iki dosyayı özelleştirebilirsiniz. Özelleştirmeniz gerekmiyorsa bu iki ayarı atlayabilirsiniz.
