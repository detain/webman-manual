# โครงสร้างไดเรกทอรี

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

ปลั๊กอินแอปพลิเคชันมีโครงสร้างไดเรกทอรีและไฟล์กำหนดค่าเหมือนกับ webman โดยประสบการณ์การพัฒนาก็แทบไม่ต่างจากการพัฒนาแอปพลิเคชัน webman ปกติ

ไดเรกทอรีและการตั้งชื่อปลั๊กอินตามข้อกำหนด PSR-4 เนื่องจากปลั๊กอินทั้งหมดอยู่ในไดเรกทอรี `plugin` จึงทำให้เนมสเปซทั้งหมดขึ้นต้นด้วย `plugin` เช่น `plugin\foo\app\controller\UserController`

## เกี่ยวกับไดเรกทอรี api

แต่ละปลั๊กอินมีไดเรกทอรี `api` หากแอปพลิเคชันของคุณมีอินเทอร์เฟซภายในให้แอปพลิเคชันอื่นเรียกใช้ ให้นำอินเทอร์เฟซเหล่านั้นวางไว้ในไดเรกทอรี `api`

โปรดทราบ: อินเทอร์เฟซที่กล่าวถึงคืออินเทอร์เฟซการเรียกใช้ฟังก์ชัน ไม่ใช่อินเทอร์เฟซเครือข่าย/HTTP

ตัวอย่างเช่น ปลั๊กอินอีเมลให้บริการอินเทอร์เฟซ `Email::send()` ที่ `plugin/email/api/Email.php` เพื่อให้แอปพลิเคชันอื่นเรียกใช้เมื่อส่งอีเมล นอกจากนี้ `plugin/email/api/Install.php` ถูกสร้างอัตโนมัติเพื่อให้ตลาดปลั๊กอิน webman-admin เรียกใช้การติดตั้งหรือถอดการติดตั้ง
