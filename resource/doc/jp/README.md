# webmanとは

webmanはWorkermanをベースに構築された高性能サービスフレームワークで、HTTP、WebSocket、TCP、UDPなど複数のモジュールを統合しています。常駐メモリ、コルーチン、コネクションプールといった先進技術により、webmanは従来のPHPのパフォーマンスボトルネックを突破するだけでなく、その応用範囲も大きく広げています。

また、webmanは強力なプラグイン機構を備えており、開発者は他の開発者が作成した機能モジュールを素早く統合・再利用できます。ウェブサイト構築、HTTP API開発、インスタントメッセージング、IoTシステム、ゲーム、TCP/UDPサービス、Unix Socketサービスなど、あらゆる用途に対応し、優れたパフォーマンスと柔軟性を発揮します。

# webmanのコンセプト
**最小のコアで最大の拡張性と最高のパフォーマンスを提供する。**

webmanは最も基本的な機能（ルーティング、ミドルウェア、セッション、カスタムプロセスインターフェース）のみを提供し、その他はすべてComposerエコシステムを再利用します。つまり、webmanではお馴染みのコンポーネントを使えます。例として、データベースにはLaravelの[illuminate/database](./db/tutorial.md)、ThinkPHPの[ThinkORM](./db/thinkorm.md)、あるいは`Medoo`などを選べます。webmanへの統合も簡単です。

# webmanの特徴

1. 高い安定性。webmanはworkermanをベースにしており、業界でも数少ないバグで高い安定性を持つソケットフレームワークです。

2. 超高性能。webmanのパフォーマンスは従来のphp-fpmフレームワークより10〜100倍高く、Goのginやechoより約2倍高いです。

3. 高い再利用性。既存のComposerエコシステムをそのまま再利用できます。

4. 高い拡張性。カスタムプロセスをサポートし、workermanでできることは何でも可能です。

5. 非常にシンプルで使いやすく、学習コストが低く、コードの書き方は従来フレームワークと変わりません。

6. [バイナリパッケージング](./others/bin.md)に対応し、PHP環境なしで直接実行できます。

7. 最も緩やかで開発者フレンドリーなMITオープンソースライセンスを採用しています。

# プロジェクトURL
GitHub: https://github.com/walkor/webman **星を忘れずに！**

Gitee: https://gitee.com/walkor/webman **星を忘れずに！**

# 第三者によるベンチマークデータ

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

データベースクエリ業務では、webmanは単一マシンで最大39万QPSを達成し、従来のphp-fpmアーキテクチャのLaravelフレームワークと比較して約80倍です。

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

データベースクエリ業務では、webmanは同種のGo Webフレームワークより約2倍のパフォーマンスを発揮します。

上記データは[techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)より。
