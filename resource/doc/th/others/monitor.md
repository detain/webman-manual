# การตรวจสอบกระบวนการ
webman มาในรูปแบบกระบวนการตรวจสอบ (monitor) ที่รองรับสองฟังก์ชัน:
1. ตรวจสอบการอัปเดตไฟล์และรีโหลดโค้ดธุรกิจใหม่โดยอัตโนมัติ (มักใช้ระหว่างการพัฒนา)
2. ตรวจสอบการใช้งานหน่วยความจำของกระบวนการทั้งหมด หากกระบวนการใดใกล้เกินขีดจำกัด `memory_limit` ใน `php.ini` จะรีสตาร์ทกระบวนการนั้นโดยอัตโนมัติอย่างปลอดภัย (ไม่กระทบการทำงาน)

## การตั้งค่าการตรวจสอบ
การตั้งค่า `monitor` ใน `config/process.php`:
```php

global $argv;

return [
    // การตรวจจับการอัปเดตไฟล์และรีโหลดอัตโนมัติ
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // ตรวจสอบไดเรกทอรีเหล่านี้
            'monitorDir' => array_merge([    // ไดเรกทอรีใดที่ต้องตรวจสอบไฟล์
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // ไฟล์ที่มีนามสกุลเหล่านี้จะถูกตรวจสอบ
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // เปิดการตรวจสอบไฟล์
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // เปิดการตรวจสอบหน่วยความจำ
            ]
        ]
    ]
];
```
`monitorDir` ใช้กำหนดไดเรกทอรีที่ต้องตรวจสอบการอัปเดต (ไม่ควรมีไฟล์มากเกินไปในไดเรกทอรีที่ตรวจสอบ)
`monitorExtensions` ใช้กำหนดนามสกุลไฟล์ที่ต้องตรวจสอบในไดเรกทอรี `monitorDir`
เมื่อ `options.enable_file_monitor` เป็น `true` จะเปิดการตรวจสอบการอัปเดตไฟล์ (บน Linux จะเปิดโดยค่าเริ่มต้นเมื่อรันในโหมด debug)
เมื่อ `options.enable_memory_monitor` เป็น `true` จะเปิดการตรวจสอบหน่วยความจำ (ไม่รองรับบน Windows)

> **เคล็ดลับ**
> บน Windows การตรวจสอบการอัปเดตไฟล์จะเปิดได้ก็ต่อเมื่อรัน `windows.bat` หรือ `php windows.php` เท่านั้น
