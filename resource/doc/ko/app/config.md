# 설정 파일

플러그인 설정은 일반적인 webman 프로젝트와 동일하게 동작하지만, 일반적으로 해당 플러그인에만 적용되며 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.app.controller_suffix` 값은 플러그인의 컨트롤러 접미사에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.app.controller_reuse` 값은 플러그인이 컨트롤러를 재사용하는지 여부에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.middleware` 값은 플러그인의 미들웨어에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.view` 값은 플러그인이 사용하는 뷰에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.container` 값은 플러그인이 사용하는 컨테이너에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.
예를 들어 `plugin.foo.exception` 값은 플러그인의 예외 처리 클래스에만 영향을 주며, 메인 프로젝트에는 영향을 주지 않습니다.

다만 라우팅은 전역이므로, 플러그인이 설정한 라우트도 전역 라우팅에 영향을 줍니다.

## 설정 가져오기
플러그인 설정을 가져오려면 `config('plugin.{plugin}.{구체적_설정}');` 를 사용합니다. 예를 들어 `plugin/foo/config/app.php` 의 모든 설정을 가져오려면 `config('plugin.foo.app')` 를 사용합니다.
마찬가지로, 메인 프로젝트나 다른 플러그인에서도 `config('plugin.foo.xxx')` 로 foo 플러그인의 설정을 가져올 수 있습니다.

## 지원되지 않는 설정
애플리케이션 플러그인은 `server.php`, `session.php` 설정 및 `app.request_class`, `app.public_path`, `app.runtime_path` 설정을 지원하지 않습니다.
