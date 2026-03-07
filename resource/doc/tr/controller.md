# Denetleyiciler

Yeni bir denetleyici dosyası oluşturun: `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

`http://127.0.0.1:8787/foo` adresine erişildiğinde sayfa `hello index` döndürür.

`http://127.0.0.1:8787/foo/hello` adresine erişildiğinde sayfa `hello webman` döndürür.

Rota kurallarını rota yapılandırması üzerinden değiştirebilirsiniz, bkz. [Rota](route.md).

> **İpucu**
> 404 hatası alırsanız `config/app.php` dosyasını açın, `controller_suffix` değerini `Controller` yapın ve yeniden başlatın.

## Denetleyici soneki
webman 1.3 sürümünden itibaren `config/app.php` içinde denetleyici soneki ayarlanabilir. `controller_suffix` boş `''` olarak ayarlanırsa denetleyici şöyle olur:

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Sonekin `Controller` olarak ayarlanması önerilir; denetleyici ile model sınıf adlarının çakışması önlenir ve güvenlik artar.

## Açıklama
- Çerçeve otomatik olarak `support\Request` nesnesini denetleyiciye iletir; bu sayede kullanıcı girdisi (get, post, header, cookie vb.) alınabilir, bkz. [İstek](request.md).
- Denetleyici sayı, dize veya `support\Response` nesnesi döndürebilir, diğer veri türleri döndürülemez.
- `support\Response` nesneleri `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` vb. yardımcı fonksiyonlarla oluşturulur.

## Denetleyici parametre bağlama

#### Örnek
webman, istek parametrelerinin denetleyici metod parametrelerine otomatik bağlanmasını destekler. Örnek:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

`name` ve `age` değerleri `GET` veya `POST` ile veya rota parametreleriyle geçirilebilir. Örnek:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

Öncelik sırası: `rota parametreleri` > `GET` > `POST`.

#### Varsayılan değerler

`/user/create?name=tom` adresine gidildiğinde şu hata alınır:

```html
Missing input parameter age
```

Nedeni `age` parametresinin geçirilmemesidir. Parametreye varsayılan değer vererek çözülebilir. Örnek:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### Parametre tipleri
`/user/create?name=tom&age=not_int` adresine gidildiğinde şu hata alınır:

> **İpucu**
> Test kolaylığı için parametreleri tarayıcı adres çubuğundan geçiriyoruz. Gerçek geliştirmede parametreler `POST` ile geçirilmelidir.

```html
Input age must be of type int, string given
```

Gelen veriler tipe göre dönüştürülür. Dönüştürülemezse `support\exception\InputTypeException` fırlatılır. `age` `int`e dönüştürülemediği için bu hata oluşur.

#### Özel hata mesajları
`Missing input parameter age` ve `Input age must be of type int, string given` gibi mesajlar çeviri ile özelleştirilebilir. Şu komutlara bakın:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => ':parameter parametresi :exceptType türünde olmalıdır, gönderilen tür :actualType',
    'Missing input parameter :parameter' => ':parameter girdi parametresi eksik',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Diğer tipler
webman `int`, `float`, `string`, `bool`, `array`, `object` ve `sınıf örnekleri` gibi parametre tiplerini destekler. Örnek:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

`/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` adresine gidildiğinde:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### Sınıf örneği
webman, tip ipuçlarıyla sınıf örneklerinin geçirilmesini destekler. Örnek:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

`/blog/create?blog[title]=hello&blog[content]=world` adresine gidildiğinde:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Model örneği

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // Ön uçtan güvensiz alanların geçirilmesini önlemek için burada doldurulabilir alanlar eklenmelidir
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

`/user/create?user[name]=tom&user[age]=18` adresine gidildiğinde benzeri bir sonuç alınır:

```json
1
```

## Denetleyici yaşam döngüsü

`config/app.php` içinde `controller_reuse` `false` olduğunda her istek için bir kez denetleyici örneği oluşturulur, istek bitince örnek serbest bırakılır. Geleneksel çerçevelerle aynı davranış.

`controller_reuse` `true` olduğunda tüm istekler aynı denetleyici örneğini kullanır; örnek oluşturulduktan sonra bellekte kalır.

> **Dikkat**
> Yeniden kullanım açıkken isteklerin denetleyici özelliklerini değiştirmemesi gerekir; değişiklik sonraki istekleri etkiler. Örnek:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // İlk update?id=1 isteğinden sonra model tutulur
        // delete?id=2 isteğinde id=1 verisi silinir
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **İpucu**
> Denetleyicinin `__construct()` yapıcısında veri döndürmenin etkisi olmaz. Örnek:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // Yapıcıda return etmenin etkisi yok, tarayıcı bu yanıtı almaz
        return response('hello'); 
    }
}
```

## Yeniden kullanım ve kullanmama arasındaki fark

#### Kullanmama
Her istek için yeni bir denetleyici örneği oluşturulur, istek bitince serbest bırakılır. Geleneksel çerçevelere benzer, çoğu geliştiricinin alışkanlığına uyar. Örnekler sürekli oluşturulup silindiği için performans biraz daha düşüktür (helloworld testinde ~%10, gerçek uygulamada genelde ihmal edilebilir).

#### Yeniden kullanma
Her süreç için sadece bir kez denetleyici oluşturulur, istek bitince örnek bırakılmaz. Sonraki istekler aynı örneği kullanır. Performans daha iyidir ama çoğu geliştiricinin alışkanlığına uymaz.

#### Yeniden kullanılamayan durumlar

İstek denetleyici özelliklerini değiştiriyorsa yeniden kullanım açılmamalıdır; değişiklikler sonraki istekleri etkiler.

Bazı geliştiriciler her istek için denetleyicinin `__construct()` yapıcısında başlatma yapar. Bu durumda yeniden kullanım mümkün değildir; yapıcı süreç başına bir kez çağrılır, her istekte değil.
