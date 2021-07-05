* リレーショナルDBはデータの保存に使われます。ファイルでもデータを保存することはできますが、リレーショナルDBは特定の目的のために設計されています。
    * 効率性
    * アクセスや管理のしやすさ
    * 規則正しい
    * データ間の関係を扱う（テーブルとして表現される）

* トランザクション：複数のステートメントをまとめて実行することができる作業単位

* ACID特性

    DBトランザクションのデータ整合性を保証する一連の特性

    * 原子性（atomicity）：各トランザクションは不可分（完全に成功または失敗するかのどちらか）である
    * 一貫性（consistency）：トランザクションは有効な状態（ルール、制約、トリガーなどを含む）にのみなる
    * 隔離性（isolation）：各トランザクションは、他のトランザクションとは独立して、並行システム内で安全に実行される
    * 耐久性（durability）：完了したトランザクションは、後の障害によって失われることはない

	上記の性質を説明するために、いくつかの例を挙げてみましょう。

    * 口座Aの残高は200円、Bの残高は400円です。この取引では、送信者からの控除と受信者の残高への追加が行われます。最初の操作が成功し、2番目の操作が失敗した場合、Aの残高は100円で、Bの残高は500円ではなく400円となります。DBの**原子性**により、この部分的に失敗したトランザクションは確実にロールバックされます。
    * 上記の2つ目の操作が失敗した場合、DBは矛盾した状態になります（操作前と操作後の口座の残高の合計が同じではありません）。**一貫性**はこのようなことが起こらないようにします。
    * Aの口座の利息を計算する操作、Aの口座に利息を追加する操作、BからAに100円を送金する操作の3つがあります。**隔離性**が保証されていないと、これら3つの操作を同時に実行すると、毎回異なる結果になる可能性があります。
    * トランザクションがディスクに書き込まれる前に、システムがクラッシュしたらどうなるでしょう？復旧時に変更が正しく適用されることを保証するのが**耐久性**です。

* リレーショナルデータ
    * テーブルは関係を表す
    * カラム（フィールド）は属性を表す
    * 行は個々のレコードである
    * スキーマはDBの構造を表す

* SQL

    データを操作、管理するためのクエリ言語。

    [CRUD操作](https://stackify.com/what-are-crud-operations/) - 作成（create）、読み取り（read）、更新（update）、削除（delete）のクエリ

    管理操作 - DB/テーブル/インデックスなどの作成、バックアップ、インポート/エクスポート、ユーザー、アクセス制御

    練習問題：以下のクエリをDDL（定義）、DML（操作）、DCL（制御）、TCL（トランザクション）の4種類に分類し、詳細に説明しなさい。

        insert、create、drop、delete、update、commit、rollback、truncate、alter、gant、revoke

    これらは[ラボセクション](/databases_sql/lab/)で練習することができます。



* 制約

    保存可能なデータのルール。テーブルに定義された制約のいずれかに違反すると、クエリは失敗します。


	主キー：ユニークな値を含む1つ以上の列で、NULL値を含むことはできません。1つのテーブルには1つの主キーしかありません。主キーに対するインデックスはデフォルトで作成されます。

    外部キー：2つのテーブルを結びつけます。その値は別のテーブルの主キーと一致します。
    
	NOT NULL：NULL値を許可しません。
	
    ユニーク：列の値はすべての行で一意でなければなりません。  
	
    デフォルト：挿入時にカラムが指定されていない場合、デフォルト値を提供します。  
    
    チェック：特定の値のみを許可します。（Balance >= 0 など）



* [インデックス](https://datageek.blog/en/2018/06/05/rdbms-basics-indexes-and-clustered-indexes/)

	ほとんどのインデックスはB+木構造を使用しています。

	使う理由：クエリの高速化（数行しか取得しない大規模なテーブル、最小/最大クエリ、検討対象から行を除外することなど）

	インデックスの種類：ユニーク、プライマリキー、フルテキスト、セカンダリ

	書き込みの多い負荷、主にフルテーブルスキャンや大量の行へのアクセスなどはインデックスの恩恵を受けません



* [結合（JOIN）](https://www.sqlservertutorial.net/sql-server-basics/sql-server-joins/)

	複数のテーブルから関連するデータを取得し、共通のフィールドでリンクすることができます。強力ですが、リソースを大量に消費するため、データベースの拡張が難しくなります。これは、大規模なクエリを実行したときにパフォーマンスが低下する多くの原因であり、解決策はほとんどの場合、結合を削減する方法を見つけることです。



* [アクセス制御](https://dev.mysql.com/doc/refman/8.0/en/access-control.html)

	DBには、管理者用の特権アカウントと、クライアント用の通常アカウントがあります。これらのアカウントにどのようなアクション（前述のDDL、DMLなど）が許されるかを細かく制御しています。

	DBはまずユーザーの資格を確認し（認証）、次にそのユーザーがリクエストを実行することを許可されているかどうかを内部テーブルで調べます（承認）。

	他にも、ユーザの行動履歴を調べるアクティビティ監査や、クエリ数や接続数などを制限するリソース制限などの制御があります。


### 人気のあるデータベース

商用のクローズドソース - Oracle、Microsoft SQL Server、IBM DB2

オープンソース - MySQL、MariaDB、PostgreSQLなど

個人や小規模な企業では、商用ソフトでは膨大なコストがかかるため、オープンソースのDBが好まれてきました。

最近では大企業でも、柔軟性とコスト削減の観点から商用ソフトウェアからオープンソースの代替品に移行しています。

また、開発者やサードパーティによる有償サポートが利用できるため、サポート不足の心配もありません。

MySQLは最も広く使用されているオープンソースのDBであり、ホスティングプロバイダによって広くサポートされているため、誰でも簡単に使用することができます。2000年代に普及したLinux-Apache-MySQL-PHP（[LAMP](https://ja.wikipedia.org/wiki/LAMP_(%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%83%90%E3%83%B3%E3%83%89%E3%83%AB))）スタックの一部です。プログラミング言語の選択肢は増えましたが、このスタックの残りの部分はまだ広く使われています。