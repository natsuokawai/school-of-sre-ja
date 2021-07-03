# PythonとWebとFlask

昔を振り返ると、ウェブサイトはシンプルで、単純な静的HTMLコンテンツでできていました。ウェブサーバーは定義されたポートで待ち受けており、受け取ったHTTPリクエストに応じて、ディスクからファイルを読み込んで応答を返していました。しかし、それ以来複雑さを増していき、ウェブサイトは動的になりました。リクエストに応じて、データベースからの読み込みや他のAPIの呼び出しなど、複数の処理を実行し、最終的に何らかのレスポンス（HTMLデータやJSONコンテンツなど）を返す必要があります。

Webリクエストを処理することは、もはやディスクからファイルを読み込んでコンテンツを返すような単純な作業ではないため、それぞれのHTTPリクエストを処理し、プログラムでいくつかの操作を行ってレスポンスを構築する必要があります。

## ソケット

Flaskのようなフレームワークがあるとはいえ、HTTPはTCPプロトコルで動作するプロトコルであることに変わりはありません。そこで、TCPサーバーを立ち上げ、HTTPリクエストを送信し、リクエストのペイロードを検査してみましょう。これはソケットプログラミングのチュートリアルではありませんが、ここでやっていることは、HTTPプロトコルを基礎レベルで検査し、その内容がどのようなものかを見ることです。(参考: [Socket Programming in Python (Guide) on RealPython](https://realpython.com/python-sockets/))

```python
import socket

HOST = '127.0.0.1' # 標準的なループバックインターフェースのアドレス（localhost）
PORT = 65432 # リッスンするポート（特権的でないポートは > 1023）

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
   s.bind((HOST, PORT))
   s.listen()
   conn, addr = s.accept()
   with conn:
       print('Connected by', addr)
       while True:
           data = conn.recv(1024)
           if not data:
               break
           print(data)
```

次に、Webブラウザで`localhost:65432`を開くと、以下のような出力が得られます。

``bash
Connected by ('127.0.0.1', 54719)
b'GET / HTTP/1.1\r\nHost: localhost:65432\r\nConnection: keep-alive\r\nDNT: 1\r\nUpgrade-Insecure-Requests: 1\r\nUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36 Edg/85.0.564.44\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\nSec-Fetch-Site: none\r\nSec-Fetch-Mode: navigate\r\nSec-Fetch-User: ?1\r\nSec-Fetch-Dest: document\r\nAccept-Encoding: gzip, deflate, br\r\nAccept-Language: en-US,en;q=0.9\r\n\r\n'
```

よく見ると、HTTPプロトコルのフォーマットに沿った内容になっています。

```
HTTP_METHOD URI_PATH HTTP_VERSION
HEADERS_SEPARATED_BY_SEPARATOR
```

これはバイトの塊ですが、[HTTPの仕様](https://tools.ietf.org/html/rfc2616)を知っていれば、その文字列を解析して（つまり、`\r\n`で分割して）、意味のある情報を得ることができます。

## Flask

Flaskやその他のフレームワークは、（より洗練されていますが）前のセクションで説明したこととほぼ同じことを行います。TCPソケットのポートを待ち受け、HTTPリクエストを受け取り、プロトコルフォーマットに従ってデータを解析し、便利な方法で利用できるようにします。

例えば、HTTPプロトコルで定義されているように、ペイロードを`\r\n`で分割することで利用可能になる `request.headers`によって、Flaskでヘッダにアクセスすることができます。

別の例として、我々は`@app.route("/hello")`によってFlaskにルートを登録します。Flaskは内部的にレジストリを維持し、`/hello`をあなたが装飾した関数にマッピングします。これで、`/hello`ルート（最初の行の2番目のコンポーネント、スペースで分割されている）を持つリクエストが来るたびにFlaskは登録された関数を呼び出し、その関数が返したものを返します。

他の言語のウェブフレームワークでも同じで、すべて似たような原理で動いています。基本的には、HTTPプロトコルを理解し、HTTPリクエストのデータを解析し、プログラマにHTTPリクエストを扱うための便利なインターフェイスを提供しています。

魔法というほどのものではありませんが、いかがでしょうか？
