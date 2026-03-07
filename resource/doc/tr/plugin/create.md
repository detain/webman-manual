# Temel Eklenti Oluşturma ve Yayınlama Süreci

## İlke
1. Cross-origin eklentisi örneğiyle: eklenti üç parçadan oluşur – cross-origin middleware dosyası, `middleware.php` yapılandırma dosyası ve komutla otomatik oluşturulan `Install.php`.
2. Bu üç dosyayı paketleyip Composer’a yayınlamak için komut kullanılır.
3. Kullanıcı Composer ile cross-origin eklentisini kurduğunda, `Install.php` middleware ve yapılandırmayı `{ana proje}/config/plugin` içine kopyalar; webman yükleyip cross-origin’i etkinleştirir.
4. Kullanıcı Composer ile eklentiyi kaldırdığında, `Install.php` ilgili middleware ve yapılandırma dosyalarını siler; eklenti otomatik olarak kaldırılır.

## Kurallar
1. Eklenti adı iki bölümden oluşur: `vendor` ve `eklenti adı`, örn. `webman/push`, Composer paket adıyla eşleşir.
2. Eklenti yapılandırma dosyaları `config/plugin/vendor/eklenti adı/` altındadır (console komutu yapılandırma dizinini otomatik oluşturur). Eklenti yapılandırma gerektirmiyorsa otomatik oluşturulan dizin silinmelidir.
3. Eklenti yapılandırma dizini yalnızca şunları destekler: `app.php` (ana yapılandırma), `bootstrap.php` (süreç başlatma), `route.php` (rotalar), `middleware.php` (middleware), `process.php` (özel süreçler), `database.php` (veritabanı), `redis.php` (Redis), `thinkorm.php` (thinkorm). Bunlar webman tarafından otomatik tanınır.
4. Yapılandırmaya erişim: `config('plugin.vendor.eklenti adı.yapılandırma dosyası.öğe');`, örn. `config('plugin.webman.push.app.app_key')`.
5. Eklentinin kendi veritabanı yapılandırması varsa: `illuminate/database` için `Db::connection('plugin.vendor.eklenti adı.bağlantı')`, `thinkorm` için `Db::connect('plugin.vendor.eklenti adı.bağlantı')`.
6. Eklenti `app/` içine iş dosyaları koyacaksa ana proje ve diğer eklentilerle çakışmaması gerekir.
7. Eklentiler mümkün olduğunca ana projeye dosya veya dizin kopyalamamalıdır. Örn. cross-origin eklentisinde yalnızca yapılandırma kopyalanır; middleware dosyaları `vendor/webman/cros/src` içinde kalır.
8. Eklenti namespace’leri için PascalCase önerilir, örn. `Webman/Console`.

## Örnek

**`webman/console` komut satırını yükleyin**

`composer require webman/console`

### Eklenti oluştur

Oluşturulacak eklentinin adı `foo/admin` olsun (Composer üzerinden yayınlanacak proje adı, küçük harf olmalı). Çalıştırın:

`php webman plugin:create --name=foo/admin`

`vendor/foo/admin` (eklenti dosyaları) ve `config/plugin/foo/admin` (yapılandırma) oluşur.

> Not
> `config/plugin/foo/admin` şunları destekler: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. webman ile aynı biçim, otomatik birleştirme.
> Erişimde `plugin` öneki kullanın, örn. `config('plugin.foo.admin.app')`.


### Eklenti dışa aktar

Geliştirme bittikten sonra çalıştırın:

`php webman plugin:export --name=foo/admin`

Dışa aktar

> Açıklama
> Dışa aktarmada `config/plugin/foo/admin`, `vendor/foo/admin/src` içine kopyalanır ve `Install.php` oluşturulur. Kurulum/kaldırma sırasında `Install.php` çalışır.
> Varsayılan kurulum: `vendor/foo/admin/src` içindeki yapılandırmayı projenin `config/plugin` dizinine kopyalar.
> Varsayılan kaldırma: projenin `config/plugin` dizinindeki eklenti yapılandırma dosyalarını siler.
> Kurulum/kaldırma sırasında özel işlem için `Install.php` değiştirilebilir.

### Eklentiyi gönder
* [GitHub](https://github.com) ve [Packagist](https://packagist.org) hesabınız olduğunu varsayın.
* [GitHub](https://github.com)’da `admin` deposu oluşturun ve kodu push edin, örn. `https://github.com/kullanici-adiniz/admin`.
* `https://github.com/kullanici-adiniz/admin/releases/new` adresine gidip release oluşturun, örn. `v1.0.0`.
* [Packagist](https://packagist.org)’te `Submit` tıklayıp `https://github.com/kullanici-adiniz/admin` gönderin; eklenti yayınlanır.

> **İpucu**
> Packagist’te isim çakışması olursa başka bir vendor seçin, örn. `foo/admin`’ı `myfoo/admin` yapın.

Güncellemelerde: kodu GitHub’a push edin, `https://github.com/kullanici-adiniz/admin/releases/new` adresinde yeni bir release oluşturun, ardından `https://packagist.org/packages/foo/admin` sayfasında `Update` tıklayın.

## Eklentiye komut ekleme
Bazı eklentilerin özel komutlara ihtiyacı vardır. Örn. `webman/redis-queue` kurunca proje `redis-queue:consumer` komutunu alır. `php webman redis-queue:consumer send-mail` çalıştırmak `SendMail.php` consumer sınıfını hızlıca üretir; geliştirme kolaylaşır.

`foo/admin` eklentisine `foo-admin:add` komutunu eklemek için:

### Komut oluştur

**`vendor/foo/admin/src/FooAdminAddCommand.php` dosyasını oluşturun**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Komut açıklaması';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Not**
> Eklentiler arasında komut çakışmasını önlemek için `vendor-plugin:komut` biçimini kullanın. Örn. `foo/admin` eklentisinin tüm komutları `foo-admin:` önekine sahip olmalıdır, örn. `foo-admin:add`.

### Yapılandırma ekle
**`config/plugin/foo/admin/command.php` oluşturun**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Gerektiğinde daha fazla ekleyin...
];
```

> **İpucu**
> `command.php` eklentinin özel komutlarını kaydeder. Her eleman bir komut sınıfıdır. `webman/console` bunları otomatik yükler. Detay için [Konsol komutları](console.md).

### Dışa aktarmayı çalıştır
`php webman plugin:export --name=foo/admin` çalıştırarak eklentiyi dışa aktarıp Packagist’e yayınlayın. `foo/admin` kurulduktan sonra `foo-admin:add` komutu kullanılabilir olur. `php webman foo-admin:add jerry` çalıştırıldığında `Admin add jerry` yazdırılır.
