# 로그
로그 클래스의 사용법은 데이터베이스 사용법과 같습니다
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

메인 프로젝트의 로그 설정을 재사용하려면 직접 다음과 같이 사용합니다
```php
use support\Log;
Log::info('로그 내용');
// 메인 프로젝트에 test 로그 설정이 있다고 가정
Log::channel('test')->info('로그 내용');
```
