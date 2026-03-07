# ENV 컴포넌트 vlucas/phpdotenv

## 설명
`vlucas/phpdotenv`는 서로 다른 환경(개발 환경, 테스트 환경 등)의 설정을 구분하기 위한 환경 변수 로드 컴포넌트입니다.

## 프로젝트 주소

https://github.com/vlucas/phpdotenv
  
## 설치
 
```php
composer require vlucas/phpdotenv
 ```
  
## 사용

### 프로젝트 루트 디렉터리에 `.env` 파일 생성
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### 설정 파일 수정
**config/database.php**
```php
return [
    // 기본 데이터베이스
    'default' => 'mysql',

    // 다양한 데이터베이스 설정
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **팁**
> `.env` 파일을 `.gitignore`에 추가하여 코드 저장소에 커밋하지 않기를 권장합니다. 저장소에는 `.env.example` 설정 예시 파일을 두고, 프로젝트 배포 시 `.env.example`을 `.env`로 복사한 뒤 현재 환경에 맞게 설정을 수정하세요. 이렇게 하면 환경마다 다른 설정을 로드할 수 있습니다.

> **주의**
> `vlucas/phpdotenv`는 PHP TS 버전(스레드 세이프 버전)에서 버그가 발생할 수 있으므로 NTS 버전(비스레드 세이프 버전)을 사용하세요. 현재 PHP 버전은 `php -v`로 확인할 수 있습니다.

## 더 알아보기

https://github.com/vlucas/phpdotenv 를 방문하세요.
  
