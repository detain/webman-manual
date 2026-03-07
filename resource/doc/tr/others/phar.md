# phar Paketleme

phar, PHP'de JAR'a benzeyen bir paketleme dosyasıdır. webman projenizi tek bir phar dosyasına paketlemek için phar kullanabilirsiniz; bu da dağıtımı kolaylaştırır.

**Burada [fuzqing](https://github.com/fuzqing)'e çok teşekkür ederiz.**

> **Not**
> `php.ini` dosyasında phar yapılandırma seçeneklerini kapatmanız gerekmektedir; yani `phar.readonly = 0` olarak ayarlanmalıdır.

## Komut Satırı Aracı Kurulumu
`composer require webman/console`

## Paketleme
webman proje kök dizininde `php webman build:phar` komutunu çalıştırın. `build` dizininde bir `webman.phar` dosyası oluşturulacaktır.

> Paketleme ile ilgili yapılandırmalar `config/plugin/webman/console/app.php` dosyasındadır.

## Başlatma ve Durdurma İlgili Komutlar
**Başlatma**
`php webman.phar start` veya `php webman.phar start -d`

**Durdurma**
`php webman.phar stop`

**Durumu Görüntüleme**
`php webman.phar status`

**Bağlantı Durumunu Görüntüleme**
`php webman.phar connections`

**Yeniden Başlatma**
`php webman.phar restart` veya `php webman.phar restart -d`

## Açıklama
* Paketlenmiş projeler reload desteklemez; kodu güncellemek için yeniden başlatma gereklidir.

* Paket boyutunun aşırı büyümesini ve bellek kullanımını önlemek için, `config/plugin/webman/console/app.php` dosyasındaki `exclude_pattern` ve `exclude_files` seçeneklerini ayarlayarak gereksiz dosyaları hariç tutabilirsiniz.

* webman.phar çalıştırıldıktan sonra, webman.phar dosyasının bulunduğu dizinde günlük dosyaları gibi geçici dosyaları depolamak için `runtime` dizini oluşturulur.

* Projenizde .env dosyası kullanıyorsanız, .env dosyasını webman.phar dosyasının bulunduğu dizine koymalısınız.

* Kullanıcıların yüklediği dosyaları asla Phar paketinin içinde depolamayın; `phar://` protokolü ile kullanıcı yüklemeleri üzerinde işlem yapmak çok tehlikelidir (Phar serileştirme kaldırma güvenlik açığı). Kullanıcı yüklemeleri Phar paketinin dışında ayrı olarak diskte depolanmalıdır. Aşağıya bakın.

* İşletmenizin public dizinine dosya yüklemesi yapmanız gerekiyorsa, public dizinini webman.phar dosyasının bulunduğu dizine çıkarmanız gerekir. Bu durumda `config/app.php` dosyasını yapılandırmanız gerekmektedir.
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
İşletme, gerçek public dizini konumunu bulmak için `public_path($göreli_yol)` yardımcı fonksiyonunu kullanabilir.

* webman.phar, Windows'ta özel işlemleri desteklemez.
