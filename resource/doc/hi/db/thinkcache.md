# think-cache

think-cache, thinkphp फ्रेमवर्क से निकाला गया एक घटक है, जिसमें कनेक्शन पूल सपोर्ट जोड़ा गया है। यह स्वचालित रूप से कोरूटीन और नॉन-कोरूटीन दोनों वातावरणों का समर्थन करता है।

## इंस्टॉलेशन
`composer require -W webman/think-cache`

इंस्टॉलेशन के बाद पुनर्आरंभ (restart) आवश्यक है (reload प्रभावी नहीं है)

### कॉन्फ़िगरेशन फाइल

कॉन्फ़िगरेशन फाइल `config/think-cache.php` है

## उपयोग

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
## प्रदान की गई API
```php
// कैश सेट करें
Cache::set('val','value',600);
// कैश मौजूद है या नहीं जांचें
Cache::has('val');
// कैश प्राप्त करें
Cache::get('val');
// कैश हटाएं
Cache::delete('val');
// कैश साफ़ करें
Cache::clear();
// पढ़ें और कैश हटाएं
Cache::pull('val');
// मौजूद न हो तो लिखें
Cache::remember('val',10);

// संख्यात्मक कैश डेटा के लिए
// कैश को 1 से बढ़ाएं
Cache::inc('val');
// कैश को 5 से बढ़ाएं
Cache::inc('val',5);
// कैश को 1 से घटाएं
Cache::dec('val');
// कैश को 5 से घटाएं
Cache::dec('val',5);

// कैश टैग का उपयोग करें
Cache::tag('tag_name')->set('val','value',600);
// किसी टैग के तहत कैश हटाएं
Cache::tag('tag_name')->clear();
// कई टैग का समर्थन करता है
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// कई टैग के तहत कैश हटाएं
Cache::tag(['tag1','tag2'])->clear();

// विभिन्न कैश स्टोर का उपयोग करें
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


