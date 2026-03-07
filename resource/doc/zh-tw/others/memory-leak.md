# 關於記憶體洩漏
webman 是一個常駐記憶體框架，因此我們需要稍微關注記憶體洩漏的情況。不過開發者不必過於擔心，因為記憶體洩漏發生在非常極端的條件下，而且很容易規避。webman 開發與傳統框架開發體驗基本一致，不必為記憶體管理做多餘的操作。

> **提示**
> webman 自帶的 monitor 進程會監控所有進程記憶體使用情況，如果進程使用記憶體即將達到 php.ini 裡 `memory_limit` 設定的值時，會自動安全重啟對應的進程，達到釋放記憶體的作用，期間對業務沒有影響。

## 記憶體洩漏定義
隨著請求增加，webman 記憶體不斷增長是正常現象。一般來說當進程達到一定請求量後（一般百萬級別），記憶體將停止增長或者偶爾小幅度增長。
大部分業務單個進程記憶體佔用最終會在 10M–100M 左右保持穩定，單進程記憶體沒超過 100M 則不用擔心。
另外當業務處理大檔案、處理大請求、從資料庫讀取大資料等業務時，PHP 會申請大量記憶體，這部分記憶體 PHP 使用後可能會保留下次複用，不會全部交還給作業系統，這時候也會出現記憶體佔用很大的現象。由於記憶體會被重複利用，所以不用擔心。

> **提示**
> 如果是 phar 打包或者二進位打包專案，本包本身尺寸很大，打包專案記憶體佔用超過 100M 也是正常現象。

## 如何確認記憶體洩漏
如果一個進程請求超過百萬，記憶體超過 100M，每次請求後記憶體還在增長，那麼有可能發生了記憶體洩漏。

## 如何查找記憶體洩漏
簡單的定位方法是通過壓測各個介面，找出哪個介面在超過百萬次請求後記憶體還在不斷增長。
當發現問題介面後，利用二分法每次註釋掉一半的業務程式碼，直到最終確定哪部分程式碼有問題。

## 記憶體洩漏是如何發生的
**記憶體洩漏發生必須滿足以下兩個條件：**
1. 存在**長生命週期**的陣列（注意是長生命週期的陣列，普通陣列沒事）
2. 並且這個**長生命週期**的陣列會無限擴張（業務無限向其插入資料，從不清理資料）

如果 1、2 條件**同時滿足**（注意是同時滿足），那麼將會產生記憶體洩漏。反之不滿足以上條件或者只滿足其中一個條件則不是記憶體洩漏。

## 長生命週期的陣列

webman 裡長生命週期的陣列包括：
1. static 關鍵字的陣列
2. 單例的陣列屬性
3. global 關鍵字的陣列

> **注意**
> webman 中允許使用長生命週期的資料，但是需要保證資料內的資料是有限的，元素個數不會無限擴張。

以下分別舉例說明

### 無限膨脹的 static 陣列
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

以 `static` 關鍵字定義的 `$data` 陣列是長生命週期的陣列，並且範例中 `$data` 陣列隨著請求不斷增加而不斷膨脹，導致記憶體洩漏。

### 無限膨脹的單例陣列屬性
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

呼叫程式碼
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

`Cache::instance()` 返回一個 Cache 單例，它是一個長生命週期的類實例。雖然它的 `$data` 屬性沒有使用 `static` 關鍵字，但是由於類本身是長生命週期，所以 `$data` 也是長生命週期的陣列。隨著不斷向 `$data` 陣列裡新增不同 key 的資料，程式佔用記憶體也越來越大，造成記憶體洩漏。

> **注意**
> 如果 Cache::instance()->set(key, value) 新增的 key 是有限數量的，則不會記憶體洩漏，因為 `$data` 陣列並沒有無限膨脹。

### 無限膨脹的 global 陣列
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
global 關鍵字定義的陣列並不會在函式或者類方法執行完畢後回收，所以它是長生命週期的陣列。以上程式碼隨著請求不斷增加會產生記憶體洩漏。同理在函式或者方法內以 static 關鍵字定義的陣列也是長生命週期的陣列，如果陣列無限膨脹也會記憶體洩漏，例如：
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

## 建議
建議開發者不用特別關注記憶體洩漏，因為它極少發生。如果不幸發生我們可以通過壓測找到哪段程式碼產生洩漏，從而定位出問題。即使開發者沒有找到洩漏點，webman 自帶的 monitor 服務會適時安全重啟發生記憶體洩漏的進程，釋放記憶體。

如果你實在想盡量規避記憶體洩漏，可以參考以下建議。
1. 盡量不使用 `global`、`static` 關鍵字的陣列，如果使用確保其不會無限膨脹
2. 對於不熟悉的類，盡量不使用單例，用 new 關鍵字初始化。如果需要單例，則查看其是否有無限膨脹的陣列屬性
