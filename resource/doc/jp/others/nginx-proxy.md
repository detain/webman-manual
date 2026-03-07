# nginxプロキシ

webmanが外部ネットワークへの直接アクセスを提供する必要がある場合、webmanの前にnginxプロキシを追加することをお勧めします。これには以下の利点があります。

- 静的リソースはnginxが処理し、webmanはビジネスロジックの処理に専念できます
- 複数のwebmanが80、443ポートを共有し、ドメイン名でサイトを区別して、1台のサーバーで複数サイトを展開できます
- php-fpmとwebmanアーキテクチャの共存が可能です
- nginxプロキシでSSLのhttps実装がより簡単で効率的です
- 外部からの不正なリクエストを厳密にフィルタリングできます

## nginxプロキシの例
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name サイトのドメイン;
  listen 80;
  access_log off;
  # 注意：rootは必ずwebmanのpublicディレクトリを指定すること。webmanのルートディレクトリではない
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

  # .phpで終わるすべてのファイルへのアクセスを拒否
  location ~ \.php$ {
      return 404;
  }

  # .well-knownディレクトリへのアクセスを許可
  location ~ ^/\.well-known/ {
    allow all;
  }

  # .で始まるすべてのファイル・ディレクトリへのアクセスを拒否
  location ~ /\. {
      return 404;
  }

}
```

一般的に、開発者はserver_nameとrootを実際の値に設定するだけでよく、他のフィールドは設定する必要はありません。

> **注意**
> rootオプションは必ずwebmanのpublicディレクトリを指定してください。webmanのルートディレクトリに設定してはいけません。そうしないと、データベース設定などの機密ファイルを含むすべてのファイルがインターネットからダウンロード・アクセス可能になってしまいます。
