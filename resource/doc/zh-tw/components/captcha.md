# 驗證碼組件

專案地址 https://github.com/webman-php/captcha

## 安裝
```
composer require webman/captcha
```

## 使用

**建立檔案 `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * 測試頁面
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * 輸出驗證碼圖像
     */
    public function captcha(Request $request)
    {
        // 初始化驗證碼類
        $builder = new CaptchaBuilder;
        // 生成驗證碼
        $builder->build();
        // 將驗證碼的值儲存到 session 中
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // 取得驗證碼圖片二進位資料
        $img_content = $builder->get();
        // 輸出驗證碼二進位資料
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * 檢查驗證碼
     */
    public function check(Request $request)
    {
        // 取得 POST 請求中的 captcha 欄位
        $captcha = $request->post('captcha');
        // 比對 session 中的 captcha 值
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => '輸入的驗證碼不正確']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**建立範本檔案 `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>驗證碼測試</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="提交" />
    </form>
</body>
</html>
```

進入頁面 `http://127.0.0.1:8787/login` 後，介面類似如下：
  ![](../../assets/img/captcha.png)

## 常見參數設置
```php
    /**
     * 輸出驗證碼圖像
     */
    public function captcha(Request $request)
    {
        $builder = new PhraseBuilder(4, 'abcdefghjkmnpqrstuvwxyzABCDEFGHJKMNPQRSTUVWXYZ');
        $captcha = new CaptchaBuilder(null, $builder);
        $captcha->build();
        $request->session()->set('join', strtolower($captcha->getPhrase()));
        $img_content = $captcha->get();
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }
```

更多介面及參數請參考 https://github.com/webman-php/captcha
