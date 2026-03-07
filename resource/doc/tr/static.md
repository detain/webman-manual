# webman Statik Dosya İşleme

webman statik dosya erişimini destekler. Statik dosyalar `public` dizinine yerleştirilir. Örneğin `http://127.0.0.8787/upload/avatar.png` adresine yapılan istek aslında `{ana_proje_dizini}/public/upload/avatar.png` dosyasına erişim sağlar.

> **Not**
> `/app/xx/dosya_adı` ile başlayan statik dosya erişimi aslında uygulama eklentisinin `public` dizinine erişim sağlar. Başka bir deyişle, `{ana_proje_dizini}/public/app/` altındaki dizinlere erişim desteklenmemektedir.
> Daha fazlası için [Uygulama Eklentileri](./plugin/app.md)'ne bakınız.

## Statik Dosya Desteğini Kapatma

Statik dosya desteğine ihtiyaç duyulmuyorsa, `config/static.php` dosyasını açarak `enable` seçeneğini `false` olarak değiştirin. Kapatıldığında tüm statik dosya erişimi 404 hatası döndürecektir.

## Statik Dosya Dizini Değiştirme

webman varsayılan olarak statik dosyalar için `public` dizinini kullanır. Değiştirmek istiyorsanız, `support/helpers.php` dosyasındaki `public_path()` yardımcı işlevini düzenleyin.

## Statik Dosya Ara Yazılımı

webman, `app/middleware/StaticFile.php` konumunda bir statik dosya ara yazılımı ile birlikte gelir.
Bazen statik dosyalar üzerinde bazı işlemler yapmamız gerekebilir (örneğin çapraz kaynak HTTP başlığı eklemek veya nokta (`.`) ile başlayan dosyalara erişimi engellemek). Bu ara yazılım bu tür işlemler için kullanılabilir.

`app/middleware/StaticFile.php` dosyası aşağıdaki gibi olabilir:
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // Nokta ile başlayan gizli dosyalara erişimi engelle
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // Çapraz kaynak (CORS) başlığı ekle
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
Bu ara yazılıma ihtiyaç duyulduğunda `config/static.php` dosyasındaki `middleware` seçeneğini etkinleştirmeniz gerekir.
