# アップグレード方法

`composer require workerman/webman-framework ^1.4.3 && composer require webman/console ^1.0.27 && php webman install`

> **注意**
> Alibaba Cloud の Composer プロキシは Composer 公式ソースからのデータ同期を停止しているため、現在、Alibaba Cloud Composer プロキシを使って最新の webman にアップグレードすることはできません。以下のコマンドを実行して Composer 公式データソースに戻してください: `composer config -g --unset repos.packagist`
