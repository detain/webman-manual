# การใช้ Transaction อย่างถูกต้อง

การใช้งาน transaction ของฐานข้อมูลใน webman เหมือนกับใน framework อื่นๆ ที่นี่เป็นจุดที่ควรใส่ใจ

## โครงสร้างโค้ด

โครงสร้างโค้ดเหมือนกับ framework อื่นๆ (เช่น Laravel think-orm คล้ายกัน):

```php
Db::beginTransaction();
try {
    // ..งานประมวลผลทางธุรกิจละไว้...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**สำคัญ:** ต้องใช้ `\Throwable` และห้ามใช้ `\Exception` เพราะระหว่างการประมวลผลอาจเกิด `Error` ได้ และ `Error` ไม่สืบทอดจาก `Exception`

## การเชื่อมต่อฐานข้อมูล

เมื่อทำงานกับ model ใน transaction ให้ตรวจว่า model กำหนดการเชื่อมต่อหรือไม่ ถ้า model กำหนดการเชื่อมต่อแล้ว เวลาเริ่ม transaction ต้องระบุการเชื่อมต่อนั้น มิฉะนั้น transaction จะไม่ทำงาน (think-orm คล้ายกัน) เช่น:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // กำหนดการเชื่อมต่อให้ model ตรงนี้
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

เมื่อ model กำหนดการเชื่อมต่อแล้ว การเริ่ม transaction commit และ rollback ต้องระบุการเชื่อมต่อทั้งสิ้น:

```php
Db::connection('mysql')->beginTransaction();
try {
    // การประมวลผลทางธุรกิจ
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## การค้นหา Request ที่ Transaction ยังไม่ได้ commit
บางครั้ง bug ในโค้ดธุรกิจทำให้ request บางอัน transaction ยังไม่ได้ commit เพื่อหาว่า method ของ controller ไหนยังไม่ได้ commit transaction ให้เร็วขึ้น สามารถติดตั้ง component `webman/log` ได้ component นี้จะตรวจสอบอัตโนมัติหลัง request เสร็จว่ามี transaction ที่ยังไม่ได้ commit หรือไม่ แล้วบันทึกลง log โดยคีย์เวิร์ดใน log คือ `Uncommitted transactions`

**วิธีติดตั้ง webman/log**

`composer require webman/log`

> **หมายเหตุ**
> หลังติดตั้งต้อง restart เพื่อเริ่มใหม่ ส่วน reload จะไม่มีผล
