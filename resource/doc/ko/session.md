# webman 세션 관리

## 예시
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

`$request->session();`을 통해 `Workerman\Protocols\Http\Session` 인스턴스를 얻고, 인스턴스 메서드로 세션 데이터를 추가·수정·삭제합니다.

> **주의**
> 세션 객체가 파괴될 때 세션 데이터가 자동으로 저장됩니다.
> 세션 객체를 전역 변수에 저장하면 객체가 파괴되지 않아 자동 저장이 되지 않습니다. 이 경우 수동으로 `$session->save()`를 호출하여 저장해야 합니다.

## 모든 세션 데이터 가져오기
```php
$session = $request->session();
$all = $session->all();
```
배열을 반환합니다. 세션 데이터가 없으면 빈 배열을 반환합니다.


## 세션에서 특정 값 가져오기
```php
$session = $request->session();
$name = $session->get('name');
```
데이터가 없으면 null을 반환합니다.

get 메서드의 두 번째 인자로 기본값을 전달할 수도 있습니다. 세션 배열에서 해당 값이 없으면 기본값을 반환합니다. 예:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## 세션 저장
단일 데이터를 저장할 때는 set 메서드를 사용합니다.
```php
$session = $request->session();
$session->set('name', 'tom');
```
set은 반환값이 없으며, 세션 객체가 파괴될 때 세션이 자동으로 저장됩니다.

여러 값을 저장할 때는 put 메서드를 사용합니다.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
마찬가지로 put도 반환값이 없습니다.

## 세션 데이터 삭제
하나 또는 여러 세션 데이터를 삭제할 때는 `forget` 메서드를 사용합니다.
```php
$session = $request->session();
// 한 항목 삭제
$session->forget('name');
// 여러 항목 삭제
$session->forget(['name', 'age']);
```

또한 delete 메서드가 제공됩니다. forget와 달리 한 항목만 삭제할 수 있습니다.
```php
$session = $request->session();
// $session->forget('name'); 과 동일
$session->delete('name');
```

## 세션 값 가져오기 후 삭제
```php
$session = $request->session();
$name = $session->pull('name');
```
다음 코드와 같은 효과입니다.
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
해당 세션이 없으면 null을 반환합니다.


## 모든 세션 데이터 삭제
```php
$request->session()->flush();
```
반환값이 없으며, 세션 객체가 파괴될 때 세션은 자동으로 스토리지에서 삭제됩니다.


## 특정 세션 데이터 존재 여부 확인
```php
$session = $request->session();
$has = $session->has('name');
```
해당 세션이 없거나 값이 null이면 false, 그렇지 않으면 true를 반환합니다.

```php
$session = $request->session();
$has = $session->exists('name');
```
위 코드도 세션 데이터 존재 여부를 확인합니다. 차이는 세션 값이 null이어도 true를 반환한다는 점입니다.

## 헬퍼 함수 session()

webman은 동일한 기능을 수행하는 헬퍼 함수 `session()`을 제공합니다.
```php
// 세션 인스턴스 가져오기
$session = session();
// 아래와 동일
$session = $request->session();

// 값 가져오기
$value = session('key', 'default');
// 아래와 동일
$value = session()->get('key', 'default');
// 아래와 동일
$value = $request->session()->get('key', 'default');

// 세션에 값 설정
session(['key1'=>'value1', 'key2' => 'value2']);
// 아래와 동일
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// 아래와 동일
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## 설정 파일
세션 설정 파일은 `config/session.php`에 있으며, 내용은 다음과 같습니다.
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class 또는 RedisSessionHandler::class 또는 RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handler가 FileSessionHandler::class이면 file,
    // handler가 RedisSessionHandler::class이면 redis
    // handler가 RedisClusterSessionHandler::class이면 redis_cluster(Redis 클러스터)
    'type'    => 'file',

    // handler별로 다른 설정 사용
    'config' => [
        // type이 file일 때 설정
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // type이 redis일 때 설정
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

    'session_name' => 'PHPSID', // session_id를 저장하는 쿠키 이름
    'auto_update_timestamp' => false,  // 세션 자동 갱신 여부, 기본값은 끔
    'lifetime' => 7*24*60*60,          // 세션 만료 시간
    'cookie_lifetime' => 365*24*60*60, // session_id 쿠키 만료 시간
    'cookie_path' => '/',              // session_id 쿠키 경로
    'domain' => '',                    // session_id 쿠키 도메인
    'http_only' => true,               // httpOnly 사용 여부, 기본값은 켬
    'secure' => false,                 // HTTPS에서만 세션 사용 여부, 기본값은 끔
    'same_site' => '',                 // CSRF 공격 및 사용자 추적 방지, 값: strict/lax/none
    'gc_probability' => [1, 1000],     // 세션 가비지 컬렉션 확률
];
```

## 보안
세션 사용 시 클래스 인스턴스를 직접 저장하는 것은 권장하지 않습니다. 특히 신뢰할 수 없는 출처의 클래스 인스턴스는 역직렬화 시 보안 위험을 초래할 수 있습니다.

