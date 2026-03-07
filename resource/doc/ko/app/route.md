# 라우트 설정 파일
플러그인의 라우트 설정 파일은 `plugin/플러그인명/config/route.php`에 있습니다.

## 기본 라우트
애플리케이션 플러그인의 URL 경로는 모두 `/app`으로 시작합니다. 예를 들어 `plugin\foo\app\controller\UserController`의 URL은 `http://127.0.0.1:8787/app/foo/user`입니다.

## 기본 라우트 비활성화
특정 애플리케이션 플러그인의 기본 라우트를 비활성화하려면 라우트 설정에 다음을 추가하세요:
```php
Route::disableDefaultRoute('foo');
```

## 404 콜백 처리
애플리케이션 플러그인에 fallback을 설정하려면 두 번째 매개변수로 플러그인 이름을 전달해야 합니다. 예:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
