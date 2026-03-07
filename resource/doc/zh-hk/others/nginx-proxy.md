# nginx代理
當webman需要直接提供外網存取時，建議在webman前增加一個nginx代理，這樣有以下好處。

- 靜態資源由nginx處理，讓webman專注業務邏輯處理
- 讓多個webman共用80、443埠，透過網域區分不同站點，實現單台伺服器部署多個站點
- 能夠實現php-fpm與webman架構共存
- nginx代理ssl實現https，更加簡單高效
- 能夠嚴格過濾外網一些不合法請求

## nginx代理範例
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name 站點網域;
  listen 80;
  access_log off;
  # 注意，這裡一定是webman下的public目錄，不能是webman根目錄
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

  # 拒絕存取所有以 .php 結尾的檔案
  location ~ \.php$ {
      return 404;
  }

  # 允許存取 .well-known 目錄
  location ~ ^/\.well-known/ {
    allow all;
  }

  # 拒絕存取所有以 . 開頭的檔案或目錄
  location ~ /\. {
      return 404;
  }

}
```

一般來說以上設定開發者只需要將server_name和root設定成實際值即可，其他欄位不需要設定。

> **注意**
> 特別注意的是，root選項一定要設定成webman下的public目錄，千萬不要直接設定成webman目錄，否則您的所有檔案可能會被外網下載存取，包括資料庫設定等敏感檔案。
