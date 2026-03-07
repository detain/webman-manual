# think-cache

think-cache مُكوِّن مُستخلَص من إطار عمل thinkphp، مع إضافة دعم تجميع الاتصالات، ويدعم تلقائياً بيئات التآزر (coroutine) وغير التآزر.

## التثبيت
`composer require -W webman/think-cache`

يجب إعادة التشغيل (restart) بعد التثبيت (reload غير فعّال)

### ملف الإعدادات

الملف هو `config/think-cache.php`

## الاستخدام

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
## الواجهات المتاحة
```php
// تعيين الذاكرة المؤقتة
Cache::set('val','value',600);
// التحقق من وجود الذاكرة المؤقتة
Cache::has('val');
// جلب الذاكرة المؤقتة
Cache::get('val');
// حذف الذاكرة المؤقتة
Cache::delete('val');
// مسح الذاكرة المؤقتة
Cache::clear();
// قراءة وحذف الذاكرة المؤقتة
Cache::pull('val');
// الكتابة إذا لم تكن موجودة
Cache::remember('val',10);

// للبيانات الرقمية في الذاكرة المؤقتة
// زيادة القيمة بمقدار 1
Cache::inc('val');
// زيادة القيمة بمقدار 5
Cache::inc('val',5);
// تقليل القيمة بمقدار 1
Cache::dec('val');
// تقليل القيمة بمقدار 5
Cache::dec('val',5);

// استخدام وسوم الذاكرة المؤقتة
Cache::tag('tag_name')->set('val','value',600);
// حذف البيانات تحت وسم معين
Cache::tag('tag_name')->clear();
// دعم وسوم متعددة
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// حذف البيانات تحت وسوم متعددة
Cache::tag(['tag1','tag2'])->clear();

// استخدام أنواع تخزين متعددة
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


