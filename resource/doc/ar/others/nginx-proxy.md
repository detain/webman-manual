# وكيل nginx

عندما يحتاج webman إلى توفير الوصول المباشر من الشبكة الخارجية، يُوصى بإضافة وكيل nginx أمام webman. وهذا يوفر الفوائد التالية:

- تتم معالجة الموارد الثابتة بواسطة nginx، مما يتيح لـ webman التركيز على منطق الأعمال
- يمكن لعدة مثيلات من webman مشاركة المنافذ 80 و 443، والتمييز بين المواقع المختلفة حسب اسم النطاق، مما يتيح تشغيل عدة مواقع على خادم واحد
- يسمح بتعايش بنية php-fpm و webman
- وكيل nginx مع SSL لـ https أبسط وأكثر كفاءة
- يمكن تصفية الطلبات غير القانونية من الشبكة الخارجية بشكل صارم

## مثال على وكيل nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name نطاق_الموقع;
  listen 80;
  access_log off;
  # مهم: يجب أن يشير root إلى مجلد public ضمن webman، وليس إلى المجلد الجذر لـ webman
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

  # رفض الوصول لجميع الملفات التي تنتهي بـ .php
  location ~ \.php$ {
      return 404;
  }

  # السماح بالوصول إلى مجلد .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # رفض الوصول لجميع الملفات أو المجلدات التي تبدأ بـ .
  location ~ /\. {
      return 404;
  }

}
```

بشكل عام، يحتاج المطورون فقط إلى تكوين server_name و root بالقيم الفعلية؛ ولا حاجة لتكوين الحقول الأخرى.

> **ملاحظة**
> من المهم بشكل خاص أن تشير خيار root إلى مجلد public ضمن webman. لا تضبطها أبداً على المجلد الجذر لـ webman، وإلا فقد تصبح جميع ملفاتك قابلة للتنزيل والوصول من الإنترنت، بما في ذلك الملفات الحساسة مثل إعدادات قاعدة البيانات.
