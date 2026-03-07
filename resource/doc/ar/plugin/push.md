# webman/push

`webman/push` هي إضافة خادم دفع مجانية، يعتمد العميل على نموذج الاشتراك، متوافقة مع [pusher](https://pusher.com)، وتضم العديد من العملاء مثل JS وAndroid (java) وIOS (swift) وIOS (Obj-C) وuniapp و.NET وUnity وFlutter وAngularJS وغيرها. يدعم SDK الدفع في الخلفية PHP وNode وRuby وAsp وJava وPython وGo وSwift وغيرها. يأتي العميل مزوداً بنبضات قلب وإعادة اتصال تلقائية عند انقطاع الاتصال، مما يجعله بسيطاً ومستقراً جداً في الاستخدام. مناسبة لسيناريوهات التواصل الفوري مثل دفع الرسائل والدردشة وغيرها.

تأتي الإضافة مع عميل JavaScript لصفحة الويب push.js وعميل uniapp `uniapp-push.js`، ويمكن تنزيل عملاء اللغات الأخرى من https://pusher.com/docs/channels/channels_libraries/libraries/

## التثبيت

```sh
composer require webman/push
```

## العميل (javascript)

**استيراد عميل javascript**
```js
<script src="/plugin/webman/push/push.js"> </script>
```

**استخدام العميل (قناة عامة)**
```js
// إنشاء الاتصال
var connection = new Push({
    url: 'ws://127.0.0.1:3131', // عنوان websocket
    app_key: '<app_key، يتم الحصول عليه من config/plugin/webman/push/app.php>',
    auth: '/plugin/webman/push/auth' // مصادقة الاشتراك (لللقنوات الخاصة فقط)
});
// افترض أن uid المستخدم هو 1
var uid = 1;
// يستمع المتصفح لرسائل قناة user-1، أي رسائل المستخدم صاحب uid 1
var user_channel = connection.subscribe('user-' + uid);

// عندما تحتوي قناة user-1 على رسالة حدث message
user_channel.on('message', function(data) {
    // data يحتوي على محتوى الرسالة
    console.log(data);
});
// عندما تحتوي قناة user-1 على حدث friendApply
user_channel.on('friendApply', function (data) {
    // data يحتوي على معلومات طلب الصداقة
    console.log(data);
});

// افترض أن معرف المجموعة هو 2
var group_id = 2;
// يستمع المتصفح لرسائل قناة group-2، أي رسائل المجموعة 2
var group_channel = connection.subscribe('group-' + group_id);
// عندما تحتوي المجموعة 2 على حدث رسالة message
group_channel.on('message', function(data) {
    // data يحتوي على محتوى الرسالة
    console.log(data);
});
```

> **نصائح**
> في المثال أعلاه، subscribe تنفذ اشتراك القناة، `message` و`friendApply` هما أحداث على القناة. القناة والأحداث عبارة عن سلاسل نصية عشوائية ولا تحتاج إلى تكوين مسبق على الخادم.

## الدفع من الخادم (PHP)
```php
use Webman\Push\Api;
$api = new Api(
    // في webman يمكن استخدام config مباشرة للحصول على التكوين، في بيئة غير webman يجب كتابة التكوين المقابل يدوياً
    'http://127.0.0.1:3232',
    config('plugin.webman.push.app.app_key'),
    config('plugin.webman.push.app.app_secret')
);
// إرسال رسالة حدث message لجميع العملاء المشتركين في user-1
$api->trigger('user-1', 'message', [
    'from_uid' => 2,
    'content'  => 'مرحباً، هذا محتوى الرسالة'
]);
```

## القنوات الخاصة
في الأمثلة أعلاه، يمكن لأي مستخدم الاشتراك في المعلومات عبر Push.js، وإذا كانت المعلومات حساسة فهذا غير آمن.

يدعم `webman/push` الاشتراك في القنوات الخاصة، والقنوات الخاصة هي القنوات التي تبدأ بـ `private-`. مثلاً
```js
var connection = new Push({
    url: 'ws://127.0.0.1:3131', // عنوان websocket
    app_key: '<app_key>',
    auth: '/plugin/webman/push/auth' // مصادقة الاشتراك (لللقنوات الخاصة فقط)
});

// افترض أن uid المستخدم هو 1
var uid = 1;
// يستمع المتصفح لرسائل القناة الخاصة private-user-1
var user_channel = connection.subscribe('private-user-' + uid);
```

عندما يشترك العميل في قناة خاصة (قناة تبدأ بـ `private-`)، يرسل المتصفح طلب مصادقة ajax (عنوان ajax هو العنوان المكون في معلمة auth عند إنشاء Push جديد)، ويمكن للمطور هنا التحقق مما إذا كان للمستخدم الحالي صلاحية الاستماع لهذه القناة. وهذا يضمن أمان الاشتراك.

> لمزيد من المعلومات حول المصادقة راجع الكود في `config/plugin/webman/push/route.php`

## الدفع من العميل
جميع الأمثلة أعلاه تتعلق باشتراك العميل في قناة معينة ونداء الخادم لواجهة API للدفع. يدعم webman/push أيضاً دفع الرسائل مباشرة من العميل.

> **ملاحظة**
> الدفع بين العملاء يدعم القنوات الخاصة فقط (القنوات التي تبدأ بـ `private-`)، ولا يمكن للعميل إلا تفعيل الأحداث التي تبدأ بـ `client-`.

مثال على تفعيل حدث الدفع من العميل
```js
var user_channel = connection.subscribe('private-user-1');
user_channel.on('client-message', function (data) {
    // 
});
user_channel.trigger('client-message', {form_uid:2, content:"مرحباً"});
```

> **ملاحظة**
> الكود أعلاه يرسل بيانات حدث `client-message` لجميع العملاء (باستثناء العميل الحالي) المشتركين في `private-user-1` (العميل المُرسل لن يستلم البيانات التي أرسلها بنفسه).

## webhooks

تُستخدم webhook لاستقبال بعض أحداث القناة.

**حالياً يوجد حدثان رئيسيان:**

- 1、channel_added
  يُفعّل عند انتقال قناة من عدم وجود عملاء متصلين إلى وجود عملاء متصلين، أو بعبارة أخرى حدث الاتصال

- 2、channel_removed
  يُفعّل عند انقطاع اتصال جميع عملاء قناة معينة، أو بعبارة أخرى حدث الانقطاع

> **نصائح**
> هذه الأحداث مفيدة جداً في الحفاظ على حالة اتصال المستخدمين.

> **ملاحظة**
> يتم تكوين عنوان webhook في `config/plugin/webman/push/app.php`.
> راجع منطق الكود لاستقبال ومعالجة أحداث webhook في `config/plugin/webman/push/route.php`
> نظراً لأن انقطاع المستخدم المؤقت بسبب تحديث الصفحة لا ينبغي اعتباره انقطاعاً، سيقوم webman/push بإجراء حكم متأخر، لذا ستتأخر أحداث الاتصال/الانقطاع من 1 إلى 3 ثوانٍ.

## وكيل wss (SSL)
لا يمكن استخدام اتصال ws تحت https، بل يجب استخدام اتصال wss. في هذه الحالة يمكن استخدام nginx كوكيل wss، بتكوين مشابه لما يلي:
```
server {
    # .... تم حذف التكوينات الأخرى هنا ...

    location /app/<app_key>
    {
        proxy_pass http://127.0.0.1:3131;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
**ملاحظة: يتم الحصول على `<app_key>` في التكوين أعلاه من `config/plugin/webman/push/app.php`**

بعد إعادة تشغيل nginx، استخدم الطريقة التالية للاتصال بالخادم
```
var connection = new Push({
    url: 'wss://example.com',
    app_key: '<app_key، يتم الحصول عليه من config/plugin/webman/push/app.php>',
    auth: '/plugin/webman/push/auth' // مصادقة الاشتراك (لللقنوات الخاصة فقط)
});
```
> **ملاحظة**
> 1. يبدأ عنوان الطلب بـ wss
> 2. لا تكتب المنفذ
> 3. يجب استخدام **النطاق المقابل لشهادة SSL** للاتصال

## تعليمات استخدام push-vue.js

1、انسخ الملف push-vue.js إلى مجلد المشروع، مثلاً: src/utils/push-vue.js

2、أضفه في صفحة vue
```js

<script lang="ts" setup>
import {  onMounted } from 'vue'
import { Push } from '../utils/push-vue'

onMounted(() => {
  console.log('تم تحميل المكون') 

  // إنشاء مثيل webman-push

  // إنشاء الاتصال
  var connection = new Push({
    url: 'ws://127.0.0.1:3131', // عنوان websocket
    app_key: '<app_key، يتم الحصول عليه من config/plugin/webman/push/app.php>',
    auth: '/plugin/webman/push/auth' // مصادقة الاشتراك (لللقنوات الخاصة فقط)
  });

  // افترض أن uid المستخدم هو 1
  var uid = 1;
  // يستمع المتصفح لرسائل قناة user-1، أي رسائل المستخدم صاحب uid 1
  var user_channel = connection.subscribe('user-' + uid);

  // عندما تحتوي قناة user-1 على رسالة حدث message
  user_channel.on('message', function (data) {
    // data يحتوي على محتوى الرسالة
    console.log(data);
  });
  // عندما تحتوي قناة user-1 على حدث friendApply
  user_channel.on('friendApply', function (data) {
    // data يحتوي على معلومات طلب الصداقة
    console.log(data);
  });

  // افترض أن معرف المجموعة هو 2
  var group_id = 2;
  // يستمع المتصفح لرسائل قناة group-2، أي رسائل المجموعة 2
  var group_channel = connection.subscribe('group-' + group_id);
  // عندما تحتوي المجموعة 2 على حدث رسالة message
  group_channel.on('message', function (data) {
    // data يحتوي على محتوى الرسالة
    console.log(data);
  });


})

</script>
```

## عناوين العملاء الأخرى
`webman/push` متوافق مع pusher، عناوين تنزيل عملاء اللغات الأخرى (Java Swift .NET Objective-C Unity Flutter Android IOS AngularJS وغيرها):
https://pusher.com/docs/channels/channels_libraries/libraries/
