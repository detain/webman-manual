# 업그레이드 방법

`composer require workerman/webman-framework ^1.4.3 && composer require webman/console ^1.0.27 && php webman install`

> **참고**
> Alibaba Cloud Composer 프록시가 Composer 공식 소스와의 데이터 동기화를 중단하여, 현재 Alibaba Cloud Composer 프록시로 최신 webman으로 업그레이드할 수 없습니다. 아래 명령어를 실행하여 Composer 공식 데이터 소스로 복원하세요: `composer config -g --unset repos.packagist`
