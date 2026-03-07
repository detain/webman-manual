# กำหนด 404 เอง

หากคุณต้องการควบคุมเนื้อหา 404 แบบไดนามิก เช่น เมื่อร้องขอ ajax ให้คืนค่า json `{"code:"404", "msg":"404 not found"}` และเมื่อร้องขอหน้าให้คืนค่าเทมเพลต `app/view/404.html` โปรดดูตัวอย่างด้านล่าง

> ตัวอย่างด้านล่างใช้เทมเพลต php แบบเนทีฟ เทมเพลตอื่น ๆ เช่น `twig` `blade` `think-template` ใช้หลักการเดียวกัน

**สร้างไฟล์ `app/view/404.html`**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**ใน `config/route.php` เพิ่มโค้ดดังนี้:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // คืนค่า json เมื่อร้องขอ ajax
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // คืนค่าเทมเพลต 404.html เมื่อร้องขอหน้า
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# กำหนด 405 เอง

ตั้งแต่ webman-framework 1.5.23 คอลแบ็กฟอลแบ็กรองรับพารามิเตอร์ `status` ค่า 404 หมายถึงไม่พบคำขอ 405 หมายถึงไม่รองรับวิธีคำขอปัจจุบัน (เช่น เข้าถึงเส้นทางที่กำหนดด้วย `Route::post()` ผ่าน GET)

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# กำหนด 500 เอง

**สร้าง `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
เทมเพลตข้อผิดพลาดที่กำหนดเอง:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**สร้าง `app/exception/Handler.php`** (หากไม่มีไดเรกทอรีโปรดสร้างเอง)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * แสดงผลและคืนค่า
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // คืนค่า json เมื่อร้องขอ ajax
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // คืนค่าเทมเพลต 500.html เมื่อร้องขอหน้า
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**กำหนดค่า `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
