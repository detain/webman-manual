# Nginx proxy

webman'ın dış ağdan doğrudan erişim sağlaması gerektiğinde, webman'ın önüne bir nginx proxy eklenmesi önerilir. Bu durumun şu faydaları vardır:

- Statik kaynaklar nginx tarafından işlenir, webman'ın iş mantığına odaklanmasını sağlar
- Birden çok webman örneği 80 ve 443 portlarını paylaşabilir, siteyi etki alanı adıyla ayırt edebilir ve tek sunucuda birden çok site barındırabilir
- php-fpm ve webman mimarisinin birlikte çalışmasına olanak tanır
- Nginx proxy ile SSL üzerinden https uygulaması daha basit ve verimlidir
- Dış ağdaki yasadışı istekleri sıkı bir şekilde filtreleyebilir

## Nginx proxy örneği
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name site_domain;
  listen 80;
  access_log off;
  # Önemli: root, webman altındaki public dizinine işaret etmelidir, webman kök dizinine değil
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

  # .php ile biten tüm dosyalara erişimi reddet
  location ~ \.php$ {
      return 404;
  }

  # .well-known dizinine erişime izin ver
  location ~ ^/\.well-known/ {
    allow all;
  }

  # . ile başlayan tüm dosya veya dizinlere erişimi reddet
  location ~ /\. {
      return 404;
  }

}
```

Genel olarak, geliştiricilerin yalnızca server_name ve root'u gerçek değerlerle yapılandırması yeterlidir; diğer alanların yapılandırılması gerekmez.

> **Not**
> Özellikle önemli: root seçeneği webman altındaki public dizinine işaret etmelidir. Asla webman kök dizinine ayarlamayın, aksi takdirde veritabanı yapılandırması gibi hassas dosyalar dahil tüm dosyalarınız internetten indirilebilir ve erişilebilir olabilir.
