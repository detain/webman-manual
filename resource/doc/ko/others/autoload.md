# 자동 로딩

## Composer로 PSR-0 규격 파일 로드하기
webman은 `PSR-4` 자동 로딩 규격을 따릅니다. 프로젝트에서 PSR-0 규격 라이브러리를 로드해야 한다면 다음 절차를 따르세요.

- `extend` 디렉터리를 만들어 PSR-0 규격 라이브러리를 저장합니다
- `composer.json`을 편집하여 `autoload`에 다음 내용을 추가합니다:

```json
"psr-0" : {
    "": "extend/"
}
```
최종 결과는 다음과 비슷합니다:
![](../../assets/img/psr0.png)

- `composer dumpautoload` 실행
- `php start.php restart` 실행으로 webman 재시작 (참고: 변경 사항 적용을 위해 재시작이 필요합니다)

## Composer로 특정 파일 로드하기

- `composer.json`을 편집하여 `autoload.files`에 로드할 파일을 추가합니다:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` 실행
- `php start.php restart` 실행으로 webman 재시작 (참고: 변경 사항 적용을 위해 재시작이 필요합니다)

> **참고**
> composer.json의 `autoload.files`에 설정된 파일은 webman 시작 전에 로드됩니다. 프레임워크 `config/autoload.php`로 로드하는 파일은 webman 시작 후에 로드됩니다.
> composer.json의 `autoload.files` 파일 변경을 적용하려면 restart가 필요하며, reload로는 적용되지 않습니다. `config/autoload.php`로 로드하는 파일은 핫 리로드를 지원하므로 변경 후 reload로 적용됩니다.

## 프레임워크로 특정 파일 로드하기
일부 파일은 PSR 규격에 맞지 않아 자동 로드할 수 없습니다. `config/autoload.php`를 설정해 해당 파일을 로드할 수 있습니다. 예:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **참고**
 > `autoload.php`에 `support/Request.php`와 `support/Response.php` 로드가 설정되어 있습니다. `vendor/workerman/webman-framework/src/support/`에도 동일한 파일이 있기 때문입니다. `autoload.php`로 프로젝트 루트 디렉터리의 파일을 우선 로드하여 `vendor` 내 파일을 수정하지 않고도 이 두 파일을 사용자 정의할 수 있습니다. 사용자 정의가 필요 없다면 이 두 설정을 생략해도 됩니다.
