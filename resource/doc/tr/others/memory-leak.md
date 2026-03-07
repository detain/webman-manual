# Bellek sızıntısı hakkında
Webman kalıcı bellek çerçevesidir, bu yüzden bellek sızıntısına biraz dikkat etmemiz gerekiyor. Ancak geliştiriciler endişelenmemeli çünkü bellek sızıntıları yalnızca çok aşırı koşullarda oluşur ve kolayca önlenir. Webman ile geliştirme deneyimi geleneksel çerçevelerle temelde aynıdır; ek bellek yönetimi işlemleri gerekmez.

> **İpucu**
> Webman'in yerleşik monitor süreci tüm süreçlerin bellek kullanımını izler. Bir sürecin bellek kullanımı php.ini'deki `memory_limit` değerine yaklaştığında, ilgili süreç belleği serbest bırakmak için otomatik olarak güvenli şekilde yeniden başlatılır; uygulama etkilenmez.

## Bellek sızıntısı tanımı
Webman'in bellek kullanımının istek sayısıyla birlikte artması normaldir. Genel olarak, bir süreç belirli bir istek hacmine (genellikle milyonlarca) ulaştığında bellek sabitlenir veya yalnızca ara sıra hafifçe artar.

Çoğu uygulamada tek bir sürecin bellek kullanımı sonunda yaklaşık 10M–100M civarında sabitlenir. Tek süreç belleği 100M'yi geçmediği sürece endişelenmeye gerek yoktur.

Ayrıca büyük dosyalar, büyük istekler işlenirken veya veritabanından büyük miktarda veri okunurken PHP önemli bellek ayırır. PHP bu belleğin bir kısmını yeniden kullanmak için tutabilir ve tamamını işletim sistemine iade etmeyebilir; bu da yüksek bellek kullanımına yol açabilir. Bellek yeniden kullanıldığından endişelenmeye gerek yoktur.

> **İpucu**
> phar veya ikili paketlenmiş projelerde, paket boyutu büyükse bellek kullanımının 100M'yi aşması normaldir.

## Bellek sızıntısı nasıl onaylanır
Bir süreç bir milyondan fazla istek işlediyse, bellek kullanımı 100M'yi aşıyorsa ve her istekten sonra bellek hâlâ artıyorsa bellek sızıntısı olabilir.

## Bellek sızıntısı nasıl bulunur
Basit bir yaklaşım her API'yi stres testine tabi tutup milyonlarca istekten sonra bellek kullanımını artırmaya devam eden API'yi tespit etmektir.

Sorunlu API bulunduktan sonra ikili arama yöntemi kullanılabilir: sorunu tespit edene kadar her seferinde iş mantığı kodunun yarısı yorum satırına alınır.

## Bellek sızıntısı nasıl oluşur
**Bellek sızıntısı yalnızca aşağıdaki iki koşulun birlikte karşılanması durumunda oluşur:**
1. **Uzun ömürlü** bir dizi vardır (sıradan diziler sorun değildir)
2. Bu **uzun ömürlü** dizi sınırsız genişler (uygulama sürekli veri ekler, hiç temizlemez)

Yalnızca **her iki** koşul da karşılandığında sızıntı oluşur. Koşullardan biri eksikse veya yalnızca biri karşılanıyorsa sızıntı yoktur.

## Uzun ömürlü diziler

Webman'de uzun ömürlü diziler şunlardır:
1. `static` anahtar kelimesiyle tanımlanan diziler
2. Singleton'ların dizi özellikleri
3. `global` anahtar kelimesiyle tanımlanan diziler

> **Not**
> Webman uzun ömürlü veri kullanımına izin verir, ancak verinin sınırlı olduğundan ve öğe sayısının sınırsız genişlemeyeceğinden emin olunmalıdır.

Aşağıda her durum için örnekler verilmiştir.

### Sınırsız genişleyen static dizi
```php
class Foo
{
    public static $data = [];
    public function index(Request $request)
    {
        self::$data[] = time();
        return response('hello');
    }
}
```

`static` anahtar kelimesiyle tanımlanan `$data` dizisi uzun ömürlü bir dizidir. Örnekte `$data` her istekle birlikte genişlemeye devam ederek bellek sızıntısına neden olur.

### Sınırsız genişleyen singleton dizi özelliği
```php
class Cache
{
    protected static $instance;
    public $data = [];
    
    public function instance()
    {
        if (!self::$instance) {
            self::$instance = new self;
        }
        return self::$instance;
    }
    
    public function set($key, $value)
    {
        $this->data[$key] = $value;
    }
}
```

Çağrı kodu
```php
class Foo
{
    public function index(Request $request)
    {
        Cache::instance()->set(time(), time());
        return response('hello');
    }
}
```

`Cache::instance()` uzun ömürlü bir Cache singleton'ı döndürür. `$data` özelliği `static` anahtar kelimesi kullanmasa da sınıfın kendisi uzun ömürlü olduğundan `$data` de uzun ömürlü bir dizidir. `$data` dizisine sürekli farklı anahtarlarla veri eklendikçe programın bellek kullanımı artar ve sızıntı oluşur.

> **Not**
> `Cache::instance()->set(key, value)` ile eklenen anahtarlar sınırlı sayıdaysa sızıntı olmaz çünkü `$data` dizisi sınırsız genişlemez.

### Sınırsız genişleyen global dizi
```php
class Index
{
    public function index(Request $request)
    {
        global $data;
        $data[] = time();
        return response($foo->sayHello());
    }
}
```
`global` anahtar kelimesiyle tanımlanan diziler işlev veya sınıf yöntemi tamamlandıktan sonra serbest bırakılmadığından uzun ömürlüdür. Yukarıdaki kod istekler arttıkça bellek sızıntısına neden olur. Benzer şekilde işlev veya yöntem içinde `static` anahtar kelimesiyle tanımlanan diziler de uzun ömürlüdür; bu tür bir dizi sınırsız genişlerse sızıntı oluşur, örneğin:
```php
class Index
{
    public function index(Request $request)
    {
        static $data = [];
        $data[] = time();
        return response($foo->sayHello());
    }
}
```

## Öneriler
Geliştiricilere bellek sızıntısına aşırı odaklanmamaları önerilir çünkü nadiren ortaya çıkar. Ortaya çıkarsa stres testi yaparak sızıntıya neden olan kodu bulmak mümkündür. Geliştirici sızıntı noktasını bulamasa bile webman'in yerleşik monitor hizmeti uygun zamanda etkilenen süreci güvenli şekilde yeniden başlatarak belleği serbest bırakır.

Bellek sızıntısını mümkün olduğunca önlemek isterseniz şu önerileri dikkate alabilirsiniz:
1. `global` veya `static` anahtar kelimesiyle diziler kullanmaktan mümkün olduğunca kaçının; kullanırsanız sınırsız genişlemediğinden emin olun.
2. Bilmediğiniz sınıflar için singleton yerine `new` anahtar kelimesiyle başlatmayı tercih edin. Singleton gerekiyorsa sınırsız genişleyen dizi özelliği olup olmadığını kontrol edin.
