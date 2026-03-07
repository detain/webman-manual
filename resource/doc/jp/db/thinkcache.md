# think-cache

think-cacheはthinkphpフレームワークから抽出されたコンポーネントで、コネクションプール機能が追加されています。コルーチン環境と非コルーチン環境の両方を自動的にサポートします。

## インストール
`composer require -W webman/think-cache`

インストール後は restart で再起動が必要です（reload は無効）

### 設定ファイル

設定ファイルは `config/think-cache.php` です

## 使用方法

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
## 提供されるAPI
```php
// キャッシュを設定
Cache::set('val','value',600);
// キャッシュが存在するか確認
Cache::has('val');
// キャッシュを取得
Cache::get('val');
// キャッシュを削除
Cache::delete('val');
// キャッシュをクリア
Cache::clear();
// 読み取りと削除
Cache::pull('val');
// 存在しない場合は書き込み
Cache::remember('val',10);

// 数値型のキャッシュデータ用
// キャッシュを1増加
Cache::inc('val');
// キャッシュを5増加
Cache::inc('val',5);
// キャッシュを1減少
Cache::dec('val');
// キャッシュを5減少
Cache::dec('val',5);

// キャッシュタグを使用
Cache::tag('tag_name')->set('val','value',600);
// 特定タグのキャッシュを削除
Cache::tag('tag_name')->clear();
// 複数タグを指定可能
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// 複数タグのキャッシュを削除
Cache::tag(['tag1','tag2'])->clear();

// 複数のキャッシュストアを使用
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


