# 디렉토리 구조
```
.
├── app                           애플리케이션 디렉토리
│   ├── controller                컨트롤러 디렉토리
│   ├── model                     모델 디렉토리
│   ├── view                      뷰 디렉토리
│   ├── middleware                미들웨어 디렉토리
│   │   └── StaticFile.php        내장 정적 파일 미들웨어
│   ├── process                   사용자 정의 프로세스 디렉토리
│   │   ├── Http.php              Http 프로세스
│   │   └── Monitor.php           모니터 프로세스
│   └── functions.php             비즈니스 맞춤 함수는 이 파일에 작성
├── config                        설정 디렉토리
│   ├── app.php                   애플리케이션 설정
│   ├── autoload.php              여기에 구성된 파일은 자동으로 로드됨
│   ├── bootstrap.php             프로세스 시작 시 onWorkerStart 콜백 구성
│   ├── container.php             컨테이너 설정
│   ├── dependence.php            컨테이너 의존성 구성
│   ├── database.php              데이터베이스 설정
│   ├── exception.php             예외 설정
│   ├── log.php                   로그 설정
│   ├── middleware.php            미들웨어 설정
│   ├── process.php               사용자 정의 프로세스 구성
│   ├── redis.php                 Redis 구성
│   ├── route.php                 라우트 구성
│   ├── server.php                포트, 프로세스 수 등 서버 설정
│   ├── view.php                  뷰 설정
│   ├── static.php                정적 파일 스위치 및 정적 파일 미들웨어 구성
│   ├── translation.php           다국어 설정
│   └── session.php               세션 설정
├── public                        정적 리소스 디렉토리
├── runtime                       애플리케이션 실행 시간 디렉토리, 쓰기 권한 필요
├── start.php                     서비스 시작 파일
├── vendor                        Composer로 설치된 서드파티 라이브러리 디렉토리
└── support                       라이브러리 어댑터(서드파티 라이브러리 포함)
    ├── Request.php               요청 클래스
    ├── Response.php              응답 클래스
    ├── Setup.php                 설치 마법사 스크립트
    └── bootstrap.php             프로세스 시작 후 초기화 스크립트
```
