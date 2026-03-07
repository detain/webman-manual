# Nginx Proxy

เมื่อ webman ต้องการให้เข้าถึงจากเครือข่ายภายนอกโดยตรง แนะนำให้เพิ่ม nginx proxy ไว้ด้านหน้าของ webman ซึ่งให้ประโยชน์ดังนี้:

- ทรัพยากรสถิตย์จัดการโดย nginx ทำให้ webman โฟกัสที่การประมวลผลตรรกะธุรกิจ
- หลาย webman สามารถใช้พอร์ต 80 และ 443 ร่วมกัน แยกแยะไซต์ตามชื่อโดเมน ทำให้มีหลายไซต์บนเซิร์ฟเวอร์เดียว
- สามารถให้สถาปัตยกรรม php-fpm และ webman ทำงานร่วมกันได้
- nginx proxy พร้อม SSL สำหรับ https ทำได้ง่ายและมีประสิทธิภาพกว่า
- สามารถกรองคำขอที่ผิดกฎหมายจากเครือข่ายภายนอกอย่างเข้มงวด

## ตัวอย่าง Nginx Proxy
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name โดเมนของไซต์;
  listen 80;
  access_log off;
  # สำคัญ: root ต้องชี้ไปที่โฟลเดอร์ public ภายใต้ webman ไม่ใช่โฟลเดอร์รากของ webman
  root /your/webman/public;

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_pass http://webman;
  }

  # ปฏิเสธการเข้าถึงไฟล์ทั้งหมดที่ลงท้ายด้วย .php
  location ~ \.php$ {
      return 404;
  }

  # อนุญาตการเข้าถึงโฟลเดอร์ .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # ปฏิเสธการเข้าถึงไฟล์หรือโฟลเดอร์ทั้งหมดที่ขึ้นต้นด้วย .
  location ~ /\. {
      return 404;
  }

}
```

โดยทั่วไป นักพัฒนาต้องกำหนดค่า server_name และ root ด้วยค่าจริงเท่านั้น ส่วนฟิลด์อื่นๆ ไม่จำเป็นต้องกำหนดค่า

> **หมายเหตุ**
> สิ่งสำคัญอย่างยิ่งคือตัวเลือก root ต้องชี้ไปที่โฟลเดอร์ public ภายใต้ webman ห้ามตั้งค่าเป็นโฟลเดอร์รากของ webman มิฉะนั้นไฟล์ทั้งหมดของคุณอาจถูกดาวน์โหลดและเข้าถึงได้จากอินเทอร์เน็ต รวมถึงไฟล์สำคัญ เช่น การตั้งค่าฐานข้อมูล
