# 実行フロー

## プロセス起動フロー

`php start.php start` を実行した後の流れは以下の通りです：

1. config/ 以下の設定を読み込む
2. Worker 関連の設定を行う（`pid_file`、`stdout_file`、`log_file`、`max_package_size` など）
3. webman プロセスを作成し、ポート（デフォルト: 8787）で待ち受ける
4. 設定に基づいてカスタムプロセスを作成する
5. webman プロセスおよびカスタムプロセスが起動後、以下のロジックを実行する（いずれも `onWorkerStart` 内で実行）：
   ① `config/autoload.php` で設定されたファイルを読み込む（例：`app/functions.php`）
   ② `config/middleware.php`（`config/plugin/*/*/middleware.php` を含む）で設定されたミドルウェアを読み込む
   ③ `config/bootstrap.php`（`config/plugin/*/*/bootstrap.php` を含む）で設定されたクラスの `start` メソッドを実行し、Laravel データベース接続の初期化などモジュールを初期化する
   ④ `config/route.php`（`config/plugin/*/*/route.php` を含む）で定義されたルートを読み込む

## リクエスト処理フロー

1. リクエスト URL が public 以下の静的ファイルに対応するか判定する。対応する場合はファイルを返す（リクエスト終了）。対応しない場合は 2 へ
2. URL が何らかのルートにマッチするか判定する。マッチしない場合は 3 へ、マッチする場合は 4 へ
3. デフォルトルートが無効になっているか判定する。無効の場合は 404 を返す（リクエスト終了）。有効の場合は 4 へ
4. リクエストに対応するコントローラのミドルウェアを特定し、ミドルウェアの前処理を順に実行（オニオンモデル要求段階）、コントローラのビジネスロジックを実行、ミドルウェアの後処理を実行（オニオンモデル応答段階）してリクエストを終了する。（[ミドルウェアのオニオンモデル](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B)を参照）
