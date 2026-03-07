# كيفية تثبيت webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: تثبيت بيئة PHP + Composer (تخطى إذا كانت مثبتة مسبقاً)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **ملاحظة**
> الأمر أعلاه ينطبق على Linux و macOS. مستخدمو Windows يحتاجون إلى تثبيت PHP بشكل منفصل.

يمكنك أيضاً تنزيل [نسخة PHP الثابتة](https://www.workerman.net/download) المقدمة من webman يدوياً واستخراجها للاستخدام.

## 1. إنشاء المشروع

```php
composer create-project workerman/webman:~2.0
```

> **نصيحة**
> في حال حدوث أخطاء، قد تكون تستخدم مرآة Composer معطوبة. نفّذ `composer config -g --unset repos.packagist` لإزالة الوكيل.

## 2. التشغيل

انتقل إلى مجلد webman

#### مستخدمو Windows
انقر نقراً مزدوجاً على `windows.bat` أو نفّذ `php windows.php` للبدء

> **نصيحة**
> في حال حدوث أخطاء، قد تكون بعض الدوال معطلة. راجع [التحقق من الدوال المعطلة](others/disable-function-check.md) لتفعيلها.

#### مستخدمو Linux
**وضع التصحيح** (للتطوير: يظهر المخرجات في الطرفية، وتتوقف الخدمة عند إغلاق الطرفية)

```php
php start.php start
```

**وضع الخدمة الخلفية** (للإنتاج: لا تظهر مخرجات في الطرفية، وتستمر الخدمة بعد إغلاق الطرفية)

```php
php start.php start -d
```

#### مستخدمو Docker

تشغيل جميع الخدمات والإرفاق بالوحدة الطرفية
```php
docker-compose up
```

تشغيل الخدمات في الخلفية
```php
docker-compose up -d
```

> **نصيحة**
> في حال حدوث أخطاء، قد تكون بعض الدوال معطلة. راجع [التحقق من الدوال المعطلة](others/disable-function-check.md) لتفعيلها.

## 3. الوصول

افتح `http://عنوان-ip:8787` في المتصفح.
