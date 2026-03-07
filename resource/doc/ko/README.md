# webman이란

webman은 Workerman을 기반으로 구축된 고성능 서비스 프레임워크로, HTTP, WebSocket, TCP, UDP 등 다양한 모듈을 통합합니다. 상주 메모리, 코루틴, 커넥션 풀과 같은 기술을 통해 webman은 기존 PHP의 성능 병목을 극복할 뿐 아니라 적용 범위를 크게 확장했습니다.

또한 webman은 강력한 플러그인 메커니즘을 제공하여 개발자가 다른 개발자가 만든 기능 모듈을 빠르게 통합하고 재사용할 수 있게 합니다. 웹사이트 구축, HTTP API 개발, 인스턴트 메시징, IoT 시스템, 게임, TCP/UDP 서비스, Unix Socket 서비스 등 모든 용도에 대응하며, 뛰어난 성능과 유연성을 제공합니다.

# webman의 철학
**최소한의 코어로 최대의 확장성과 최고의 성능을 제공한다.**

webman은 핵심 기능(라우팅, 미들웨어, 세션, 커스텀 프로세스 인터페이스)만 제공하고, 나머지는 모두 Composer 생태계를 재사용합니다. 따라서 webman에서 익숙한 컴포넌트를 그대로 사용할 수 있습니다. 예를 들어 데이터베이스는 Laravel의 [illuminate/database](./db/tutorial.md), ThinkPHP의 [ThinkORM](./db/thinkorm.md), 또는 `Medoo` 등을 선택할 수 있으며, webman에 통합하는 것도 매우 쉽습니다.

# webman의 특징

1. 높은 안정성. webman은 workerman 기반이며, workerman은 업계에서 버그가 적고 안정적인 소켓 프레임워크로 알려져 있습니다.

2. 초고성능. webman은 기존 php-fpm 프레임워크보다 10~100배, Go의 gin·echo보다는 약 2배 정도 성능이 높습니다.

3. 높은 재사용성. 기존 Composer 생태계를 수정 없이 그대로 재사용할 수 있습니다.

4. 높은 확장성. 커스텀 프로세스를 지원하며, workerman으로 할 수 있는 모든 작업을 수행할 수 있습니다.

5. 매우 간단하고 사용이 쉬우며, 학습 비용이 낮고, 코드 작성 방식은 기존 프레임워크와 동일합니다.

6. [바이너리 패키징](./others/bin.md)을 지원하여 PHP 환경 없이 직접 실행할 수 있습니다.

7. 가장 완화되고 친화적인 MIT 오픈소스 라이선스를 사용합니다.

# 프로젝트 링크
GitHub: https://github.com/walkor/webman **별을 눌러 주세요!**

Gitee: https://gitee.com/walkor/webman **별을 눌러 주세요!**

# 제3자 벤치마크 데이터

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

데이터베이스 쿼리 작업 시 webman은 단일 머신에서 최대 39만 QPS를 달성하며, 기존 php-fpm 아키텍처의 Laravel 프레임워크 대비 약 80배 높습니다.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

데이터베이스 쿼리 작업 시 webman은 유사한 Go 웹 프레임워크 대비 약 2배의 성능을 보입니다.

위 데이터는 [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)에서 가져왔습니다.
