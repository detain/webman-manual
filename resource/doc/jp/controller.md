# コントローラー

新しいコントローラーファイル `app/controller/FooController.php` を作成します。

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

`http://127.0.0.1:8787/foo` にアクセスすると、ページは `hello index` を返します。

`http://127.0.0.1:8787/foo/hello` にアクセスすると、ページは `hello webman` を返します。

もちろん、[ルート](route.md)を参照してルート設定でルールを変更できます。

> **ヒント**
> 404エラーが発生する場合は、`config/app.php` を開いて `controller_suffix` を `Controller` に設定し、再起動してください。

## コントローラーのサフィックス
webman 1.3 以降、`config/app.php` でコントローラーのサフィックスを設定できます。`controller_suffix` が空文字 `''` に設定されている場合、コントローラーは以下のようになります。

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

コントローラーのサフィックスを `Controller` に設定することを強く推奨します。これによりコントローラーとモデルクラス名の衝突を避け、セキュリティも向上します。

## 説明
- フレームワークは自動的に `support\Request` オブジェクトをコントローラーに渡し、それを介してユーザー入力データ（get、post、header、cookie など）を取得できます。[リクエスト](request.md)を参照してください。
- コントローラーでは数値、文字列、または `support\Response` オブジェクトを返すことができますが、他の型のデータは返せません。
- `support\Response` オブジェクトは `response()`、`json()`、`xml()`、`jsonp()`、`redirect()` などのヘルパー関数で作成できます。

## コントローラーのパラメータバインディング

#### 例
webman はコントローラーメソッドのパラメータへのリクエストパラメータの自動バインディングをサポートしています。例えば：

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

`name` と `age` の値は `GET` または `POST` で渡すことも、ルートパラメータで渡すこともできます。例えば：

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

優先順位は `ルートパラメータ` > `GET` > `POST` パラメータです。

#### デフォルト値

`/user/create?name=tom` にアクセスすると、次のエラーが発生します。

```html
Missing input parameter age
```

`age` パラメータを渡していないためです。パラメータにデフォルト値を設定することで解決できます。例えば：

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

#### パラメータの型
`/user/create?name=tom&age=not_int` にアクセスすると、次のエラーが発生します。

> **ヒント**
> テストの便宜上、ブラウザのアドレスバーに直接パラメータを入力しています。実際の開発では `POST` でパラメータを渡すべきです。

```html
Input age must be of type int, string given
```

受け取ったデータは型に応じて変換されます。変換できない場合は `support\exception\InputTypeException` がスローされます。`age` パラメータを `int` 型に変換できないため、上記のエラーが発生します。

#### カスタムエラーメッセージ
`Missing input parameter age` や `Input age must be of type int, string given` などのエラーメッセージは、多言語対応でカスタマイズできます。以下のコマンドを参照してください。

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => '入力パラメータ :parameter は :exceptType 型である必要があります。渡された型は :actualType です',
    'Missing input parameter :parameter' => '入力パラメータ :parameter が不足しています',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### その他の型
webman がサポートするパラメータ型は `int`、`float`、`string`、`bool`、`array`、`object`、`クラスインスタンス` などです。例えば：

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

`/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` にアクセスすると、次の結果が得られます。

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

#### クラスインスタンス
webman はパラメータの型ヒントによるクラスインスタンスの渡し方をサポートしています。例えば：

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

`/blog/create?blog[title]=hello&blog[content]=world` にアクセスすると、次の結果が得られます。

```json
{
  "title": "hello",
  "content": "world"
}
```

#### モデルインスタンス

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
    // フロントエンドから安全でないフィールドが渡されないよう、ここにフィル可能なフィールドを追加する必要があります
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

`/user/create?user[name]=tom&user[age]=18` にアクセスすると、次のような結果が得られます。

```json
1
```

## コントローラーのライフサイクル

`config/app.php` で `controller_reuse` が `false` の場合、各リクエストで対応するコントローラーインスタンスが1回初期化され、リクエスト終了後に破棄されます。これは従来のフレームワークの動作と同じです。

`config/app.php` で `controller_reuse` が `true` の場合、すべてのリクエストでコントローラーインスタンスが再利用されます。つまり、一度作成されたコントローラーインスタンスはメモリに常駐し、すべてのリクエストで再利用されます。

> **注意**
> コントローラーの再利用を有効にしている場合、リクエストでコントローラーのプロパティを変更しないでください。変更は後続のリクエストに影響します。例えば：

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
        // このメソッドは最初のリクエスト update?id=1 の後にモデルを保持します
        // 次に delete?id=2 をリクエストすると、id=1 のデータが削除されます
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **ヒント**
> コントローラーの `__construct()` コンストラクタで return しても効果はありません。例えば：

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // コンストラクタで return しても効果はありません。ブラウザはこのレスポンスを受け取りません
        return response('hello'); 
    }
}
```

## コントローラーの再利用と非再利用の違い
違いは以下のとおりです。

#### 非再利用コントローラー
各リクエストで新しいコントローラーインスタンスが作成され、リクエスト終了後に解放されメモリが回収されます。非再利用は従来のフレームワークと同様で、多くの開発者の慣習に合致します。コントローラーの繰り返し作成・破棄により、パフォーマンスは再利用よりやや劣ります（helloworld ベンチマークで約10%程度、実ビジネスではほぼ無視できます）。

#### 再利用コントローラー
再利用の場合、プロセスごとに1回だけコントローラーが new され、リクエスト終了後もインスタンスは解放されません。同じプロセスの後続リクエストでこのインスタンスが再利用されます。再利用はパフォーマンスが優れますが、多くの開発者の慣習には合いません。

#### 以下の場合はコントローラーの再利用を使用できません

リクエストがコントローラーのプロパティを変更する場合は、コントローラーの再利用を有効にできません。プロパティの変更は後続のリクエストに影響します。

コントローラーの `__construct()` コンストラクタでリクエストごとに初期化を行いたい開発者の場合も、コントローラーの再利用は使えません。現在のプロセスではコンストラクタは1回だけ呼ばれ、リクエストごとには呼ばれないためです。
