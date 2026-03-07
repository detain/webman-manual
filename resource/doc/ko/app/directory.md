# 디렉터리 구조

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

애플리케이션 플러그인은 webman과 동일한 디렉터리 구조와 설정 파일을 갖습니다. 실무에서는 일반 webman 애플리케이션을 개발하는 것과 거의 동일한 개발 경험을 제공합니다.

플러그인 디렉터리와 명명 규칙은 PSR-4 규격을 따릅니다. 플러그인이 모두 `plugin` 디렉터리에 위치하므로 네임스페이스는 모두 `plugin`으로 시작합니다. 예: `plugin\foo\app\controller\UserController`.

## api 디렉터리 설명

각 플러그인에는 `api` 디렉터리가 있습니다. 다른 애플리케이션에서 호출할 내부 인터페이스를 제공하려면 해당 인터페이스를 `api` 디렉터리에 둡니다.

참고: 여기서 말하는 인터페이스는 함수 호출 인터페이스이며, 네트워크/HTTP 인터페이스가 아닙니다.

예를 들어 메일 플러그인은 `plugin/email/api/Email.php`에 `Email::send()` 인터페이스를 제공하여 다른 애플리케이션에서 메일을 보낼 때 호출합니다. 또한 `plugin/email/api/Install.php`는 webman-admin 플러그인 마켓에서 설치·제거 작업을 실행하기 위해 자동으로 생성됩니다.
