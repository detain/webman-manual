# 설정 파일

## 위치
webman 설정 파일은 `config/` 디렉토리에 있습니다. 프로젝트에서 `config()` 함수를 사용하여 해당 설정에 접근할 수 있습니다.

## 설정 접근

모든 설정 가져오기:
```php
config();
```

`config/app.php`의 모든 설정 가져오기:
```php
config('app');
```

`config/app.php`의 `debug` 설정 가져오기:
```php
config('app.debug');
```

설정이 배열인 경우 `.`을 사용하여 중첩된 값에 접근할 수 있습니다. 예:
```php
config('file.key1.key2');
```

## 기본값
```php
config($key, $default);
```
두 번째 매개변수로 기본값을 전달합니다. 설정이 존재하지 않으면 기본값이 반환됩니다. 설정이 존재하지 않고 기본값도 설정되지 않은 경우 `null`이 반환됩니다.

## 사용자 설정
개발자는 `config/` 디렉토리에 자체 설정 파일을 추가할 수 있습니다. 예:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**설정 접근 시 사용법**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## 설정 수정
Webman은 동적 설정 변경을 지원하지 않습니다. 모든 설정은 해당 설정 파일에서 수동으로 수정한 후 애플리케이션을 리로드하거나 재시작해야 합니다.

> **참고**
> 서버 설정 `config/server.php` 및 프로세스 설정 `config/process.php`는 리로드를 지원하지 않습니다. 변경 사항을 적용하려면 재시작해야 합니다.

## 중요 안내
`config/` 하위 디렉토리에 설정 파일을 생성하는 경우, 예를 들어 `config/order/status.php`를 사용할 때 `config/order/` 디렉토리에 다음 내용의 `app.php` 파일이 필요합니다:
```php
<?php
return [
    'enable' => true,
];
```
`enable`을 `true`로 설정하면 프레임워크가 이 디렉토리에서 설정을 로드합니다.
설정 디렉토리 구조는 다음과 같아야 합니다:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
그런 다음 `config.order.status`를 통해 `status.php`의 배열 또는 특정 키에 접근할 수 있습니다.


## 설정 파일 참조

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // 수신 포트 (1.6.0부터 제거됨, config/process.php에서 설정)
    'transport' => 'tcp', // 전송 프로토콜 (1.6.0부터 제거됨, config/process.php에서 설정)
    'context' => [], // SSL 등 (1.6.0부터 제거됨, config/process.php에서 설정)
    'name' => 'webman', // 프로세스 이름 (1.6.0부터 제거됨, config/process.php에서 설정)
    'count' => cpu_count() * 4, // 프로세스 수 (1.6.0부터 제거됨, config/process.php에서 설정)
    'user' => '', // 사용자 (1.6.0부터 제거됨, config/process.php에서 설정)
    'group' => '', // 그룹 (1.6.0부터 제거됨, config/process.php에서 설정)
    'reusePort' => false, // 포트 재사용 활성화 (1.6.0부터 제거됨, config/process.php에서 설정)
    'event_loop' => '',  // 이벤트 루프 클래스, 기본적으로 자동 선택
    'stop_timeout' => 2, // stop/restart/reload 신호 수신 시 최대 대기 시간, 시간 내에 프로세스가 종료되지 않으면 강제 종료
    'pid_file' => runtime_path() . '/webman.pid', // PID 파일 위치
    'status_file' => runtime_path() . '/webman.status', // 상태 파일 위치
    'stdout_file' => runtime_path() . '/logs/stdout.log', // stdout 파일 위치, webman 시작 후 모든 출력이 여기에 기록됨
    'log_file' => runtime_path() . '/logs/workerman.log', // Workerman 로그 파일 위치
    'max_package_size' => 10 * 1024 * 1024 // 최대 패킷 크기, 10M. 파일 업로드 크기가 이 값으로 제한됨
];
```

#### app.php
```php
return [
    'debug' => true,  // 디버그 모드, 오류 시 스택 트레이스 등 활성화. 프로덕션에서는 비활성화해야 함
    'error_reporting' => E_ALL, // 오류 보고 수준
    'default_timezone' => 'Asia/Shanghai', // 기본 타임존
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // 퍼블릭 디렉토리 경로
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // 런타임 디렉토리 경로
    'controller_suffix' => 'Controller', // 컨트롤러 접미사
    'controller_reuse' => false, // 컨트롤러 재사용 여부
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman 프로세스 설정
    'webman' => [ 
        'handler' => Http::class, // 프로세스 핸들러 클래스
        'listen' => 'http://0.0.0.0:8787', // 수신 주소
        'count' => cpu_count() * 4, // 프로세스 수, 기본적으로 CPU 4배
        'user' => '', // 프로세스 사용자, 낮은 권한 사용자 사용 권장
        'group' => '', // 프로세스 그룹, 낮은 권한 그룹 사용 권장
        'reusePort' => false, // reusePort 활성화, worker 프로세스 간 연결 분배
        'eventLoop' => '', // 이벤트 루프 클래스, 비어 있으면 server.event_loop 사용
        'context' => [], // 수신 컨텍스트, 예: SSL
        'constructor' => [ // 프로세스 핸들러 생성자 매개변수, 여기서는 Http 클래스
            'requestClass' => Request::class, // 사용자 정의 요청 클래스
            'logger' => Log::channel('default'), // 로거 인스턴스
            'appPath' => app_path(), // 앱 디렉토리 경로
            'publicPath' => public_path() // 퍼블릭 디렉토리 경로
        ]
    ],
    // 파일 변경 자동 리로드 및 메모리 누수 감지를 위한 모니터 프로세스
    'monitor' => [
        'handler' => app\process\Monitor::class, // 핸들러 클래스
        'reloadable' => false, // 이 프로세스는 리로드를 실행하지 않음
        'constructor' => [ // 프로세스 핸들러 생성자 매개변수
            // 감시할 디렉토리, 너무 많으면 감지가 느려짐
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // 변경 감시를 위한 파일 확장자
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // 기타 옵션
            'options' => [
                // 파일 모니터링 활성화, Linux 전용, daemon 모드에서는 기본적으로 비활성화됨
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // 메모리 모니터링 활성화, Linux 전용
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11 의존성 주입 컨테이너 인스턴스 반환
return new Webman\Container;
```

#### dependence.php
```php
// 의존성 주입 컨테이너에 서비스 및 의존성 설정
return [];
```

#### route.php
```php

use support\Route;
// /test 경로에 대한 라우트 정의
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // 기본 뷰 핸들러 클래스
];
```

### autoload.php
```php
// 프레임워크 자동 로드 파일 설정
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// 캐시 설정
return [
    'default' => 'file', // 기본 드라이버
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // 캐시 파일 저장 경로
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Redis 연결 이름, redis.php의 설정 참조
        ],
        'array' => [
            'driver' => 'array' // 메모리 캐시, 재시작 시 초기화됨
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // 기본 데이터베이스
 'default' => 'mysql',
 // 데이터베이스 연결 설정
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // 예외 핸들러 클래스 설정
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // 핸들러
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // 로그 파일 이름
                    7, //$maxFiles // 로그 7일간 보관
                    Monolog\Logger::DEBUG, // 로그 수준
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // 포매터
                    'constructor' => [null, 'Y-m-d H:i:s', true], // 포매터 매개변수
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // 유형
    'type' => 'file', // 또는 redis 또는 redis_cluster
     // 핸들러
    'handler' => FileSessionHandler::class,
     // 설정
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // 저장 디렉토리
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // 세션 이름
    'auto_update_timestamp' => false, // 세션 만료 방지를 위한 타임스탬프 자동 업데이트
    'lifetime' => 7*24*60*60, // 수명
    'cookie_lifetime' => 365*24*60*60, // 쿠키 수명
    'cookie_path' => '/', // 쿠키 경로
    'domain' => '', // 쿠키 도메인
    'http_only' => true, // HTTP 전용
    'secure' => false, // HTTPS 전용
    'same_site' => '', // SameSite 속성
    'gc_probability' => [1, 1000], // 세션 가비지 컬렉션 확률
];
```

#### middleware.php
```php
// 미들웨어 설정
return [];
```

#### static.php
```php
return [
    'enable' => true, // webman 정적 파일 서빙 활성화
    'middleware' => [ // 정적 파일 미들웨어, 캐시 정책, CORS 등용
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // 기본 언어
    'locale' => 'zh_CN',
    // 대체 언어
    'fallback_locale' => ['zh_CN', 'en'],
    // 번역 파일 위치
    'path' => base_path() . '/resource/translations',
];
```
