# Nginx প্রক্সি

যখন webman-কে বাহ্যিক নেটওয়ার্ক থেকে সরাসরি অ্যাক্সেস প্রদান করতে হয়, তখন webman-এর সামনে একটি nginx প্রক্সি যোগ করার পরামর্শ দেওয়া হয়। এর ফলে নিম্নলিখিত সুবিধা পাওয়া যায়:

- স্থির সম্পদ nginx দ্বারা প্রক্রিয়া করা হয়, webman ব্যবসায়িক যুক্তির দিকে মনোযোগ দিতে পারে
- একাধিক webman উদাহরণ 80 এবং 443 পোর্ট শেয়ার করতে পারে, ডোমেইন নাম অনুযায়ী সাইটগুলি আলাদা করতে পারে, একটি সার্ভারে একাধিক সাইট চালাতে দেয়
- php-fpm এবং webman আর্কিটেকচার একসাথে কাজ করতে সক্ষম করে
- HTTPS-এর জন্য nginx প্রক্সি SSL সহ সহজ এবং আরও দক্ষ
- বাহ্যিক নেটওয়ার্ক থেকে অবৈধ অনুরোধ কঠোরভাবে ফিল্টার করতে পারে

## Nginx প্রক্সি উদাহরণ
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name site_domain;
  listen 80;
  access_log off;
  # গুরুত্বপূর্ণ: root অবশ্যই webman-এর নিচের public ডাইরেক্টরি নির্দেশ করতে হবে, webman রুট ডাইরেক্টরি নয়
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

  # .php দিয়ে শেষ হওয়া সমস্ত ফাইলের অ্যাক্সেস প্রত্যাখ্যান করুন
  location ~ \.php$ {
      return 404;
  }

  # .well-known ডাইরেক্টরির অ্যাক্সেস অনুমতি দিন
  location ~ ^/\.well-known/ {
    allow all;
  }

  # . দিয়ে শুরু হওয়া সমস্ত ফাইল বা ডাইরেক্টরির অ্যাক্সেস প্রত্যাখ্যান করুন
  location ~ /\. {
      return 404;
  }

}
```

সাধারণত, ডেভেলপারদের কেবল server_name এবং root প্রকৃত মান দিয়ে কনফিগার করতে হয়; অন্যান্য ফিল্ড কনফিগার করার প্রয়োজন নেই।

> **দ্রষ্টব্য**
> বিশেষভাবে গুরুত্বপূর্ণ: root অপশন অবশ্যই webman-এর নিচের public ডাইরেক্টরি নির্দেশ করতে হবে। কখনই webman রুট ডাইরেক্টরিতে সেট করবেন না, অন্যথায় ডেটাবেস কনফিগারেশনের মতো সংবেদনশীল ফাইল সহ আপনার সমস্ত ফাইল ইন্টারনেট থেকে ডাউনলোড এবং অ্যাক্সেসযোগ্য হতে পারে।
