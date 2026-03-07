# 데이터베이스 트랜잭션 올바른 사용법
webman에서 데이터베이스 트랜잭션 사용 방식은 다른 프레임워크와 동일합니다. 여기서 주의할 점을 정리합니다.

## 코드 구조

코드 구조는 다른 프레임워크(Laravel, think-orm 등)와 동일합니다:

```php
Db::beginTransaction();
try {
    // ..비즈니스 로직 생략...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

특히 주의할 점은 `\Exception`이 아니라 **반드시** `\Throwable`을 사용해야 한다는 것입니다. 비즈니스 처리 중 `Error`가 발생할 수 있으며, `Error`는 `Exception`을 상속하지 않기 때문입니다.

## 데이터베이스 연결

트랜잭션 내에서 모델을 다룰 때는 모델에 연결이 설정되어 있는지 특히 확인해야 합니다. 모델에 연결이 설정된 경우, 트랜잭션 시작 시 해당 연결을 지정해야 하며, 지정하지 않으면 트랜잭션이 적용되지 않습니다(think-orm도 동일). 예:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // 모델에 연결 지정
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

모델에 연결이 지정된 경우, 트랜잭션 시작·커밋·롤백 모두에서 연결을 지정해야 합니다:

```php
Db::connection('mysql')->beginTransaction();
try {
    // 비즈니스 처리
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## 커밋되지 않은 트랜잭션 찾기
비즈니스 코드 버그로 인해 특정 요청의 트랜잭션이 커밋되지 않는 경우가 있습니다. 커밋되지 않은 트랜잭션이 있는 컨트롤러 메서드를 빠르게 찾으려면 `webman/log` 컴포넌트를 설치할 수 있습니다. 이 컴포넌트는 요청 완료 후 미커밋 트랜잭션을 자동으로 검사하고 로그에 기록합니다. 로그 키워드는 `Uncommitted transactions`입니다.

**webman/log 설치 방법**

`composer require webman/log`

> **주의**
> 설치 후 반드시 restart로 재시작해야 합니다. reload로는 적용되지 않습니다.
