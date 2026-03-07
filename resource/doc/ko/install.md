# webman 설치 방법

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: PHP + Composer 환경 설치 (이미 환경이 있으면 건너뛰기)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **참고**
> 위 명령어는 Linux/Mac 시스템에 적용됩니다. Windows 사용자는 별도로 PHP 환경을 설치해야 합니다.

webman 공식에서 제공하는 [정적 PHP](https://www.workerman.net/download)를 수동으로 다운로드하여 압축을 풀어 사용할 수도 있습니다.

## 1. 프로젝트 생성

```php
composer create-project workerman/webman:~2.0
```

> **팁**
> 오류가 발생하면 문제가 있는 Composer 미러를 사용 중일 수 있습니다. `composer config -g --unset repos.packagist` 를 실행하여 프록시를 해제하세요.

## 2. 실행

webman 디렉토리로 이동

#### Windows 사용자
`windows.bat` 을 더블클릭하거나 `php windows.php` 를 실행하여 시작

> **팁**
> 오류가 발생하면 일부 함수가 비활성화되어 있을 수 있습니다. [비활성화된 함수 확인](others/disable-function-check.md) 을 참조하여 해제하세요.

#### Linux 사용자
**디버그 모드** (개발/디버깅용: 출력이 터미널에 표시되며, 터미널 종료 시 webman 서비스도 함께 종료됨)

```php
php start.php start
```

**데몬 모드** (운영 환경용: 출력이 터미널에 표시되지 않으며, 터미널 종료 후에도 webman 서비스가 계속 실행됨)

```php
php start.php start -d
```

#### Docker 사용자

모든 서비스를 시작하고 콘솔에 연결
```php
docker-compose up
```

백그라운드 모드로 서비스 실행
```php
docker-compose up -d
```

> **팁**
> 오류가 발생하면 일부 함수가 비활성화되어 있을 수 있습니다. [비활성화된 함수 확인](others/disable-function-check.md) 을 참조하여 해제하세요.

## 3. 접속

브라우저에서 `http://IP주소:8787` 에 접속하세요.
