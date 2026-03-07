# think-cache

think-cache, thinkphp çerçevesinden çıkarılmış bir bileşendir ve bağlantı havuzu desteği eklenmiştir. Hem korutin hem de korutin-olmayan ortamları otomatik olarak destekler.

## Kurulum
`composer require -W webman/think-cache`

Kurulumdan sonra yeniden başlatma (restart) gereklidir (reload etkisizdir)

### Yapılandırma dosyası

Yapılandırma dosyası `config/think-cache.php` dosyasıdır

## Kullanım

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
  class UserController
  {
      public function db(Request $request)
      {
          $key = 'test_key';
          Cache::set($key, rand());
          return response(Cache::get($key));
      }
  }
  ```
## Sağlanan API
```php
// Önbellek ayarla
Cache::set('val','value',600);
// Önbelleğin var olup olmadığını kontrol et
Cache::has('val');
// Önbelleği al
Cache::get('val');
// Önbelleği sil
Cache::delete('val');
// Önbelleği temizle
Cache::clear();
// Oku ve önbelleği sil
Cache::pull('val');
// Yoksa yaz
Cache::remember('val',10);

// Sayısal önbellek verileri için
// Önbelleği 1 artır
Cache::inc('val');
// Önbelleği 5 artır
Cache::inc('val',5);
// Önbelleği 1 azalt
Cache::dec('val');
// Önbelleği 5 azalt
Cache::dec('val',5);

// Önbellek etiketleri kullan
Cache::tag('tag_name')->set('val','value',600);
// Belirli bir etiket altındaki önbelleği sil
Cache::tag('tag_name')->clear();
// Birden fazla etiket destekler
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Birden fazla etiket altındaki önbelleği sil
Cache::tag(['tag1','tag2'])->clear();

// Farklı önbellek depoları kullan
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


