# think-cache

think-cache는 thinkphp 프레임워크에서 추출한 컴포넌트로, 커넥션 풀 기능이 추가되었습니다. 코루틴 및 비코루틴 환경을 자동으로 지원합니다.

## 설치
`composer require -W webman/think-cache`

설치 후 restart로 재시작해야 합니다 (reload는 효과 없음)

### 설정 파일

설정 파일은 `config/think-cache.php` 입니다

## 사용법

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
  class UserController
  {
      public function db(Request $request)
      {
          $key = 'test_key';
          Cache::set($key, rand());
          return response(Cache::get($key));
      }
  }
  ```
## 제공 API
```php
// 캐시 설정
Cache::set('val','value',600);
// 캐시 존재 여부 확인
Cache::has('val');
// 캐시 조회
Cache::get('val');
// 캐시 삭제
Cache::delete('val');
// 캐시 비우기
Cache::clear();
// 읽은 후 삭제
Cache::pull('val');
// 없으면 쓰기
Cache::remember('val',10);

// 숫자형 캐시 데이터용
// 캐시 1 증가
Cache::inc('val');
// 캐시 5 증가
Cache::inc('val',5);
// 캐시 1 감소
Cache::dec('val');
// 캐시 5 감소
Cache::dec('val',5);

// 캐시 태그 사용
Cache::tag('tag_name')->set('val','value',600);
// 특정 태그의 캐시 삭제
Cache::tag('tag_name')->clear();
// 여러 태그 지정 지원
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// 여러 태그의 캐시 삭제
Cache::tag(['tag1','tag2'])->clear();

// 다양한 캐시 스토어 사용
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


