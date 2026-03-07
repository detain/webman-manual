# 프로세스 모니터링
webman에는 기본 제공되는 monitor 프로세스가 있으며, 두 가지 기능을 지원합니다.
1. 파일 업데이트를 모니터링하고 새로운 비즈니스 코드를 자동으로 리로드합니다(일반적으로 개발 시 사용).
2. 모든 프로세스의 메모리 사용량을 모니터링하며, 프로세스가 `php.ini`의 `memory_limit`을 초과하기 직전이면 해당 프로세스를 자동으로 안전하게 재시작합니다(비즈니스에 영향 없음).

## 모니터링 구성
`config/process.php`의 `monitor` 구성:
```php

global $argv;

return [
    // 파일 업데이트 감지 및 자동 리로드
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // 이 디렉터리를 모니터링
            'monitorDir' => array_merge([    // 모니터링할 디렉터리의 파일
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // 다음 확장자를 가진 파일을 모니터링
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // 파일 모니터링 활성화 여부
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // 메모리 모니터링 활성화 여부
            ]
        ]
    ]
];
```
`monitorDir`은 업데이트를 모니터링할 디렉터리를 구성합니다(모니터링하는 파일 수가 너무 많으면 안 됨).
`monitorExtensions`은 `monitorDir` 디렉터리에서 모니터링할 파일 확장자를 구성합니다.
`options.enable_file_monitor`가 `true`이면 파일 업데이트 모니터링이 활성화됩니다(Linux에서 디버그 모드로 실행 시 기본 활성화).
`options.enable_memory_monitor`가 `true`이면 메모리 사용량 모니터링이 활성화됩니다(Windows에서는 지원되지 않음).

> **팁**
> Windows에서는 파일 업데이트 모니터링을 활성화하려면 `windows.bat` 또는 `php windows.php`를 실행해야 합니다.
