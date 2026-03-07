# 控制器

新建控制器檔案 `app/controller/FooController.php`。

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

當存取 `http://127.0.0.1:8787/foo` 時，頁面會回傳 `hello index`。

當存取 `http://127.0.0.1:8787/foo/hello` 時，頁面會回傳 `hello webman`。

當然你可以透過路由設定來變更路由規則，請參閱[路由](route.md)。

> **提示**
> 如果出現 404 無法存取，請開啟 `config/app.php`，將 `controller_suffix` 設為 `Controller`，並重新啟動。

## 控制器後綴
從 webman 1.3 版本開始，支援在 `config/app.php` 設定控制器後綴。若 `config/app.php` 中的 `controller_suffix` 設為空字串 `''`，則控制器類似如下：

`app\controller\Foo.php`。

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

強烈建議將控制器後綴設為 `Controller`，如此可避免控制器與模型類別名稱衝突，同時提高安全性。

## 說明
 - 框架會自動將 `support\Request` 物件傳入控制器，透過它可取得使用者輸入資料（get、post、header、cookie 等），請參閱[請求](request.md)
 - 控制器可以回傳數字、字串或 `support\Response` 物件，但不能回傳其他類型的資料。
 - `support\Response` 物件可透過 `response()`、`json()`、`xml()`、`jsonp()`、`redirect()` 等輔助函式建立。

## 控制器參數綁定

#### 範例
webman 支援透過控制器方法參數自動綁定請求參數，例如：

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

你可以透過 `GET` 或 `POST` 傳遞 `name` 和 `age` 的值，也可透過路由參數傳遞，例如：

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

優先順序為：`路由參數` > `GET` > `POST` 參數

#### 預設值

假設我們存取 `/user/create?name=tom`，會得到以下錯誤：

```html
Missing input parameter age
```

原因是我們沒有傳遞 `age` 參數，可藉由為參數設定預設值解決，例如：

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

#### 參數類型
當我們存取 `/user/create?name=tom&age=not_int`，會得到以下錯誤：

> **提示**
> 這裡為方便測試，我們直接在瀏覽器網址列輸入參數，實際開發中應透過 `POST` 方式傳遞參數

```html
Input age must be of type int, string given
```

這是因為接收的資料會依類型進行轉換，若無法轉換則會拋出 `support\exception\InputTypeException` 例外。由於傳遞的 `age` 參數無法轉換為 `int` 類型，故會得到上述錯誤。

#### 自訂錯誤訊息
我們可以利用多語系自訂 `Missing input parameter age` 和 `Input age must be of type int, string given` 等錯誤訊息，請參考以下指令：

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => '輸入參數 :parameter 必須是 :exceptType 類型，傳遞的類型是 :actualType',
    'Missing input parameter :parameter' => '缺少輸入參數 :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### 其他類型
webman 支援的參數類型有 `int`、`float`、`string`、`bool`、`array`、`object`、`類別實例` 等，例如：

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

當我們存取 `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`，會得到以下結果：

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

#### 類別實例
webman 支援透過參數型別提示傳遞類別實例，例如：

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

當我們存取 `/blog/create?blog[title]=hello&blog[content]=world`，會得到以下結果：

```json
{
  "title": "hello",
  "content": "world"
}
```

#### 模型實例

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
    // 這裡需要新增可填充欄位，防止前端傳入不安全的欄位
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

當我們存取 `/user/create?user[name]=tom&user[age]=18`，會得到類似以下的結果：

```json
1
```

## 控制器生命週期

當 `config/app.php` 中 `controller_reuse` 為 `false` 時，每個請求都會初始化一次對應的控制器實例，請求結束後控制器實例銷毀，這與傳統框架的執行機制相同。

當 `config/app.php` 中 `controller_reuse` 為 `true` 時，所有請求會複用控制器實例，也就是控制器實例一旦建立便常駐記憶體，所有請求共用。

> **注意**
> 開啟控制器複用時，請求不應變更控制器的任何屬性，因為這些變更會影響後續請求，例如：

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
        // 此方法會在第一次請求 update?id=1 後保留 model
        // 若再次請求 delete?id=2 時，會刪除 id=1 的資料
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **提示**
> 在控制器 `__construct()` 建構函式中 return 資料不會產生任何效果，例如：

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // 在建構函式中 return 資料沒有任何效果，瀏覽器不會收到此回應
        return response('hello'); 
    }
}
```

## 控制器不複用與複用的差異
差異如下：

#### 不複用控制器
每個請求都會重新 new 一個新的控制器實例，請求結束後釋放該實例並回收記憶體。不複用控制器與傳統框架相同，符合多數開發者習慣。由於控制器反覆建立與銷毀，效能會比複用控制器略差（helloworld 壓測約差 10%，帶業務邏輯時可基本忽略）。

#### 複用控制器
複用的話，一個行程只 new 一次控制器，請求結束後不釋放該控制器實例，當前行程的後續請求會複用此實例。複用控制器效能較好，但不符合多數開發者習慣。

#### 以下情況不能使用控制器複用

當請求會變更控制器的屬性時，不能開啟控制器複用，因為這些屬性的變更會影響後續請求。

有些開發者習慣在控制器建構函式 `__construct()` 中針對每個請求做一些初始化，此時就不能複用控制器，因為當前行程的建構函式只會呼叫一次，並非每個請求都會呼叫。
