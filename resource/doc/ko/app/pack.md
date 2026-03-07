# 패키징

예를 들어 foo 애플리케이션 플러그인을 패키징하는 방법:

* `plugin/foo/config/app.php`에서 버전 번호를 설정합니다 (**중요**)
* `plugin/foo`에서 패키징에 불필요한 파일을 삭제합니다. 특히 `plugin/foo/public` 하위의 업로드 기능 테스트용 임시 파일을 삭제합니다
* 프로젝트에 데이터베이스 테이블 생성 등의 작업이 포함된 경우 `plugin/foo/install.sql`을 올바르게 설정합니다. [데이터베이스](database.md) 참조
* 프로젝트에 독립적인 데이터베이스 및 Redis 설정이 있는 경우, 먼저 이러한 설정을 삭제합니다. 이러한 설정은 애플리케이션 첫 방문 시 설치 마법사에서 입력받도록 합니다(직접 구현해야 함). 관리자가 수동으로 입력하고 생성할 수 있도록 합니다
* 프로젝트에 webman admin 백엔드 메뉴가 포함된 경우 `plugin/foo/config/menu.php`를 설정합니다. 플러그인 설치 시 해당 메뉴가 자동으로 설정됩니다. [webman-admin 메뉴 가져오기](https://www.workerman.net/doc/webman-admin/app-development/menu.html) 참조
* 원래 상태로 복원해야 하는 기타 파일을 복원합니다
* 위 작업을 완료한 후 `{주 프로젝트}/plugin/` 디렉터리로 이동합니다
* Linux 사용자: `zip -r foo.zip foo` 명령을 실행하여 foo.zip을 생성합니다
* Windows 사용자: foo 폴더를 마우스 오른쪽 버튼으로 클릭하고 "ZIP 파일로 압축"을 선택하여 foo.zip을 생성합니다

**foo.zip이 패키징된 파일입니다. 다음 장 [플러그인 출시](publish.md)를 참조하세요**
