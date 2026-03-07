# 캐시

[webman/cache](https://github.com/webman-php/cache)는 [symfony/cache](https://github.com/symfony/cache)를 기반으로 한 캐시 구성 요소로, 코루틴 및 비코루틴 환경 모두 지원하며 연결 풀을 지원합니다.

## 설치

```php
composer require -W webman/cache
```

## 예제
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

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

## 설정 파일 위치
설정 파일은 `config/cache.php`에 있습니다. 없으면 수동으로 생성하세요.

## 설정 파일 내용
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver`는 **file**, **redis**, **array**, **apcu** 네 가지 드라이버를 지원합니다.

#### file 드라이버
기본 드라이버입니다. 외부 의존성이 없습니다. 프로세스 간 캐시 공유를 지원합니다. 여러 서버 간 공유는 지원하지 않습니다.

#### array 드라이버
메모리 저장소로 성능이 가장 좋지만 메모리를 소비합니다. 프로세스 및 서버 간 공유를 지원하지 않습니다. 프로세스 재시작 시 데이터가 사라집니다. 캐시 데이터량이 적은 프로젝트에 적합합니다.

#### apcu 드라이버
메모리 저장소입니다. 성능은 array에 이어 두 번째입니다. 프로세스 간 캐시 공유를 지원합니다. 여러 서버 간 공유는 지원하지 않습니다. 프로세스 재시작 시 데이터가 사라집니다. 캐시 데이터량이 적은 프로젝트에 적합합니다.

> [APCu 확장 기능](https://pecl.php.net/package/APCu) 설치 및 활성화가 필요합니다. 캐시 쓰기/삭제가 빈번한 시나리오에는 권장하지 않으며, 성능이 눈에 띄게 저하될 수 있습니다.

#### redis 드라이버
[webman/redis](./redis.md) 구성 요소에 의존합니다. 프로세스 및 서버 간 캐시 공유를 지원합니다.

**stores.redis.connection**

`stores.redis.connection`은 `config/redis.php`에 정의된 키에 해당합니다. Redis 사용 시 연결 풀 설정을 포함한 `webman/redis` 설정을 재사용합니다.

**`config/redis.php`에 캐시 전용 설정을 추가하는 것을 권장합니다. 예:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== 새로 추가
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

그다음 `stores.redis.connection`을 `cache`로 설정합니다. 최종 `config/cache.php`는 다음과 같습니다:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## 스토어 전환
다른 드라이버를 사용하려면 다음과 같이 수동으로 스토어를 전환할 수 있습니다:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **참고**
> 캐시 키 이름은 [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions)에 의해 제한되며 `{}()/\@:` 문자를 포함할 수 없습니다. `symfony/cache` 7.2.4부터 PHP ini 옵션 `zend.assertions=-1`로 이 검사를 우회할 수 있습니다.

## 다른 캐시 구성 요소 사용

[ThinkCache](https://github.com/webman-php/think-cache) 구성 요소는 [다른 데이터베이스](others.md#ThinkCache)를 참고하세요.
