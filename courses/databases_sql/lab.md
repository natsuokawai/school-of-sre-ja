**前提条件**

Dockerがインストールされていること


**セットアップ**

sosなどの作業用ディレクトリを作成し、そこにcdで移動します。

customというディレクトリにy.cnfというファイルに以下の内容を入力します。


```
sos $ cat custom/my.cnf
[mysqld]
# These settings apply to MySQL server
# You can set port, socket path, buffer size etc.
# Below, we are configuring slow query settings
slow_query_log=1
slow_query_log_file=/var/log/mysqlslow.log
long_query_time=1
```


コンテナを起動し、以下のようにスロークエリログを有効にします。


```
sos $ docker run --name db -v custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=realsecret -d mysql:8
sos $ docker cp custom/my.cnf $(docker ps -qf "name=db"):/etc/mysql/conf.d/custom.cnf
sos $ docker restart $(docker ps -qf "name=db")
```


サンプルデータベースのインポート


```
sos $ git clone git@github.com:datacharmer/test_db.git
sos $ docker cp test_db $(docker ps -qf "name=db"):/home/test_db/
sos $ docker exec -it $(docker ps -qf "name=db") bash
root@3ab5b18b0c7d:/# cd /home/test_db/
root@3ab5b18b0c7d:/# mysql -uroot -prealsecret mysql < employees.sql
root@3ab5b18b0c7d:/etc# touch /var/log/mysqlslow.log
root@3ab5b18b0c7d:/etc# chown mysql:mysql /var/log/mysqlslow.log
```


_ワークショップ1：サンプルクエリの実行_
以下を実行してください。
```
$ mysql -uroot -prealsecret mysql
mysql>

# DBとテーブルの検査
# 最後の4つはMySQLの内部DB

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

> use employees;
mysql> show tables;
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+

# 数行を読み込む
mysql> select * from employees limit 5;

# 条件によるデータのフィルタリング
mysql> select count(*) from employees where gender = 'M' limit 5;

# 特定のデータのカウントを見つける
mysql> select count(*) from employees where first_name = 'Sachin'; 
```

_ワークショップ2：explainとexplain analyzeを使用してクエリをプロファイリングし、パフォーマンスを向上させるために必要なインデックスを特定して追加する。_
```
# テーブルのインデックスを表示
#（'\G'をつけると結果を縦方向に表示します。';'に置き換えるとテーブル出力になります。）
mysql> show index from employees from employees\G
*************************** 1. row ***************************
        Table: employees
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: emp_no
    Collation: A
  Cardinality: 299113
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
      Visible: YES
   Expression: NULL

# このクエリは、'key'フィールドで識別されるインデックスを使用しています。
# コマンドの前にexplainキーワードを付けることで
# クエリプラン（使用されたキーを含む）を取得します。
mysql> explain select * from employees where emp_no < 10005\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using where

# インデックスを利用していない次のクエリと比較してみましょう。
mysql> explain select first_name, last_name from employees where first_name = 'Sachin'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 299113
     filtered: 10.00
        Extra: Using where

# このクエリにどれだけの時間がかかるか見てみましょう
mysql> explain analyze select first_name, last_name from employees where first_name = 'Sachin'\G
*************************** 1. row ***************************
EXPLAIN: -> Filter: (employees.first_name = 'Sachin')  (cost=30143.55 rows=29911) (actual time=28.284..3952.428 rows=232 loops=1)
    -> Table scan on employees  (cost=30143.55 rows=299113) (actual time=0.095..1996.092 rows=300024 loops=1)


# （クエリプランナーによって予測された）コストは30143.55です
# actual time=28.284ms for first row, 3952.428 for all rows
# それでは、インデックスを追加して再度クエリを実行してみましょう。
mysql> create index idx_firstname on employees(first_name);
Query OK, 0 rows affected (1.25 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain analyze select first_name, last_name from employees where first_name = 'Sachin';
+--------------------------------------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                                                    |
+--------------------------------------------------------------------------------------------------------------------------------------------+
| -> Index lookup on employees using idx_firstname (first_name='Sachin')  (cost=81.20 rows=232) (actual time=0.551..2.934 rows=232 loops=1)
 |
+--------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

# 1行目の実時間=0.551ms
# すべての行に関して2.934ms。大幅な改善です！
# また、このクエリはインデックス検索のみで、テーブルスキャン（テーブルの全行を読み取ること）がないことにも注目してください。
# DBの負荷が大幅に軽減されています。
```

_ワークショップ3：MySQLサーバ上でスロークエリを特定する_
```
# 以下のコマンドを2つのターミナルタブで実行し、コンテナ内に2つのシェルを開きます。
docker exec -it $(docker ps -qf "name=db") bash

# そのうちの一つでmysqlプロンプトを開き、以下のコマンドを実行します。
# 1秒以上かかったクエリをログに記録するように設定しています。
# したがって、このsleep(3)はログに記録されます。
mysql -uroot -prealsecret mysql
mysql> select sleep(3);

# ここで、もう一方のターミナルで、クエリの詳細を見つけるために、スローログをtailします。
root@62c92c89234d:/etc# tail -f /var/log/mysqlslow.log
/usr/sbin/mysqld, Version: 8.0.21 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
# Time: 2020-11-26T14:53:44.822348Z
# User@Host: root[root] @ localhost []  Id:     9
# Query_time: 5.404938  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use employees;
# Time: 2020-11-26T14:53:58.015736Z
# User@Host: root[root] @ localhost []  Id:     9
# Query_time: 10.000225  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1606402428;
select sleep(3);
```

これらは複雑さを最小限に抑えたシミュレーションの例でした。実際にはクエリはもっと複雑で、explanation/analyzeやスロークエリのログにはもっと詳細な情報があるでしょう。
