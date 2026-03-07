# phar 패키징

phar은 PHP에서 JAR와 유사한 패키지 파일로, webman 프로젝트를 하나의 phar 파일로 패키징하여 쉽게 배포할 수 있습니다.

**[fuzqing](https://github.com/fuzqing)의 PR에 대해 매우 감사합니다.**

> **주의**
> `php.ini` 파일의 phar 구성 옵션을 비활성화해야 합니다. 즉, `phar.readonly = 0`으로 설정해야 합니다.

## 명령줄 도구 설치
`composer require webman/console`

## 패키징
webman 프로젝트의 루트 디렉터리에서 `php webman build:phar` 명령을 실행하면 `build` 디렉터리에 `webman.phar` 파일이 생성됩니다.

> 패키징 관련 구성은 `config/plugin/webman/console/app.php` 파일에 있습니다.

## 시작 및 중지 관련 명령
**시작**
`php webman.phar start` 또는 `php webman.phar start -d`

**중지**
`php webman.phar stop`

**상태 확인**
`php webman.phar status`

**연결 상태 확인**
`php webman.phar connections`

**재시작**
`php webman.phar restart` 또는 `php webman.phar restart -d`

## 설명
* 패키징된 프로젝트는 reload를 지원하지 않으며, 코드 업데이트를 위해서는 restart로 재시작해야 합니다.

* 패키지 크기와 메모리 사용량을 줄이기 위해 `config/plugin/webman/console/app.php`의 `exclude_pattern` 및 `exclude_files` 옵션으로 불필요한 파일을 제외할 수 있습니다.

* webman.phar를 실행하면 webman.phar가 있는 디렉터리에 runtime 디렉터리가 생성되어 로그 등의 임시 파일을 저장합니다.

* 프로젝트에서 .env 파일을 사용하는 경우 .env 파일을 webman.phar가 있는 디렉터리에 두어야 합니다.

* 사용자가 업로드한 파일을 phar 패키지 내에 저장하지 마세요. `phar://` 프로토콜로 사용자 업로드 파일을 조작하는 것은 매우 위험합니다(phar 역직렬화 취약점). 사용자 업로드 파일은 phar 패키지 밖의 디스크에 별도로 저장해야 합니다. 아래 참조.

* 업무에서 public 디렉터리에 파일을 업로드해야 하는 경우 public 디렉터리를 webman.phar가 있는 디렉터리 밖으로 독립시켜 배치해야 하며, 이때 `config/app.php`를 구성해야 합니다.
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
`public_path($상대경로)` 헬퍼 함수를 사용하여 실제 public 디렉터리 위치를 찾을 수 있습니다.

* webman.phar는 Windows에서 사용자 정의 프로세스를 지원하지 않습니다.
