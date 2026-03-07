# webman session 管理

## 範例
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

透過 `$request->session();` 取得 `Workerman\Protocols\Http\Session` 實例，並透過實例的方法來新增、修改、刪除 session 資料。

> **注意**
> session 物件銷毀時會自動儲存 session 資料
> 將 session 物件儲存到全域變數會阻止 session 銷毀，導致無法自動儲存 session 資料，此時需手動呼叫 `$session->save()` 儲存

## 取得所有 session 資料
```php
$session = $request->session();
$all = $session->all();
```
回傳值為陣列。若沒有任何 session 資料，則回傳空陣列。


## 取得 session 中某個值
```php
$session = $request->session();
$name = $session->get('name');
```
若資料不存在則回傳 null。

你也可以在 get 方法的第二個參數傳入預設值，若 session 陣列中沒找到對應值則回傳該預設值。例如：
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## 儲存 session
儲存單一筆資料時使用 set 方法。
```php
$session = $request->session();
$session->set('name', 'tom');
```
set 沒有回傳值，session 物件銷毀時 session 會自動儲存。

當儲存多個值時使用 put 方法。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同樣地，put 也沒有回傳值。

## 刪除 session 資料
刪除單一項或多項 session 資料時使用 `forget` 方法。
```php
$session = $request->session();
// 刪除一項
$session->forget('name');
// 刪除多項
$session->forget(['name', 'age']);
```

另外系統提供了 delete 方法，與 forget 方法不同的是，delete 只能刪除一項。
```php
$session = $request->session();
// 等同於 $session->forget('name');
$session->delete('name');
```

## 取得並刪除 session 某個值
```php
$session = $request->session();
$name = $session->pull('name');
```
效果與以下程式碼相同
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
若對應的 session 不存在，則回傳 null。


## 刪除所有 session 資料
```php
$request->session()->flush();
```
沒有回傳值，session 物件銷毀時 session 會自動從儲存中刪除。


## 判斷對應 session 資料是否存在
```php
$session = $request->session();
$has = $session->has('name');
```
以上當對應的 session 不存在或對應的 session 值為 null 時回傳 false，否則回傳 true。

```php
$session = $request->session();
$has = $session->exists('name');
```
以上程式碼也可用來判斷 session 資料是否存在，差別在於當對應的 session 項值為 null 時，也會回傳 true。

## 輔助函數 session()

webman 提供了輔助函數 `session()` 來完成相同功能。
```php
// 取得 session 實例
$session = session();
// 等同於
$session = $request->session();

// 取得某個值
$value = session('key', 'default');
// 等同於
$value = session()->get('key', 'default');
// 等同於
$value = $request->session()->get('key', 'default');

// 給 session 賦值
session(['key1'=>'value1', 'key2' => 'value2']);
// 相當於
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// 相當於
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## 設定檔
session 設定檔位於 `config/session.php`，內容類似如下：
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class 或 RedisSessionHandler::class 或 RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handler 為 FileSessionHandler::class 時值為 file，
    // handler 為 RedisSessionHandler::class 時值為 redis
    // handler 為 RedisClusterSessionHandler::class 時值為 redis_cluster（即 Redis 叢集）
    'type'    => 'file',

    // 不同的 handler 使用不同的設定
    'config' => [
        // type 為 file 時的設定
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // type 為 redis 時的設定
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // 儲存 session_id 的 cookie 名稱
    'auto_update_timestamp' => false,  // 是否自動重新整理 session，預設關閉
    'lifetime' => 7*24*60*60,          // session 過期時間
    'cookie_lifetime' => 365*24*60*60, // 儲存 session_id 的 cookie 過期時間
    'cookie_path' => '/',              // 儲存 session_id 的 cookie 路徑
    'domain' => '',                    // 儲存 session_id 的 cookie 網域
    'http_only' => true,               // 是否開啟 httpOnly，預設開啟
    'secure' => false,                 // 僅在 https 下開啟 session，預設關閉
    'same_site' => '',                 // 用於防止 CSRF 攻擊與使用者追蹤，可選值 strict/lax/none
    'gc_probability' => [1, 1000],     // 回收 session 的機率
];
```

## 安全性
使用 session 時不建議直接儲存類的實例物件，尤其來自不可控來源的類實例，反序列化時可能造成潛在安全風險。

