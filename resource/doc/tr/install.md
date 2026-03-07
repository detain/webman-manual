# webman nasıl kurulur

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: PHP + Composer ortamı kurulumu (zaten varsa atlayın)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Not**
> Yukarıdaki komut Linux ve macOS içindir. Windows kullanıcıları PHP'yi ayrıca kurmalıdır.

webman tarafından sunulan [statik PHP sürümünü](https://www.workerman.net/download) manuel olarak indirip açarak da kullanabilirsiniz.

## 1. Proje oluşturma

```php
composer create-project workerman/webman:~2.0
```

> **İpucu**
> Hata alırsanız, sorunlu bir Composer aynası kullanıyor olabilirsiniz. Proksi kaldırmak için `composer config -g --unset repos.packagist` komutunu çalıştırın.

## 2. Çalıştırma

Webman dizinine girin

#### Windows kullanıcıları
Başlatmak için `windows.bat` dosyasına çift tıklayın veya `php windows.php` çalıştırın

> **İpucu**
> Hata alırsanız, işlevler devre dışı bırakılmış olabilir. Devre dışı bırakmayı kaldırmak için [İşlev devre dışı bırakma kontrolü](others/disable-function-check.md) belgesine bakın.

#### Linux kullanıcıları
**Hata ayıklama modu** (geliştirme için: çıktı terminalde görünür; terminal kapatıldığında hizmet durur)

```php
php start.php start
```

**Daemon modu** (üretim için: terminalde çıktı görünmez; terminal kapatıldıktan sonra hizmet devam eder)

```php
php start.php start -d
```

#### Docker kullanıcıları

Tüm hizmetleri başlatıp konsola bağlayın
```php
docker-compose up
```

Hizmetleri arka planda çalıştırın
```php
docker-compose up -d
```

> **İpucu**
> Hata alırsanız, işlevler devre dışı bırakılmış olabilir. Devre dışı bırakmayı kaldırmak için [İşlev devre dışı bırakma kontrolü](others/disable-function-check.md) belgesine bakın.

## 3. Erişim

Tarayıcıda `http://ip-adresi:8787` adresine gidin.
