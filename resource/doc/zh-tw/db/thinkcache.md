# think-cache

think-cache 是從 thinkphp 框架抽離出的一個元件，並加入了連線池功能，自動支援協程與非協程環境。

## 安裝
`composer require -W webman/think-cache`

安裝後需使用 restart 重啟（reload 無效）

### 設定檔

設定檔為 `config/think-cache.php`

## 使用

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
## 提供的介面
```php
// 設定快取
Cache::set('val','value',600);
// 判斷快取是否已設定
Cache::has('val');
// 取得快取
Cache::get('val');
// 刪除快取
Cache::delete('val');
// 清除快取
Cache::clear();
// 讀取並刪除快取
Cache::pull('val');
// 不存在則寫入
Cache::remember('val',10);

// 對於數值類型的快取資料可使用
// 快取加 1
Cache::inc('val');
// 快取加 5
Cache::inc('val',5);
// 快取減 1
Cache::dec('val');
// 快取減 5
Cache::dec('val',5);

// 使用快取標籤
Cache::tag('tag_name')->set('val','value',600);
// 刪除某個標籤下的快取資料
Cache::tag('tag_name')->clear();
// 支援指定多個標籤
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// 刪除多個標籤下的快取資料
Cache::tag(['tag1','tag2'])->clear();

// 使用多種快取類型
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


