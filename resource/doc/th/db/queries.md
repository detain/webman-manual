# การใช้งานฐานข้อมูล (คอมโพเนนต์ Laravel)

## รับข้อมูลทั้งหมด
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function all(Request $request)
    {
        $users = Db::table('users')->get();
        return view('user/all', ['users' => $users]);
    }
}
```

## รับคอลัมน์ที่ระบุ
```php
$users = Db::table('user')->select('name', 'email as user_email')->get();
```

## รับแถวเดียว
```php
$user = Db::table('users')->where('name', 'John')->first();
```

## รับคอลัมน์เดียว
```php
$titles = Db::table('roles')->pluck('title');
```
ระบุค่า id ให้เป็นดัชนี
```php
$roles = Db::table('roles')->pluck('title', 'id');

foreach ($roles as $id => $title) {
    echo $title;
}
```

## รับค่าเดียว (คอลัมน์)
```php
$email = Db::table('users')->where('name', 'John')->value('email');
```

## การเอาค่าที่ซ้ำออก
```php
$email = Db::table('user')->select('nickname')->distinct()->get();
```

## ผลลัพธ์แบ่งเป็นชุด
หากคุณต้องการจัดการกับบันทึกในฐานข้อมูลจำนวนมาก การรับข้อมูลทั้งหมดในครั้งเดียวอาจใช้เวลานานและเป็นแรงงาน และอาจทำให้เกินขีดจำกัดของหน่วยความจำ ในกรณีนี้คุณสามารถพิจารณาใช้เมธอด chunkById โดยเฉพาะ วิธีการนี้จะรับชุดของผลลัพธ์เล็ก ๆ และส่งให้กับ ฟังก์ชันปิด เพื่อการประมวลผล เช่น เราสามารถแบ่งข้อมูลทั้งหมดในตาราง users เป็นชุดขนาด 100 รายการต่อ 1 ชุดการประมวลผล
```php
Db::table('users')->orderBy('id')->chunkById(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```
คุณสามารถใช้คำสั่ง return false ในฟังก์ชันปิดเพื่อหยุดการรับผลลัพธ์เป็นชุดต่อไป
```php
Db::table('users')->orderBy('id')->chunkById(100, function ($users) {
    // Process the records...

    return false;
});
```

> หมายเหตุ: อย่าลบข้อมูลในการเรียกคืนเพราะอาจทำให้บางบันทึกไม่ได้รับการรวมอยู่ในชุดผลลัพธ์

## การรวมเอาตราของ
ตัวบูรณาการค้นหายังมีวิธีการรวมเอาตราของต่าง ๆ เช่นคำสั่ง count, max, min, avg, sum ฯลฯ
```php
$users = Db::table('users')->count();
$price = Db::table('orders')->max('price');
$price = Db::table('orders')->where('finalized', 1)->avg('price');
```

## ตรวจสอบว่ามีบันทึกหรือไม่
```php
return Db::table('orders')->where('finalized', 1)->exists();
return Db::table('orders')->where('finalized', 1)->doesntExist();
```

## สำหรับวิสาหะของตาราง
ตราบบบ บับสำหรับการสร้างการนำข้อมูลเข้า แทนด้วย(Db::raw($value))ใช้สำหรับการสร้าแรงงานค่าตราบบ
```php
$orders = Db::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();

```
การด้หากน้ ใช้ด้ารุกดับเรื่วียื เช้าณ่้ใช้กันประเดินเวียณ่
```php
$orders = Db::table('orders')
                ->select('department', Db::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();

```

## คำสั่ง Join
```php
// join
$users = Db::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();

// leftJoin
$users = Db::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

// rightJoin
$users = Db::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

// crossJoin
$users = Db::table('sizes')
            ->crossJoin('colors')
            ->get();
```

## คำสั่ง Union
```php
$first = Db::table('users')
            ->whereNull('first_name');

$users = Db::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```
## คำสั่ง Where
โครงสร้าง
```php
where($column, $operator = null, $value = null)
```
พารามิเตอร์แรกคือชื่อคอลัมน์ พารามิเตอร์ที่สองคือตัวดำเนินการที่ระบบฐานข้อมูลรองรับอย่างใดอย่างหนึ่ง และพารามิเตอร์สุดท้ายคือค่าที่ต้องเปรียบเทียบกับคอลัมน์นั้น
```php
$users = Db::table('users')->where('votes', '=', 100)->get();

// เมื่อตัวดำเนินการเป็น เท่ากับ คุณสามารถละพารามิเตอร์นี้ได้ ดังนั้นประโยคนี้เหมือนกับประโยคก่อนหน้า
$users = Db::table('users')->where('votes', 100)->get();

$users = Db::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = Db::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = Db::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```

คุณยังสามารถส่งอาร์เรย์เงื่อนไขไปยังฟังก์ชัน where ได้เช่นกัน:
```php
$users = Db::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

เมทอด orWhere และเมทอด where รับพารามิเตอร์เหมือนกัน:

```php
$users = Db::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

คุณสามารถส่งคล็อสเจอร์เป็นอาร์กิวเมนต์ที่หนึ่งให้กับเมทอด orWhere ได้:

```php
// SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
$users = Db::table('users')
            ->where('votes', '>', 100)
            ->orWhere(function($query) {
                $query->where('name', 'Abigail')
                      ->where('votes', '>', 50);
            })
            ->get();
```

เมทอด whereBetween / orWhereBetween ใช้ในการตรวจสอบค่าฟิลด์ว่าอยู่ระหว่างค่าสองค่าที่กำหนด:
```php
$users = Db::table('users')
           ->whereBetween('votes', [1, 100])
           ->get();
```

เมทอด whereNotBetween / orWhereNotBetween ใช้ในการตรวจสอบค่าฟิลด์ว่าไม่อยู่ระหว่างค่าสองค่าที่กำหนด:
```php
$users = Db::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
```

เมทอด whereIn / whereNotIn / orWhereIn / orWhereNotIn ใช้ในการตรวจสอบค่าของฟิลด์ว่าต้องอยู่ในอาร์เรย์ที่กำหนด:
```php
$users = Db::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
```

เมทอด whereNull / whereNotNull / orWhereNull / orWhereNotNull ใช้ในการตรวจสอบว่าฟิลด์ที่กำหนดต้องเป็นค่าว่าง:
```php
$users = Db::table('users')
                    ->whereNull('updated_at')
                    ->get();
```

เมทอด whereNotNull ใช้ในการตรวจสอบว่าฟิลด์ที่กำหนดต้องไม่เป็นค่าว่าง:
```php
$users = Db::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
```

เมทอด whereDate / whereMonth / whereDay / whereYear / whereTime ใช้ในการเปรียบเทียบค่าของฟิลด์กับวันที่ที่กำหนด:
```php
$users = Db::table('users')
                ->whereDate('created_at', '2016-12-31')
                ->get();
```

เมทอด whereColumn / orWhereColumn ใช้ในการเปรียบเทียบค่าของฟิลด์สองคอลัมน์ว่าเท่ากันหรือไม่:

```php
$users = Db::table('users')
                ->whereColumn('first_name', 'last_name')
                ->get();
                
// คุณสามารถส่งตัวดำเนินการไปได้เช่นกัน
$users = Db::table('users')
                ->whereColumn('updated_at', '>', 'created_at')
                ->get();
                
// เมทอด whereColumn ยังสามารถรับอาร์เรย์เป็นพารามิเตอร์ได้
$users = Db::table('users')
                ->whereColumn([
                    ['first_name', '=', 'last_name'],
                    ['updated_at', '>', 'created_at'],
                ])->get();
```

การจัดกลุ่มพารามิเตอร์:

```php
// select * from users where name = 'John' and (votes > 100 or title = 'Admin')
$users = Db::table('users')
           ->where('name', '=', 'John')
           ->where(function ($query) {
               $query->where('votes', '>', 100)
                     ->orWhere('title', '=', 'Admin');
           })
           ->get();
```

whereExists:

```php
// select * from users where exists ( select 1 from orders where orders.user_id = users.id )
$users = Db::table('users')
           ->whereExists(function ($query) {
               $query->select(Db::raw(1))
                     ->from('orders')
                     ->whereRaw('orders.user_id = users.id');
           })
           ->get();
```

## การเรียงลำดับ
```php
$users = Db::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

## เรียงลำดับแบบสุ่ม
```php
$randomUser = Db::table('users')
                ->inRandomOrder()
                ->first();
```
> เรียงลำดับแบบสุ่มจะส่งผลกระทบต่อประสิทธิภาพของเซิร์ฟเวอร์อย่างมาก ไม่แนะนำให้ใช้

## groupBy / having
```php
$users = Db::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
// คุณสามารถส่งพารามิเตอร์หลายตัวไปให้กับเมทอด groupBy
$users = Db::table('users')
                ->groupBy('first_name', 'status')
                ->having('account_id', '>', 100)
                ->get();
```

## offset / limit
```php
$users = Db::table('users')
                ->offset(10)
                ->limit(5)
                ->get();
```

## การแทรก
แทรกเร็คคอร์ดเดียว
```php
Db::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
```
แทรกหลายเร็คคอร์ด
```php
Db::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

## รับ ID ที่เพิ่มขึ้น
```php
$id = Db::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```
> โปรดทราบว่าเมื่อใช้ PostgreSQL การใช้เมทอด insertGetId จะเป็นค่าเริ่มต้นของการสร้างฟิลด์ที่เพิ่มขึ้นโดยอัตโนมัติเป็นชื่อเข็ม

## การปรับปรุง
```php
$affected = Db::table('users')
              ->where('id', 1)
              ->update(['votes' => 1]);
```

## การปรับปรุงหรือเพิ่มข้อมูล
ในบางครั้งคุณอาจต้องการปรับปรุงบันทึกที่มีในฐานข้อมูลหรือสร้างมันหากไม่มีบันทึกที่ตรงกัน:

```php
Db::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```
เมทอด updateOrInsert จะพยายามใช้คีย์และค่าในอาร์เรย์แรกเพื่อค้นหาบันทึกฐานข้อมูลที่ตรงกัน หากพบบันทึก จะใช้ค่าในอาร์เรย์ที่สองในการปรับปรุงบันทึก หากไม่พบบันทึก จะแทรกบันทึกใหม่โดยใช้ข้อมูลจากทั้งสองอาร์เรย์

## เพิ่มค่าและลดค่า
เมทอดทั้งสองนี้จะรับอาร์เรย์ที่ เป็นพารามิเตอร์อย่างน้อยหนึ่งตัว ซึ่งคือคอลัมน์ที่ต้องการเปลี่ยนแปลง พารามิเตอร์ที่สองเป็นทางเลือก ที่ใช้ควบคุมปริมาณการเพิ่มหรือลด:

```php
Db::table('users')->increment('votes');

Db::table('users')->increment('votes', 5);

Db::table('users')->decrement('votes');

Db::table('users')->decrement('votes', 5);
```
คุณยังสามารถระบุฟิลด์ที่ต้องการปรับเปลี่ยนระหว่างกระบวนการด้วยครั้ง:

```php
Db::table('users')->increment('votes', 1, ['name' => 'John']);
```
## ลบ
```php
Db::table('users')->delete();

Db::table('users')->where('votes', '>', 100)->delete();
```
หากคุณต้องการลบข้อมูลในตาราง คุณสามารถใช้เมธอด truncate ซึ่งจะลบแถวทั้งหมดและรีเซ็ต ID ที่เพิ่มขึ้นเป็นศูนย์:
```php
Db::table('users')->truncate();
```

## ธุรกรรม
ดู [ธุรกรรมฐานข้อมูล](../others/transaction.md)

## การล็อกแบบทึกจิต
Query builder ยังรวมฟังก์ชั่นบางอย่างที่ช่วยให้คุณสามารถทำการ "ล็อกแบบทึกจิต" ในคำสั่ง select ได้ ถ้าคุณต้องการที่จะดึงข้อมูลกับ "ล็อกแบบแชร์" คุณสามารถใช้เมธอด sharedLock  "ล็อกแบบแชร์" จะป้องกันการแก้ไขคอลัมน์ข้อมูลที่ถูกเลือกไว้จนกระจัดรอการดำเนินการ:
```php
Db::table('users')->where('votes', '>', 100)->sharedLock()->get();
```
หรือคุณสามารถใช้เมธอด lockForUpdate  "ล็อกสำหรับการอัพเดต"  การใช้ "ล็อกสำหรับการอัพเดต" จะป้องกันแถวไม่ให้ถูกการเปลี่ยนแปลงหรือถูกการเลือกโดย "ล็อกแบบแชร์" ได้:
```php
Db::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

## การดีบัค
คุณสามารถใช้เมธอด dd หรือ dump เพื่อแสดงผลลัพธ์ของคำสั่งคิวรีหรือคำสั่ง SQL  การใช้เมธอด dd สามารถแสดงข้อมูลดีบัคและหยุดการทำงานของคำสั่ง ณ ที่ๆเอง ส่วน dump จะแสดงข้อมูลดีบัคเช่นเดียวกัน แต่จะไม่หยุดการทำงานของคำสั่ง:
```php
Db::table('users')->where('votes', '>', 100)->dd();
Db::table('users')->where('votes', '>', 100)->dump();
```
> **หมายเหตุ**
> การดีบัคจำเป็นต้องติดตั้ง `symfony/var-dumper`, คำสั่งคือ `composer require symfony/var-dumper`
