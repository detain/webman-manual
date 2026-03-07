# think-cache

think-cache হল thinkphp ফ্রেমওয়ার্ক থেকে নিষ্কাশিত একটি উপাদান, যাতে সংযোগ পুল সমর্থন যুক্ত করা হয়েছে। এটি করউটিন এবং নন-করউটিন পরিবেশ উভয়েই স্বয়ংক্রিয়ভাবে সমর্থন করে।

## ইনস্টলেশন
`composer require -W webman/think-cache`

ইনস্টলেশনের পর পুনঃআরম্ভ (restart) প্রয়োজন (reload কার্যকর নয়)

### কনফিগারেশন ফাইল

কনফিগারেশন ফাইল হল `config/think-cache.php`

## ব্যবহার

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
## প্রদত্ত API
```php
// ক্যাশ সেট করুন
Cache::set('val','value',600);
// ক্যাশ আছে কিনা যাচাই করুন
Cache::has('val');
// ক্যাশ পান
Cache::get('val');
// ক্যাশ মুছুন
Cache::delete('val');
// ক্যাশ সাফ করুন
Cache::clear();
// পড়ুন এবং ক্যাশ মুছুন
Cache::pull('val');
// না থাকলে লিখুন
Cache::remember('val',10);

// সংখ্যাসূচক ক্যাশ ডেটার জন্য
// ক্যাশ 1 করে বাড়ান
Cache::inc('val');
// ক্যাশ 5 করে বাড়ান
Cache::inc('val',5);
// ক্যাশ 1 করে কমান
Cache::dec('val');
// ক্যাশ 5 করে কমান
Cache::dec('val',5);

// ক্যাশ ট্যাগ ব্যবহার করুন
Cache::tag('tag_name')->set('val','value',600);
// নির্দিষ্ট ট্যাগের আওতায় ক্যাশ মুছুন
Cache::tag('tag_name')->clear();
// একাধিক ট্যাগ সমর্থন করে
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// একাধিক ট্যাগের আওতায় ক্যাশ মুছুন
Cache::tag(['tag1','tag2'])->clear();

// বিভিন্ন ক্যাশ স্টোর ব্যবহার করুন
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


