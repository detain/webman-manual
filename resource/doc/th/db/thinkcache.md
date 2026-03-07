# think-cache

think-cache เป็นคอมโพเนนต์ที่แยกออกมาจากเฟรมเวิร์ก thinkphp โดยเพิ่มการสนับสนุนคอนเนคชันพูล และรองรับทั้งสภาพแวดล้อมแบบ coroutine และ non-coroutine โดยอัตโนมัติ

## การติดตั้ง
`composer require -W webman/think-cache`

หลังการติดตั้งต้อง restart (reload ไม่มีผล)

### ไฟล์การกำหนดค่า

ไฟล์การกำหนดค่าคือ `config/think-cache.php`

## การใช้งาน

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
## API ที่ให้มา
```php
// ตั้งค่าแคช
Cache::set('val','value',600);
// ตรวจสอบว่าแคชมีอยู่หรือไม่
Cache::has('val');
// ดึงแคช
Cache::get('val');
// ลบแคช
Cache::delete('val');
// ล้างแคช
Cache::clear();
// อ่านและลบแคช
Cache::pull('val');
// เขียนถ้าไม่มีอยู่
Cache::remember('val',10);

// สำหรับข้อมูลแคชที่เป็นตัวเลข
// เพิ่มแคชทีละ 1
Cache::inc('val');
// เพิ่มแคชทีละ 5
Cache::inc('val',5);
// ลดแคชทีละ 1
Cache::dec('val');
// ลดแคชทีละ 5
Cache::dec('val',5);

// ใช้แท็กแคช
Cache::tag('tag_name')->set('val','value',600);
// ลบแคชภายใต้แท็กที่ระบุ
Cache::tag('tag_name')->clear();
// รองรับหลายแท็ก
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// ลบแคชภายใต้หลายแท็ก
Cache::tag(['tag1','tag2'])->clear();

// ใช้สโตร์แคชแบบต่าง ๆ
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


