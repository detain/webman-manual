# 데이터베이스 설정(Laravel 스타일)
webman/database가 지원하는 데이터베이스 및 버전은 다음과 같습니다:

- MySQL 5.6+
- PostgreSQL 9.4+
- SQLite 3.8.8+
- SQL Server 2017+

데이터베이스 설정 파일은 `config/database.php`에 위치합니다.

```php
return [
    // 기본 데이터베이스
    'default' => 'mysql',
    // 각종 데이터베이스 설정
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
            'pool' => [ // 연결 풀 설정, swoole/swow 드라이버만 지원
                'max_connections' => 5, // 최대 연결 수
                'min_connections' => 1, // 최소 연결 수
                'wait_timeout' => 3,    // 풀에서 연결 획득 대기 최대 시간, 초과 시 예외 발생
                'idle_timeout' => 60,   // 풀 내 연결의 최대 유휴 시간, 초과 시 회수하여 min_connections까지 감소
                'heartbeat_interval' => 50, // 하트비트 간격(초), 60초 미만 권장
            ],
         ],
         
         'sqlite' => [
             'driver'   => 'sqlite',
             'database' => '',
             'prefix'   => '',
             'pool' => [ // 연결 풀 설정, swoole/swow 드라이버만 지원
                'max_connections' => 5, // 최대 연결 수
                'min_connections' => 1, // 최소 연결 수
                'wait_timeout' => 3,    // 풀에서 연결 획득 대기 최대 시간, 초과 시 예외 발생
                'idle_timeout' => 60,   // 풀 내 연결의 최대 유휴 시간, 초과 시 회수하여 min_connections까지 감소
                'heartbeat_interval' => 50, // 하트비트 간격(초), 60초 미만 권장
            ],
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
             'pool' => [ // 연결 풀 설정, swoole/swow 드라이버만 지원
                'max_connections' => 5, // 최대 연결 수
                'min_connections' => 1, // 최소 연결 수
                'wait_timeout' => 3,    // 풀에서 연결 획득 대기 최대 시간, 초과 시 예외 발생
                'idle_timeout' => 60,   // 풀 내 연결의 최대 유휴 시간, 초과 시 회수하여 min_connections까지 감소
                'heartbeat_interval' => 50, // 하트비트 간격(초), 60초 미만 권장
            ],
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
             'pool' => [ // 연결 풀 설정, swoole/swow 드라이버만 지원
                'max_connections' => 5, // 최대 연결 수
                'min_connections' => 1, // 최소 연결 수
                'wait_timeout' => 3,    // 풀에서 연결 획득 대기 최대 시간, 초과 시 예외 발생
                'idle_timeout' => 60,   // 풀 내 연결의 최대 유휴 시간, 초과 시 회수하여 min_connections까지 감소
                'heartbeat_interval' => 50, // 하트비트 간격(초), 60초 미만 권장
            ],
         ],
     ],
 ];
```

## 여러 데이터베이스 사용
`Db::connection('설정명')`으로 사용할 데이터베이스를 선택합니다. `설정명`은 설정 파일 `config/database.php`의 해당 설정 `key`입니다.

예를 들어 다음과 같은 데이터베이스 설정인 경우:

```php
 return [
     // 기본 데이터베이스
     'default' => 'mysql',
     // 각종 데이터베이스 설정
     'connections' => [

         'mysql' => [
             'driver'      => 'mysql',
             'host'        =>   '127.0.0.1',
             'port'        => 3306,
             'database'    => 'webman',
             'username'    => 'webman',
             'password'    => '',
             'unix_socket' =>  '',
             'charset'     => 'utf8',
             'collation'   => 'utf8_unicode_ci',
             'prefix'      => '',
             'strict'      => true,
             'engine'      => null,
         ],
         
         'mysql2' => [
              'driver'      => 'mysql',
              'host'        => '127.0.0.1',
              'port'        => 3306,
              'database'    => 'webman2',
              'username'    => 'webman2',
              'password'    => '',
              'unix_socket' => '',
              'charset'     => 'utf8',
              'collation'   => 'utf8_unicode_ci',
              'prefix'      => '',
              'strict'      => true,
              'engine'      => null,
         ],
         'pgsql' => [
              'driver'   => 'pgsql',
              'host'     => '127.0.0.1',
              'port'     =>  5432,
              'database' => 'webman',
              'username' =>  'webman',
              'password' => '',
              'charset'  => 'utf8',
              'prefix'   => '',
              'schema'   => 'public',
              'sslmode'  => 'prefer',
          ],
 ];
```

이렇게 데이터베이스를 전환합니다.
```php
// 기본 데이터베이스 사용, Db::connection('mysql')->table('users')->where('name', 'John')->first(); 와 동일
$users = Db::table('users')->where('name', 'John')->first(); 
// mysql2 사용
$users = Db::connection('mysql2')->table('users')->where('name', 'John')->first();
// pgsql 사용
$users = Db::connection('pgsql')->table('users')->where('name', 'John')->first();
```
