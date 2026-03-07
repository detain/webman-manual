# 기본 플러그인 생성 및 배포 프로세스

## 원리
1. cross-origin 플러그인을 예로 들면, 플러그인은 세 부분으로 구성됩니다: cross-origin 미들웨어 파일, 설정 파일 `middleware.php`, 그리고 명령으로 자동 생성되는 `Install.php`.
2. 이 세 파일을 패키징하여 Composer로 배포하는 명령을 사용합니다.
3. 사용자가 Composer로 cross-origin 플러그인을 설치하면, `Install.php`가 미들웨어 파일과 설정을 `{주 프로젝트}/config/plugin`에 복사하여 webman이 로드하고, cross-origin 설정을 자동으로 적용합니다.
4. 사용자가 Composer로 플러그인을 제거하면, `Install.php`가 해당 미들웨어 및 설정 파일을 삭제하여 플러그인을 자동으로 제거합니다.

## 규격
1. 플러그인 이름은 `vendor`와 `플러그인 이름`으로 구성됩니다. 예: `webman/push`. Composer 패키지 이름과 대응합니다.
2. 플러그인 설정 파일은 `config/plugin/vendor/플러그인 이름/`에 둡니다(console 명령이 설정 디렉터리를 자동 생성). 설정이 필요 없는 경우 자동 생성된 디렉터리는 삭제해야 합니다.
3. 플러그인 설정 디렉터리는 다음만 지원합니다: `app.php`(메인 설정), `bootstrap.php`(프로세스 부트스트랩), `route.php`(라우트), `middleware.php`(미들웨어), `process.php`(커스텀 프로세스), `database.php`(DB), `redis.php`(Redis), `thinkorm.php`(thinkorm). webman이 자동으로 인식합니다.
4. 설정 접근: `config('plugin.vendor.플러그인 이름.설정 파일.항목');`, 예: `config('plugin.webman.push.app.app_key')`.
5. 플러그인 전용 DB 설정이 있는 경우: `illuminate/database`는 `Db::connection('plugin.vendor.플러그인 이름.연결')`, `thinkorm`는 `Db::connect('plugin.vendor.플러그인 이름.연결')`을 사용합니다.
6. 플러그인이 `app/`에 비즈니스 파일을 두는 경우, 주 프로젝트 및 다른 플러그인과 충돌하지 않아야 합니다.
7. 플러그인은 주 프로젝트로 파일이나 디렉터리를 복사하지 않는 것이 좋습니다. 예: cross-origin 플러그인은 설정만 복사하고, 미들웨어 파일은 `vendor/webman/cros/src`에 두면 됩니다.
8. 플러그인 네임스페이스는 PascalCase 사용을 권장합니다. 예: `Webman/Console`.

## 예시

**`webman/console` 명령줄 설치**

`composer require webman/console`

### 플러그인 생성

생성할 플러그인 이름을 `foo/admin`으로 가정합니다(Composer로 배포할 프로젝트 이름, 소문자여야 함). 실행:

`php webman plugin:create --name=foo/admin`

`vendor/foo/admin`(플러그인 파일)과 `config/plugin/foo/admin`(설정)이 생성됩니다.

> 참고
> `config/plugin/foo/admin`은 `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`를 지원합니다. webman과 동일한 형식, 자동 병합됩니다.
> 접근 시 `plugin`을 접두어로 사용합니다. 예: `config('plugin.foo.admin.app')`.


### 플러그인 내보내기

개발이 끝나면 다음을 실행하여 내보냅니다:

`php webman plugin:export --name=foo/admin`

내보내기

> 설명
> 내보내면 `config/plugin/foo/admin`이 `vendor/foo/admin/src`로 복사되고 `Install.php`가 생성됩니다. `Install.php`는 설치 및 제거 시 실행됩니다.
> 기본 설치: `vendor/foo/admin/src`의 설정을 프로젝트의 `config/plugin`으로 복사합니다.
> 기본 제거: 프로젝트의 `config/plugin`에서 해당 플러그인 설정 파일을 삭제합니다.
> `Install.php`를 수정하여 설치/제거 시 원하는 로직을 추가할 수 있습니다.

### 플러그인 제출
* [GitHub](https://github.com)와 [Packagist](https://packagist.org) 계정이 있다고 가정합니다.
* [GitHub](https://github.com)에 `admin` 저장소를 만들고 코드를 푸시합니다. 예: `https://github.com/사용자명/admin`.
* `https://github.com/사용자명/admin/releases/new`로 이동해 릴리스(예: `v1.0.0`)를 만듭니다.
* [Packagist](https://packagist.org)에서 `Submit`을 클릭하고 `https://github.com/사용자명/admin`을 제출하면 플러그인이 배포됩니다.

> **팁**
> Packagist에서 이름 충돌이 나면 다른 vendor를 선택하세요. 예: `foo/admin`을 `myfoo/admin`으로 변경.

업데이트 시: 코드를 GitHub에 푸시하고, `https://github.com/사용자명/admin/releases/new`에서 새 릴리스를 만든 뒤, `https://packagist.org/packages/foo/admin`에서 `Update`를 클릭하세요.

## 플러그인에 명령 추가하기
일부 플러그인은 커스텀 명령이 필요합니다. 예: `webman/redis-queue`를 설치하면 `redis-queue:consumer` 명령이 추가됩니다. `php webman redis-queue:consumer send-mail`을 실행하면 `SendMail.php` consumer 클래스를 빠르게 생성해 개발을 돕습니다.

`foo/admin` 플러그인에 `foo-admin:add` 명령을 추가하려면:

### 명령 생성

**`vendor/foo/admin/src/FooAdminAddCommand.php` 파일 생성**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = '명령 설명';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **참고**
> 플러그인 간 명령 충돌을 피하려면 `vendor-plugin:명령` 형식을 사용하세요. 예: `foo/admin`의 모든 명령은 `foo-admin:`을 접두어로 가져야 합니다. 예: `foo-admin:add`.

### 설정 추가
**`config/plugin/foo/admin/command.php` 생성**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // 필요시 추가...
];
```

> **팁**
> `command.php`는 플러그인의 커스텀 명령을 등록합니다. 각 항목은 명령 클래스입니다. `webman/console`이 자동으로 로드합니다. 자세한 내용은 [콘솔 명령](console.md)을 참조하세요.

### 내보내기 실행
`php webman plugin:export --name=foo/admin`을 실행해 플러그인을 내보내고 Packagist에 배포합니다. `foo/admin` 설치 후 `foo-admin:add` 명령을 사용할 수 있습니다. `php webman foo-admin:add jerry`를 실행하면 `Admin add jerry`가 출력됩니다.
