# event 事件處理
`webman/event` 提供一種精巧的事件機制，可實現在不侵入程式碼的情況下執行一些業務邏輯，實現業務模組之間的解耦。典型的場景如一個新用戶註冊成功時，只要發佈一個自訂事件如`user.register`，各個模組便能收到該事件執行相應的業務邏輯。

## 安裝
`composer require webman/event`

## 訂閱事件
訂閱事件統一透過檔案`config/event.php`來配置
```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...其它事件處理函數...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...其它事件處理函數...
    ]
];
```
**說明：**
- `user.register`、`user.logout` 等是事件名稱，字串類型，建議小寫單詞並以點(`.`)分割
- 一個事件可以對應多個事件處理函數，呼叫順序為配置的順序

## 事件處理函數
事件處理函數可以是任意的類別方法、函數、閉包函數等。
例如建立事件處理類別 `app/event/User.php`（目錄不存在請自行建立）
```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## 發佈事件
使用 `Event::dispatch($event_name, $data);` 或 `Event::emit($event_name, $data);` 發佈事件，例如
```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

發佈事件有兩個函數，`Event::dispatch($event_name, $data);` 和 `Event::emit($event_name, $data);`，二者參數相同。
差別在於 emit 內部會自動捕獲異常，也就是說一個事件若有多個處理函數，某個處理函數發生異常不會影響其它處理函數的執行。
而 dispatch 內部不會自動捕獲異常，當前事件的任一個處理函數發生異常時，會停止執行下一個處理函數並直接向上拋出異常。

> **提示**
> 參數 $data 可以是任意的資料，例如陣列、類別實例、字串等

## 萬用字元事件監聽
萬用字元註冊監聽允許您在同一個監聽器上處理多個事件，例如在`config/event.php`裡配置
```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```
我們可以透過事件處理函數第二個參數`$event_data`取得具體的事件名稱
```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // 具體的事件名稱，如 user.register、user.logout 等
        var_export($user);
    }
}
```

## 停止事件廣播
當我們在事件處理函數裡回傳`false`時，該事件將停止廣播

## 閉包函數處理事件
事件處理函數可以是類別方法，也可以是閉包函數，例如

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## 查看事件及監聽器
使用指令 `php webman event:list` 查看項目配置的所有事件及監聽器

## 支援範圍
除了主項目，[基礎插件](../plugin/base.md)與[應用插件](../app/app.md)同樣支援 event.php 配置。
**基礎插件配置檔** `config/plugin/插件廠商/插件名稱/event.php`
**應用插件配置檔** `plugin/插件名稱/config/event.php`

## 注意事項
event 事件處理並非非同步，event 不適合處理慢業務，慢業務應使用訊息佇列處理，例如 [webman/redis-queue](https://www.workerman.net/plugin/12)
