# การแพ็ค phar

phar เป็นไฟล์บีบอัดใน PHP ที่คล้ายกับ JAR คุณสามารถใช้ phar เพื่อแพ็คโปรเจค webman ของคุณเป็นไฟล์ phar เดียวเพื่อง่ายต่อการ deploy

**ขอขอบคุณ [fuzqing](https://github.com/fuzqing) สำหรับการ PR นี้**

> **โปรดทราบ**
> ต้องปิดการตั้งค่า phar ใน `php.ini` โดยตั้งค่า `phar.readonly = 0`

## การติดตั้งเครื่องมือคำสั่ง
`composer require webman/console`

## การแพ็ค
ในโฟลเดอร์หลักของ webman ให้รันคำสั่ง `php webman build:phar` จะสร้างไฟล์ `webman.phar` ในโฟลเดอร์ build

> การตั้งค่าการแพ็คอยู่ใน `config/plugin/webman/console/app.php`

## คำสั่งเริ่มและหยุด
**เริ่ม**
`php webman.phar start` หรือ `php webman.phar start -d`

**หยุด**
`php webman.phar stop`

**ดูสถานะ**
`php webman.phar status`

**ดูสถานะการเชื่อมต่อ**
`php webman.phar connections`

**รีสตาร์ท**
`php webman.phar restart` หรือ `php webman.phar restart -d`

## คำอธิบาย
* โปรเจคที่แพ็คแล้วไม่รองรับ reload ต้อง restart เพื่ออัปเดตโค้ด

* เพื่อป้องกันไฟล์ phar ใหญ่เกินไปและใช้หน่วยความจำมาก สามารถตั้งค่า options `exclude_pattern` และ `exclude_files` ใน `config/plugin/webman/console/app.php` เพื่อไม่รวมไฟล์ที่ไม่จำเป็น

* เมื่อเรียกใช้ webman.phar จะสร้างโฟลเดอร์ runtime ในโฟลเดอร์ที่มี webman.phar เพื่อเก็บไฟล์ล็อกและไฟล์ชั่วคราวอื่น ๆ

* หากโปรเจคของคุณมีไฟล์ .env คุณต้องวางไฟล์ .env ไว้ในโฟลเดอร์ที่มี webman.phar

* ห้ามเก็บไฟล์ที่ผู้ใช้อัปโหลดไว้ในแพ็ค phar เนื่องจากการปฏิบัติการกับไฟล์ผู้ใช้ผ่านโปรโตคอล `phar://` เป็นอันตรายมาก (ช่องโหว่ deserialization Phar) ไฟล์ที่ผู้ใช้อัปโหลดต้องเก็บแยกต่างหากบนดิสก์นอกแพ็ค phar ดูด้านล่าง

* หากธุรกิจของคุณต้องการอัปโหลดไฟล์ไปยังไดเรกทอรี public คุณต้องแยกไดเรกทอรี public และวางไว้ในโฟลเดอร์ที่มี webman.phar โดยต้องตั้งค่า `config/app.php`
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
ธุรกิจสามารถใช้ฟังก์ชันช่วยเหลือ `public_path($เส้นทางสัมพัทธ์)` เพื่อหาตำแหน่งจริงของไดเรกทอรี public

* webman.phar ไม่รองรับ custom process บน Windows
