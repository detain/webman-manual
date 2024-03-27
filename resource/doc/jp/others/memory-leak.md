# メモリリークについて
webmanは常駐メモリフレームワークですので、メモリリークには少し注意が必要です。ただし、開発者が過度に心配する必要はありません。なぜなら、メモリリークは非常に極端な条件で発生し、かつ容易に回避できるからです。webmanの開発は従来のフレームワークとほぼ同じですので、メモリ管理について余分な操作を行う必要はありません。

> **注意**
> webmanにはモニタープロセスが搭載されており、すべてのプロセスのメモリ使用状況を監視しています。プロセスのメモリ使用量がphp.iniの「memory_limit」で設定された値にほぼ達すると、該当プロセスを安全に再起動し、メモリを解放します。この間、ビジネスに影響を与えません。

## メモリリークの定義
リクエストが増えるにつれてwebmanが使用するメモリも**無限に増加**し、数百メガバイト、またはそれ以上に達する場合、それはメモリリークです。後続の増加がなくなればメモリリークではありません。

一般的にプロセスが数十メガバイトのメモリを使用することは非常に正常な状況です。しかし、プロセスが超大規模なリクエストを処理したり、大量の接続を維持する場合、単一のプロセスのメモリ使用量は数百メガバイトに達することもよくあります。このようなメモリ使用後、PHPはすべてのメモリをオペレーティングシステムに返さない可能性があります。そのため、ある大規模なリクエストを処理した後にメモリ使用量が解放されずに増加する事象は正常な現象です。（`gc_mem_caches()`メソッドを呼び出すことで一部の空きメモリが解放されます）

## メモリリークが発生する原因
**メモリリークは、以下の2つの条件を満たす場合に発生します：**
1. **長寿命の**配列が存在する（通常の配列ではありません）
2. そして、この**長寿命の**配列が無限に拡張される（ビジネスが終わることなくデータが挿入され続ける）

1と2の条件が**同時に**満たされると、メモリリークが発生します。条件を満たさないか、または1つの条件のみを満たす場合はメモリリークではありません。

## 長寿命の配列
webmanでの長寿命の配列は次の通りです：
1. staticキーワードを使用した配列
2. シングルトンの配列プロパティ
3. globalキーワードを使用した配列

> **注意**
> webmanでは、長寿命のデータを使用することができますが、そのデータ内の要素が有限であること、また要素数が無限に増えないことを保証する必要があります。

以下にそれぞれの例を示します。

#### 無限膨張するstatic配列
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

`static`キーワードで定義された`$data`配列は長寿命の配列です。そして、この例では、`$data`配列がリクエストごとに増加し続け、メモリリークを引き起こします。

#### 無限膨張するシングルトン配列プロパティ
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

呼び出しコード
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

`Cache::instance()`はCacheのシングルトンを返しますが、これは長寿命のクラスインスタンスです。`$data`プロパティに`static`キーワードは使用されていませんが、クラス自体が長寿命であるため、`$data`も長寿命の配列です。異なるキーのデータを`$data`配列に継続的に追加すると、プログラムのメモリ使用量が増大し、メモリリークが発生します。

> **注意**
> `Cache::instance()->set(key, value)`で追加されるキーが有限であれば、メモリリークは発生しません。なぜなら`$data`配列が無制限に膨張しないからです。

#### 無限膨張するglobal配列
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

関数やクラスメソッドの実行が終わってもglobalキーワードで定義された配列は回収されませんので、これは長寿命の配列です。上記のコードはリクエストが増加するとメモリリークを引き起こします。同様に、関数やメソッド内でstaticキーワードで定義された配列も長寿命の配列です。もし配列が無限に膨張すればメモリリークが発生します。例えば次のような場合です：
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

## アドバイス
メモリリークに特に注意を払う必要はありません。なぜなら、それは非常にまれなことですし、もし不幸にも発生した場合は、負荷テストを行い、リークを発生させるコードを特定することができます。開発者がリークポイントを見つけられなくても、webmanによって提供されるモニターサービスが必要な時にプロセスを安全に再起動し、メモリを解放します。

もしできるだけメモリリークを回避したい場合は、以下のアドバイスに従うことをお勧めします。
1. `global`, `static`キーワードで配列を使用しないようにし、使用する場合はその配列が無制限に膨張しないようにする
2. 不慣れなクラスについては、シングルトンを使用するのではなく、`new`キーワードで初期化するようにします。もしシングルトンが必要な場合は、そのクラスが無限に膨張する配列プロパティを持っていないかを確認してください。