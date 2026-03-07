# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm)은 [top-think/think-orm](https://github.com/top-think/think-orm)을 기반으로 개발된 데이터베이스 컴포넌트로, 연결 풀과 코루틴/비코루틴 환경을 모두 지원합니다.

## think-orm 설치

`composer require -W webman/think-orm`

설치 후 restart(재시작)가 필요합니다(reload는 적용되지 않음).

## 설정 파일

실제 환경에 맞게 설정 파일 `config/think-orm.php`를 수정하세요.

## 문서

https://www.kancloud.cn/manual/think-orm

## 사용법

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## 모델 생성

think-orm 모델은 `support\think\Model`을 상속합니다. 예시는 다음과 같습니다.

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

다음 명령어로 think-orm 기반 모델을 생성할 수도 있습니다.

```
php webman make:model 테이블명
```

> **팁**
> 이 명령어를 사용하려면 `webman/console`이 필요합니다. 설치 명령어: `composer require webman/console ^1.2.13`

> **주의**
> make:model 명령어가 주 프로젝트에서 `illuminate/database`를 사용 중임을 감지하면 think-orm이 아닌 `illuminate/database` 기반 모델 파일을 생성합니다. 이 경우 `tp` 매개변수를 추가하여 think-orm 모델을 강제 생성할 수 있습니다. 예: `php webman make:model 테이블명 tp` (동작하지 않으면 `webman/console`을 업그레이드하세요).
