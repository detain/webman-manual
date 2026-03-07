# 실행 흐름

## 프로세스 시작 흐름

`php start.php start` 실행 후 흐름은 다음과 같습니다:

1. config/ 하위 설정 로드
2. `pid_file`, `stdout_file`, `log_file`, `max_package_size` 등 Worker 관련 설정 구성
3. webman 프로세스 생성 및 포트 감시 (기본값: 8787)
4. 설정에 따라 사용자 정의 프로세스 생성
5. webman 프로세스 및 사용자 정의 프로세스가 시작된 후 다음 로직 실행 (`onWorkerStart` 내에서 모두 실행):
   ① `config/autoload.php`에 설정된 파일 로드 (예: `app/functions.php`)
   ② `config/middleware.php`(`config/plugin/*/*/middleware.php` 포함)에 설정된 미들웨어 로드
   ③ `config/bootstrap.php`(`config/plugin/*/*/bootstrap.php` 포함)에 설정된 클래스의 `start` 메서드 실행하여 Laravel 데이터베이스 연결 초기화 등 모듈 초기화
   ④ `config/route.php`(`config/plugin/*/*/route.php` 포함)에 정의된 라우트 로드

## 요청 처리 흐름

1. 요청 URL이 public 하위 정적 파일에 해당하는지 확인. 해당하면 파일 반환(요청 종료). 해당하지 않으면 2로
2. URL이 특정 라우트와 일치하는지 판단. 일치하지 않으면 3으로, 일치하면 4로
3. 기본 라우트가 비활성화되었는지 확인. 비활성화되었으면 404 반환(요청 종료). 그렇지 않으면 4로
4. 요청에 해당하는 컨트롤러의 미들웨어를 찾아 미들웨어 선처리를 순서대로 실행(양파 모델 요청 단계), 컨트롤러 비즈니스 로직 실행, 미들웨어 후처리 실행(양파 모델 응답 단계) 후 요청 종료. ([미들웨어 양파 모델](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B) 참조)
