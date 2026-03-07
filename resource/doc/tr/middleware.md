# Arak Katmanı
Arak katmanı genellikle isteği veya yanıtı durdurmak için kullanılır. Örneğin, bir denetleyiciyi çalıştırmadan önce kullanıcı kimliğini doğrulamak için genel olarak kullanılır, kullanıcı giriş yapmamışsa giriş sayfasına yönlendirir veya isteğe belirli bir başlık ekler. Örneğin, belirli bir URI isteğinin yüzdesini hesaplar.

## Arak Katmanı Soğan Modeli

```                      
            ┌──────────────────────────────────────────────────────┐
            │                     arak_katmanı1                    │ 
            │     ┌──────────────────────────────────────────┐     │
            │     │               arak_katmanı2              │     │
            │     │     ┌──────────────────────────────┐     │     │
            │     │     │         arak_katmanı3        │     │     │        
            │     │     │     ┌──────────────────┐     │     │     │
            │     │     │     │                  │     │     │     │
   ── İstek ─────────────────────> Denetleyici ── Yanıt ───────────────────────────> İstemci
            │     │     │     │                  │     │     │     │
            │     │     │     └──────────────────┘     │     │     │
            │     │     │                              │     │     │
            │     │     └──────────────────────────────┘     │     │
            │     │                                          │     │
            │     └──────────────────────────────────────────┘     │
            │                                                      │
            └──────────────────────────────────────────────────────┘
```
Arak katmanı ve denetleyici, klasik bir soğan modeli oluşturur. Arak katmanı, birbirine geçmiş bir dizi soğan kabuğuna benzerken, denetleyici soğanın iç kısmını oluşturur. Yukarıdaki şekilde gösterildiği gibi istek, arak_katmanı1,2,3'ten geçerek denetleyiciye ulaşır, denetleyici bir yanıt döner ve ardından yanıt 3,2,1 sırasıyla arak katmanından geçerek nihayetinde istemciye döner. Yani her arak katmanında isteği alabilir ve yanıtı elde edebiliriz.

## İstek Engellemesi
Bazen isteğin denetleyici katmanına ulaşmasını istemeyebiliriz, örneğin middleware2'de geçerli bir kullanıcı oturumunun olmadığını tespit edersek, isteği doğrudan engelleyip bir giriş yanıtı döndürebiliriz. Bu durum aşağıdaki gibi olacaktır:

```                      
            ┌────────────────────────────────────────────────────────────┐
            │                         arak_katmanı1                      │ 
            │     ┌────────────────────────────────────────────────┐     │
            │     │                   arak_katmanı2                │     │
            │     │          ┌──────────────────────────────┐      │     │
            │     │          │        arak_katmanı3         │      │     │       
            │     │          │    ┌──────────────────┐      │      │     │
            │     │          │    │                  │      │      │     │
   ── İstek ────────────┐    │    │    Denetleyici   │      │      │     │
            │     │   Yanıt  │    │                  │      │      │     │
   <────────────────────┘    │    └──────────────────┘      │      │     │
            │     │          │                              │      │     │
            │     │          └──────────────────────────────┘      │     │
            │     │                                                │     │
            │     └────────────────────────────────────────────────┘     │
            │                                                            │
            └────────────────────────────────────────────────────────────┘
```

Yukarıdaki gibi istek arak_katmanı2'ye ulaştıktan sonra bir giriş yanıtı oluşturulur ve yanıt arak_katmanı1'e geçerek istemciye döner.
## Orta Yazılım Arabirimi
Orta yazılım, `Webman\MiddlewareInterface` arabirimini uygulamalıdır.
```php
interface MiddlewareInterface
{
    /**
     * Gelen sunucu isteğini işler.
     *
     * Bir yanıt üretmek için gelen sunucu isteğini işler.
     * Kendisi yanıtı üretemezse, sağlanan istek işleyiciye devretmek için kullanabilir.
     */
    public function process(Request $request, callable $handler): Response;
}
```
Yani, `process` yöntemini uygulamak zorunludur. `process` yöntemi, bir `support\Response` nesnesi dönmelidir. Bu nesne varsayılan olarak `$handler($request)` tarafından oluşturulur (isteğin soğan katmanında devam etmesine izin verir), isteğin durmasını veya diğer yardımcı işlevler tarafından oluşturulan yanıtı (örneğin `response()`, `json()`, `xml()`, `redirect()` vb.) döndürebilir (isteğin soğan katmanında durmasına neden olur).

## İstek ve Yanıtı Orta Yazılımda Almak
Orta yazılım içinde isteği alabilir ve denetleyici tarafından oluşturulan yanıtı alabiliriz, bu nedenle orta yazılım içinde üç bölüme ayrılır.
1. İstek geçişi aşaması, yani isteği işlemeden önceki aşama
2. Denetleyici tarafından isteğin işlenme aşaması
3. Yanıt çıkış aşaması, yani isteğin işlenmesinden sonraki aşama

Orta yazılım içindeki bu üç aşama şu şekilde gösterilir
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Test implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        echo 'Bu istek geçişi aşamasıdır, yani isteği işlemeden önce';

        $response = $handler($request); // Denetleyiciye ulaşana kadar devam eden istekler

        echo 'Bu yanıt çıkış aşamasıdır, yani isteği işledikten sonra';

        return $response;
    }
}
```

## Örnek: Kimlik Doğrulama Orta Yazılımı
`app/middleware/AuthCheckTest.php` dosyası oluşturun (dizin yoksa kendiniz oluşturun) aşağıdaki gibi:
```php
<?php
namespace app\middleware;

use ReflectionClass;
use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AuthCheckTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        if (session('user')) {
            // Giriş yapılmış, istek devam eder
            return $handler($request);
        }

        // Kontrolcü hangi yöntemlerin giriş yapmaya gerek olmadığını yansıtacak şekilde yansıyın
        $controller = new ReflectionClass($request->controller);
        $noNeedLogin = $controller->getDefaultProperties()['noNeedLogin'] ?? [];

        // Erişilmek istenen yöntem giriş gerektiriyor
        if (!in_array($request->action, $noNeedLogin)) {
            // İsteği engelle, yönlendirme yanıtı dönerek isteği durdur
            return redirect('/user/login');
        }

        // Giriş yapmaya gerek olmadığı için istek devam eder
        return $handler($request);
    }
}
```

Yeni kontrolcü oluşturun `app/controller/UserController.php`
```php
<?php
namespace app\controller;
use support\Request;

class UserController
{
    /**
     * Giriş yapmaya gerek olmayan yöntemler
     */
    protected $noNeedLogin = ['login'];

    public function login(Request $request)
    {
        $request->session()->set('user', ['id' => 10, 'name' => 'webman']);
        return json(['code' => 0, 'msg' => 'giriş başarılı']);
    }

    public function info()
    {
        return json(['code' => 0, 'msg' => 'başarılı', 'data' => session('user')]);
    }
}
```

> **Not**
> `$noNeedLogin` içerisinde, kullanıcı girişi yapmadan erişilebilecek yöntemler kaydedilmiştir.

`config/middleware.php` içine global bir orta yazılım ekleyin:
```php
return [
    // Global orta yazılımlar
    '' => [
        // ... Diğer orta yazılımlar buraya eklenir
        app\middleware\AuthCheckTest::class,
    ]
];
```

Kimlik doğrulama orta yazılımı sayesinde, kontrolcü katmanında giriş yapılıp yapılmadığını düşünmeden iş mantığı kodlarını yazabiliriz.

## Örnek: Cross-Origin Request (CORS) Orta Yazılımı
`app/middleware/AccessControlTest.php` dosyası oluşturun (dizin yoksa kendiniz oluşturun) aşağıdaki gibi:
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AccessControlTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        // Eğer options isteği ise boş bir yanıt döndür, aksi halde devam et ve bir yanıt al
        $response = $request->method() == 'OPTIONS' ? response('') : $handler($request);
        
        // Yanıta CORS ile ilgili HTTP başlıkları ekle
        $response->withHeaders([
            'Access-Control-Allow-Credentials' => 'true',
            'Access-Control-Allow-Origin' => $request->header('origin', '*'),
            'Access-Control-Allow-Methods' => $request->header('access-control-request-method', '*'),
            'Access-Control-Allow-Headers' => $request->header('access-control-request-headers', '*'),
        ]);
        
        return $response;
    }
}
```

> **İpucu**
> Cross-origin istekleri OPTIONS ön uçuş istekleri tetikleyebilir. OPTIONS isteklerinin denetleyiciye ulaşmasını istemiyoruz, bu yüzden onları engellemek için doğrudan boş bir yanıt döndürüyoruz (`response('')`).
> API'nizin yönlendirmeye ihtiyacı varsa, `Route::any(..)` veya `Route::add(['POST', 'OPTIONS'], ..)` kullanın.

`config/middleware.php` içine global bir orta yazılım ekleyin:
```php
return [
    // Global orta yazılımlar
    '' => [
        // ... Diğer orta yazılımlar buraya eklenir
        app\middleware\AccessControlTest::class,
    ]
];
```

> **Not**
> Eğer ajax isteği özel header'ları ayarladıysa, bu özel header'ı `Access-Control-Allow-Headers` alanına eklemeyi unutmayın. Aksi halde `Request header field XXXX is not allowed by Access-Control-Allow-Headers in preflight response.` hatası alabilirsiniz.
## Açıklama

- Middleware, genel middleware, uygulama middleware (yalnızca çoklu uygulama modunda geçerlidir, bkz. [Çoklu Uygulama](multiapp.md)) ve route (yol) middleware olarak üçe ayrılır.
- Middleware yapılandırma dosyası konumu `config/middleware.php` içindedir.
- Genel middleware yapılandırması `''` anahtarı altında yapılır.
- Uygulama middleware yapılandırması belirli bir uygulama adının altında yapılır, örneğin

```php
return [
    // Genel middleware
    '' => [
        app\middleware\AuthCheckTest::class,
        app\middleware\AccessControlTest::class,
    ],
    // api uygulama middleware (uygulama middleware yalnızca çoklu uygulama modunda geçerlidir)
    'api' => [
        app\middleware\ApiOnly::class,
    ]
];
```

## Denetleyici Middleware ve Metot Middleware

Açıklamalar (annotations) kullanarak, bir denetleyiciye veya denetleyicinin belirli metotlarına middleware atayabiliriz.

```php
<?php
namespace app\controller;
use app\middleware\Controller1Middleware;
use app\middleware\Controller2Middleware;
use app\middleware\Method1Middleware;
use app\middleware\Method2Middleware;
use support\annotation\Middleware;
use support\Request;

#[Middleware(Controller1Middleware::class, Controller2Middleware::class)]
class IndexController
{
    #[Middleware(Method1Middleware::class, Method2Middleware::class)]
    public function index(Request $request): string
    {
        return 'hello';
    }
}
```

## Route (Yol) Middleware

Belirli bir veya bir grup routelere middleware (ara yazılım) atanabilir.
Örneğin `config/route.php` içine aşağıdaki yapılandırmayı ekleyebiliriz:

```php
<?php
use support\Request;
use Webman\Route;

Route::any('/admin', [app\admin\controller\IndexController::class, 'index'])->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);

Route::group('/blog', function () {
   Route::any('/create', function () {return response('create');});
   Route::any('/edit', function () {return response('edit');});
   Route::any('/view/{id}', function ($r, $id) {response("view $id");});
})->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);
```

## Middleware İnşa Fonksiyonuna Parametre Geçme

Yapılandırma dosyası doğrudan middleware örneklemesini destekler, böylece constructor üzerinden middleware'e parametre geçmek kolaylaşır.

Örneğin `config/middleware.php` dosyasında şu şekilde yapılandırma yapılabilir:

```php
return [
    // Genel middleware
    '' => [
        new app\middleware\AuthCheckTest($param1, $param2, ...),
        new app\middleware\AccessControlTest($param1, $param2, ...)
    ],
    // api uygulama middleware (uygulama middleware yalnızca çoklu uygulama modunda geçerlidir)
    'api' => [
        app\middleware\ApiOnly::class,
    ]
];
```

Aynı zamanda route (yol) middleware de constructor üzerinden parametre geçebilir, örneğin `config/route.php` içinde:

```php
Route::any('/admin', [app\admin\controller\IndexController::class, 'index'])->middleware([
    new app\middleware\MiddlewareA($param1, $param2, ...),
    new app\middleware\MiddlewareB($param1, $param2, ...),
]);
```

Middleware içinde parametre kullanım örneği:

```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class MiddlewareA implements MiddlewareInterface
{
    protected $param1;

    protected $param2;

    public function __construct($param1, $param2)
    {
        $this->param1 = $param1;
        $this->param2 = $param2;
    }

    public function process(Request $request, callable $handler) : Response
    {
        var_dump($this->param1, $this->param2);
        return $handler($request);
    }
}
```

## Middleware Yürütme Sırası
- Middleware yürütme sırası: `genel middleware` -> `uygulama middleware` -> `denetleyici middleware` -> `route (yol) middleware` -> `metot middleware`.
- Aynı seviyede birden fazla middleware olduğunda, yapılandırılan sıraya göre çalışır.
- 404 istekleri varsayılan olarak hiçbir middleware tetiklemez (yine de `Route::fallback(function(){})->middleware()` ile middleware eklenebilir).

## Route (Yol) Middleware'e Parametre Geçme (route->setParams)

**Route yapılandırması `config/route.php`**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/test', [app\controller\IndexController::class, 'index'])->setParams(['some_key' =>'some value']);
```

**Middleware (varsayılan olarak genel middleware alalım)**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        // Varsayılan route $request->route null olduğu için, $request->route'un boş olup olmadığını kontrol etmemiz gerekiyor
        if ($route = $request->route) {
            $value = $route->param('some_key');
            var_export($value);
        }
        return $handler($request);
    }
}
```

## Middleware'den Denetleyiciye Parametre Geçme

Bazı durumlarda denetleyicinin middleware'de oluşturulan verileri kullanması gerekebilir, bu durumda middleware'de `$request` nesnesine özellik ekleyerek denetleyiciye parametre geçebiliriz. Örneğin:

**Middleware**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $request->data = 'some value';
        return $handler($request);
    }
}
```

**Denetleyici:**
```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response($request->data);
    }
}
```
## Middleware, mevcut istek rota bilgilerini almak

`$request->route` kullanarak route objesini alabiliriz ve ilgili bilgileri almak için ilgili yöntemleri çağırabiliriz.

**Route Ayarı**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/user/{uid}', [app\controller\UserController::class, 'view']);
```

**Middleware**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $route = $request->route;
        // İstek hiçbir rota ile eşleşmiyorsa (varsayılan rota hariç), $request->route null olur
        // Diyelim ki tarayıcı adresine /user/111 girdik, o zaman şu bilgileri yazdırabiliriz
        if ($route) {
            var_export($route->getPath());       // /user/{uid}
            var_export($route->getMethods());    // ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD','OPTIONS']
            var_export($route->getName());       // user_view
            var_export($route->getMiddleware()); // []
            var_export($route->getCallback());   // ['app\\controller\\UserController', 'view']
            var_export($route->param());         // ['uid'=>111]
            var_export($route->param('uid'));    // 111 
        }
        return $handler($request);
    }
}
```

## Middleware, istisnaları almak

Middleware içerisinde `$response->exception()` kullanarak işlem sırasında oluşan istisnaları alabiliriz.

**Rota Ayarı**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/user/{uid}', function (Request $request, $uid) {
    throw new \Exception('exception test');
});
```

**Middleware:**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $response = $handler($request);
        $exception = $response->exception();
        if ($exception) {
            echo $exception->getMessage();
        }
        return $response;
    }
}
```


## Süper Global Middleware

Ana projenin küresel middleware'leri sadece ana projeyi etkiler, [uygulama eklentileri](app/app.md) üzerinde herhangi bir etkiye sahip değildir. Bazı durumlarda, tüm eklentileri de etkileyen bir middleware eklemek isteyebiliriz, bu durumda süper global middleware'i kullanabiliriz.

`config/middleware.php` dosyasında aşağıdaki gibi yapılandırın:
```php
return [
    '@' => [ // Ana projeyi ve tüm eklentileri etkileyecek global middleware ekleyin
        app\middleware\MiddlewareGlobl::class,
    ], 
    '' => [], // Sadece ana projeye global middleware ekleyin
];
```

> **İpucu**
> `@` süper global middleware'ı sadece ana projede değil, aynı zamanda belirli bir eklentide de yapılandırabilirsiniz, örneğin `plugin/ai/config/middleware.php` dosyasında `@` süper global middleware'ı yapılandırırsanız, bu durumda ana projeyi ve tüm eklentileri etkiler.


## Belirli bir eklentiye middleware ekleme

Bazen [uygulama eklentileri](app/app.md) için belirli bir middleware eklemek isteyebiliriz, ancak eklentinin kodunu değiştirmek istemeyiz (çünkü güncellemelerde üzerine yazılabilir), bu durumda ana projede middleware ekleyebiliriz.

`config/middleware.php` dosyasında aşağıdaki gibi yapılandırın:
```php
return [
    'plugin.ai' => [], // ai eklentisine middleware ekleyin
    'plugin.ai.admin' => [], // ai eklentisinin admin modülüne middleware ekleyin (plugin\ai\app\admin dizini)
];
```

> **İpucu**
> Tabii ki, aynı yapılandırmayı bir eklentiye etki etmek için bu tür bir yapılandırmayı diğer eklentilere de ekleyebilirsiniz, örneğin `plugin/foo/config/middleware.php` dosyasına yukarıdaki yapılandırmayı eklerseniz, bu durum ai eklentisini etkileyecektir.
