# 認証コードコンポーネント

プロジェクトリンク https://github.com/webman-php/captcha

## インストール
```
composer require webman/captcha
```

## 使用方法

**ファイル `app/controller/LoginController.php` を作成**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * テストページ
     */
    public function index(Request $request)
    {
        return view('login/index');
    }
    
    /**
     * 認証コード画像を出力
     */
    public function captcha(Request $request)
    {
        // 認証コードクラスを初期化
        $builder = new CaptchaBuilder;
        // 認証コードを生成
        $builder->build();
        // 認証コードの値をセッションに保存
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // 認証コード画像のバイナリデータを取得
        $img_content = $builder->get();
        // 認証コードのバイナリデータを出力
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * 認証コードをチェック
     */
    public function check(Request $request)
    {
        // POSTリクエストからcaptchaフィールドを取得
        $captcha = $request->post('captcha');
        // セッション内のcaptcha値と比較
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => '入力した認証コードが正しくありません']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**テンプレートファイル `app/view/login/index.html` を作成**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>認証コードテスト</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="送信" />
    </form>
</body>
</html>
```

ページ `http://127.0.0.1:8787/login` にアクセスすると、以下のような画面が表示されます：
  ![](../../assets/img/captcha.png)

## よくあるパラメータ設定
```php
    /**
     * 認証コード画像を出力
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

その他のAPIおよびパラメータについては https://github.com/webman-php/captcha を参照してください。
