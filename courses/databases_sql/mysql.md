### MySQLアーキテクチャ

![alt_text](images/mysql_architecture.png "MySQLアーキテクチャ図")

MySQLアーキテクチャは、ニーズに合わせて適切なストレージエンジンを選択することができ、一貫性のある安定したAPIを知るだけでよいエンドユーザー（アプリケーションエンジニアや[DBA](https://en.wikipedia.org/wiki/Database_administrator)）からは、すべての実装の詳細が抽象化されています。

アプリケーション層：

* 接続処理：各クライアントは独自の接続を取得し、アクセスの間キャッシュされます。
* 認証：サーバーはクライアントの情報（ユーザー名、パスワード、ホスト）をチェックし、接続を許可/拒否します。
* セキュリティ：サーバーは、クライアントが各クエリを実行するための権限を持っているかどうかを判断します（`show privileges`コマンドで確認）。

サーバー層：

* サービスとユーティリティ：バックアップ/リストア、レプリケーション、クラスタなど
* SQLインターフェイス：クライアントはデータへのアクセスと操作のためにクエリを実行します。
* SQLパーサー：クエリから解析ツリーを作成 (字句/構文/意味解析とコード生成)
* オプティマイザ：様々なアルゴリズムと利用可能なデータ（テーブルレベルの統計情報）を使用してクエリを最適化し、クエリ、スキャンの順序、使用するインデックスなどを修正します。(explanationコマンドで確認)
* キャッシュとバッファ：キャッシュはクエリの結果を保存し、バッファプール(InnoDB)はテーブルとインデックスのデータを[LRU](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))で保存します。

ストレージエンジンのオプション：

* InnoDB：最も広く使用されており、トランザクションサポート、ACID準拠、行レベルロック、クラッシュリカバリ、マルチバージョンコンカレンシーコントロールをサポートしています。MySQL 5.5以降のデフォルトです。
* MyISAM：高速で、トランザクションをサポートせず、テーブルレベルのロッキングを提供し、主にWebやデータウェアハウスなどの読み取りの多いワークロードに最適です。MySQL 5.1までのデフォルトです。
* Archive：高速挿入に最適化され、挿入時にデータを圧縮します。トランザクションをサポートしません。
* メモリ：メモリ内のテーブル。トランザクションをサポートしていません。一時テーブルの作成やクイックルックアップに最適ですが、シャットダウンするとデータは失われます。
* CSV：データをCSVファイルに格納します。このフォーマットを使用する他のアプリケーションとの統合に最適です。

...など

あるストレージエンジンから別のストレージエンジンに移行することは可能です。しかし、この移行はすべての操作に対してテーブルをロックし、データの物理的レイアウトを変更するため、オンラインではありません。移行には長い時間がかかり、一般的にはお勧めできません。したがって、最初に正しいストレージエンジンを選ぶことが重要です。

一般的なガイドラインとしては、他のストレージエンジンに特別な必要性がない限り、InnoDBを使用することです。

`mysql> SHOW ENGINES;`を実行すると、あなたのMySQLサーバでサポートされているエンジンが表示されます。