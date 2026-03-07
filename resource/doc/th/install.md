# วิธีติดตั้ง webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: ติดตั้งสภาพแวดล้อม PHP + Composer (ข้ามได้ถ้ามีอยู่แล้ว)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **หมายเหตุ**
> คำสั่งด้านบนใช้ได้กับ Linux และ macOS ผู้ใช้ Windows ต้องติดตั้ง PHP แยกต่างหาก

คุณสามารถดาวน์โหลด [PHP แบบ static](https://www.workerman.net/download) ที่ webman จัดเตรียมไว้ด้วยตนเอง แล้วแตกไฟล์เพื่อใช้งานได้เช่นกัน

## 1. สร้างโปรเจค

```php
composer create-project workerman/webman:~2.0
```

> **เคล็ดลับ**
> หากเกิดข้อผิดพลาด อาจเป็นเพราะใช้ Composer mirror ที่มีปัญหา ให้รัน `composer config -g --unset repos.packagist` เพื่อยกเลิก proxy

## 2. รัน

เข้าไปในไดเรกทอรี webman

#### ผู้ใช้ Windows
ดับเบิลคลิก `windows.bat` หรือรัน `php windows.php` เพื่อเริ่มต้น

> **เคล็ดลับ**
> หากมีข้อผิดพลาด ฟังก์ชันอาจถูกปิดใช้งาน ดูที่ [การตรวจสอบฟังก์ชันที่ถูกปิดใช้งาน](others/disable-function-check.md) เพื่อยกเลิกการปิดใช้งาน

#### ผู้ใช้ Linux
**โหมด debug** (สำหรับพัฒนา: ข้อมูลจะแสดงในเทอร์มินัล เมื่อปิดเทอร์มินัล บริการ webman จะหยุดด้วย)

```php
php start.php start
```

**โหมด daemon** (สำหรับสภาพแวดล้อมจริง: ข้อมูลไม่แสดงในเทอร์มินัล เมื่อปิดเทอร์มินัล บริการ webman จะทำงานต่อเนื่อง)

```php
php start.php start -d
```

#### ผู้ใช้ Docker

เริ่มบริการทั้งหมดและแนบกับคอนโซล
```php
docker-compose up
```

รันบริการในโหมดพื้นหลัง
```php
docker-compose up -d
```

> **เคล็ดลับ**
> หากมีข้อผิดพลาด ฟังก์ชันอาจถูกปิดใช้งาน ดูที่ [การตรวจสอบฟังก์ชันที่ถูกปิดใช้งาน](others/disable-function-check.md) เพื่อยกเลิกการปิดใช้งาน

## 3. เข้าถึง

เปิดเบราว์เซอร์ไปที่ `http://ที่อยู่-ip:8787`
