# नियंत्रक

नया नियंत्रक फ़ाइल `app/controller/FooController.php` बनाएं।

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

जब `http://127.0.0.1:8787/foo` तक पहुंचते हैं, पेज `hello index` लौटाता है।

जब `http://127.0.0.1:8787/foo/hello` तक पहुंचते हैं, पेज `hello webman` लौटाता है।

रूट कॉन्फ़िगरेशन के जरिए रूट नियम बदल सकते हैं, [रूट](route.md) देखें।

> **सुझाव**
> 404 त्रुटि होने पर `config/app.php` खोलें, `controller_suffix` को `Controller` पर सेट करें और पुनः आरंभ करें।

## नियंत्रक प्रत्यय
webman 1.3 से `config/app.php` में नियंत्रक प्रत्यय सेट करने का समर्थन है। अगर `controller_suffix` खाली `''` है, तो नियंत्रक ऐसा होगा:

`app\controller\Foo.php`।

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

नियंत्रक प्रत्यय को `Controller` पर सेट करने की सख्त सिफारिश है, ताकि नियंत्रक और मॉडल क्लास के नाम टकराएं नहीं और सुरक्षा बढ़े।

## व्याख्या
- फ़्रेमवर्क नियंत्रक को `support\Request` ऑब्जेक्ट अपने आप भेजता है, जिससे उपयोगकर्ता इनपुट (get, post, header, cookie आदि) मिल सकता है, [अनुरोध](request.md) देखें।
- नियंत्रक संख्या, स्ट्रिंग या `support\Response` ऑब्जेक्ट लौटा सकता है, अन्य डेटा प्रकार नहीं।
- `support\Response` ऑब्जेक्ट `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` आदि हेल्पर फ़ंक्शन से बनाए जा सकते हैं।

## नियंत्रक पैरामीटर बाइंडिंग

#### उदाहरण
webman नियंत्रक मेथड पैरामीटर में अनुरोध पैरामीटर की अपने आप बाइंडिंग की सपोर्ट करता है। उदाहरण:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

`name` और `age` के मान `GET` या `POST` से या रूट पैरामीटर से भेज सकते हैं। उदाहरण:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

अग्रिमता: `रूट पैरामीटर` > `GET` > `POST` पैरामीटर।

#### डिफ़ॉल्ट मान

`/user/create?name=tom` पर जाने पर यह त्रुटि मिलेगी:

```html
Missing input parameter age
```

कारण कि हमने `age` पैरामीटर नहीं भेजा। पैरामीटर में डिफ़ॉल्ट मान देकर ठीक किया जा सकता है। उदाहरण:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### पैरामीटर प्रकार
`/user/create?name=tom&age=not_int` पर जाने पर यह त्रुटि मिलेगी:

> **सुझाव**
> परीक्षण में आसानी के लिए पैरामीटर ब्राउज़र एड्रेस बार में डाल रहे हैं। वास्तविक विकास में पैरामीटर `POST` से भेजने चाहिए।

```html
Input age must be of type int, string given
```

प्राप्त डेटा प्रकार के हिसाब से बदला जाता है। बदल नहीं पाने पर `support\exception\InputTypeException` फेंकी जाती है। `age` को `int` में नहीं बदला जा सका इसलिए यह त्रुटि आई।

#### कस्टम त्रुटि संदेश
`Missing input parameter age` और `Input age must be of type int, string given` जैसे संदेश बहुभाषा से कस्टमाइज़ किए जा सकते हैं। ये कमांड देखें:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'इनपुट पैरामीटर :parameter प्रकार :exceptType का होना चाहिए, भेजा गया प्रकार :actualType है',
    'Missing input parameter :parameter' => 'इनपुट पैरामीटर :parameter गायब है',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### अन्य प्रकार
webman `int`, `float`, `string`, `bool`, `array`, `object` और `क्लास इन्स्टैंस` जैसे पैरामीटर प्रकार सपोर्ट करता है। उदाहरण:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

`/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` पर जाने पर:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### क्लास इन्स्टैंस
webman पैरामीटर टाइप हिंट से क्लास इन्स्टैंस पास करने की सपोर्ट करता है। उदाहरण:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

`/blog/create?blog[title]=hello&blog[content]=world` पर जाने पर:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### मॉडल इन्स्टैंस

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // सामने के छोर से असुरक्षित फील्ड आने से रोकने के लिए यहाँ भरने योग्य फील्ड जोड़ें
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

`/user/create?user[name]=tom&user[age]=18` पर जाने पर लगभग ऐसा नतीजा मिलेगा:

```json
1
```

## नियंत्रक जीवन चक्र

जब `config/app.php` में `controller_reuse` `false` हो, हर अनुरोध पर एक बार नियंत्रक इन्स्टैंस बनता है और अनुरोध खत्म होने पर हट जाता है। पारंपरिक फ़्रेमवर्क की तरह।

जब `controller_reuse` `true` हो, सभी अनुरोध एक ही नियंत्रक इन्स्टैंस इस्तेमाल करते हैं, यानी बनने के बाद वह मेमोरी में रहता है।

> **ध्यान**
> रीयूज़ चालू होने पर अनुरोध को नियंत्रक की कोई भी प्रॉपर्टी बदलनी नहीं चाहिए, क्योंकि उसका असर बाद के अनुरोधों पर पड़ेगा। उदाहरण:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // पहले update?id=1 अनुरोध के बाद model रह जाएगा
        // delete?id=2 अनुरोध आने पर id=1 का डेटा मिट जाएगा
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **सुझाव**
> नियंत्रक के `__construct()` में डेटा return करने का कोई असर नहीं होता। उदाहरण:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // कन्स्ट्रक्टर में return का कोई असर नहीं, ब्राउज़र को यह रिस्पॉन्स नहीं मिलेगा
        return response('hello'); 
    }
}
```

## रीयूज़ न करने और रीयूज़ करने में अंतर

#### रीयूज़ न करना
हर अनुरोध पर नया नियंत्रक इन्स्टैंस बनता है, अनुरोध खत्म पर हट जाता है। पारंपरिक फ़्रेमवर्क जैसा, ज़्यादातर डेवलपर्स की आदत के अनुकूल। बार-बार बनाने/हटाने की वजह से प्रदर्शन रीयूज़ से थोड़ा कम (helloworld बेंचमार्क में लगभग 10%, असली बिजनेस में आमतौर पर नगण्य)।

#### रीयूज़ करना
एक प्रोसेस में सिर्फ एक बार नियंत्रक बनता है, अनुरोध खत्म होने पर इन्स्टैंस हटाया नहीं जाता। उस प्रोसेस के बाद के अनुरोध वही इन्स्टैंस इस्तेमाल करते हैं। बेहतर प्रदर्शन, लेकिन ज़्यादातर डेवलपर्स की आदत से कम मेल खाता है।

#### इन हालात में रीयूज़ नहीं करना चाहिए

जब अनुरोध नियंत्रक की प्रॉपर्टी बदलता हो – इसका असर बाद के अनुरोधों पर पड़ेगा।

कुछ डेवलपर्स हर अनुरोध के लिए नियंत्रक के `__construct()` में कुछ इनिशियलाइज़ करते हैं। ऐसे में रीयूज़ नहीं करना चाहिए, क्योंकि प्रोसेस में कन्स्ट्रक्टर सिर्फ एक बार चलता है, हर अनुरोध पर नहीं।
