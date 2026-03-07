# Nginx 프록시

webman이 외부 네트워크 직접 접근을 제공해야 할 때, webman 앞에 nginx 프록시를 추가하는 것이 좋습니다. 이렇게 하면 다음과 같은 이점이 있습니다.

- 정적 자원은 nginx가 처리하여 webman이 비즈니스 로직 처리에 집중할 수 있습니다
- 여러 webman 인스턴스가 80, 443 포트를 공유하고, 도메인 이름으로 사이트를 구분하여 단일 서버에 여러 사이트를 배포할 수 있습니다
- php-fpm과 webman 아키텍처의 공존이 가능합니다
- nginx 프록시의 SSL https 구현이 더 간단하고 효율적입니다
- 외부의 불법 요청을 엄격히 필터링할 수 있습니다

## Nginx 프록시 예시
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name 사이트_도메인;
  listen 80;
  access_log off;
  # 중요: root는 반드시 webman의 public 디렉토리를 가리켜야 하며, webman 루트 디렉토리가 아니어야 합니다
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

  # .php로 끝나는 모든 파일에 대한 접근 거부
  location ~ \.php$ {
      return 404;
  }

  # .well-known 디렉토리에 대한 접근 허용
  location ~ ^/\.well-known/ {
    allow all;
  }

  # .으로 시작하는 모든 파일 또는 디렉토리에 대한 접근 거부
  location ~ /\. {
      return 404;
  }

}
```

일반적으로 개발자는 server_name과 root만 실제 값으로 설정하면 되며, 다른 필드는 설정할 필요가 없습니다.

> **참고**
> root 옵션은 반드시 webman의 public 디렉토리를 가리켜야 합니다. 절대 webman 루트 디렉토리로 설정하지 마세요. 그렇지 않으면 데이터베이스 설정 등 민감한 파일을 포함한 모든 파일이 인터넷에서 다운로드 및 접근 가능해질 수 있습니다.
