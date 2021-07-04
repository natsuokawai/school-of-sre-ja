# URL短縮アプリ

Flaskを使って非常にシンプルなURL短縮アプリを作り、信頼性の面も含めて開発プロセスのすべての側面を取り入れてみましょう。UIは構築せず、アプリが十分に機能するための最小限のAPIを考えます。

## 設計

いきなりコーディングに飛びつくことはしません。まず最初にすることは、要件を集めることです。アプローチを考えましょう。アプローチや設計を同僚にレビューしてもらいましょう。進化させ、反復させ、決定事項とトレードオフを文書化しましょう。そして、最後に実装します。ここでは本格的な設計書は作成しませんが、設計にあたって重要となる問題提起をします。

### 1. 高度なオペレーションとAPIエンドポイント

URL短縮アプリなので、元のリンクから短縮リンクを生成するためのAPIが必要になります。また、短縮されたリンクを受け取り、元のURLにリダイレクトするAPI（エンドポイント）も必要です。物事を最小限にするため、アプリのUIは含めていません。この2つのAPIでアプリを機能するようにし、誰でも使えるようにする必要があります。

### 2. 短縮するには？

URLが与えられたら、その短縮版を生成する必要があります。一つの方法は、各リンクにランダムな文字を使用することです。もう一つの方法は、ある種のハッシュアルゴリズムを使用することです。ここでの利点は、同じリンクに対して同じハッシュを再利用することです。例えば、多くの人が`https://www.linkedin.com`を短縮した場合、ランダムな文字を選択した場合にDBに複数のエントリがあるのと比較して、同じ値を持つことになります。

ハッシュの衝突についてはどうでしょうか？ランダムな文字を使う方法でも、確率は低いですがハッシュの衝突は起こり得るので、注意する必要があります。その場合には、文字列の前にランダムな値を付加して衝突を回避することも考えられます。

また、ハッシュアルゴリズムの選択も重要です。我々はアルゴリズムを分析する必要があります。それらのCPU要件とその特徴。最も適したものを選んでください。

### 3. URLは有効か？

短縮したいURLがある場合、そのURLが有効かどうかをどうやって確認するのでしょうか？基本的なチェックとしては、そのURLがURLの正規表現と一致するかどうかを確認します。さらに進んで、URLを開いてみたり、訪問してみたりすることもできます。しかし、ここにはいくつかの問題があります。

1. 例えば、HTTP 200は有効であることを意味します。
2. URLがプライベートネットワーク内にある場合はどうするか？
3. URLが一時的にダウンした場合は？

### 4. ストレージ

最後にストレージです。時間の経過とともに生成されるデータをどこに保存するのでしょうか？複数のデータベースソリューションがありますが、このアプリに最も適したものを選択する必要があります。MySQLのようなリレーショナルデータベースを選択するのが妥当ですが、 **School of SREの[SQLデータベースのセクション](/databases_sql/intro/)と[NoSQLデータベースのセクション](/databases_nosql/intro/)をチェックして、より詳細な情報に基づいた決定を行ってください。**

### 5. その他

このアプリでは、ユーザーを考慮していません。レート制限やカスタマイズされたリンクなど、その他の機能も考えられますが、時間の経過とともに出てくるでしょう。要求に応じて、これらの機能を組み込む必要があるかもしれません。

参考までに最低限の動作コードを以下に示しますが、自分で考えてみることをお勧めします。

```python
from flask import Flask, redirect, request

from hashlib import md5

app = Flask("url_shortener")

mapping = {}

@app.route("/shorten", methods=["POST"])
def shorten():
    global mapping
    payload = request.json

    if "url" not in payload:
        return "Missing URL Parameter", 400

    # TODO: URLが有効かどうかのチェック

    hash_ = md5()
    hash_.update(payload["url"].encode())
    digest = hash_.hexdigest()[:5] # 5文字に制限しています。制限が少ないほど衝突の可能性が高くなります

    if digest not in mapping:
        mapping[digest] = payload["url"]
        return f"Shortened: r/{digest}\n"
    else:
        # TODO: ハッシュの衝突をチェックする
        return f"Already exists: r/{digest}\n"


@app.route("/r/<hash_>")
def redirect_(hash_):
    if hash_ not in mapping:
        return "URL Not Found", 404
    return redirect(mapping[hash_])


if __name__ == "__main__":
    app.run(debug=True)

"""
OUTPUT:


===> 短縮

$ curl localhost:5000/shorten -H "content-type: application/json" --data '{"url":"https://linkedin.com"}'
Shortened: r/a62a4


===> リダイレクト: レスポンスコード302とLocationに注目してください

$ curl localhost:5000/r/a62a4 -v
* Uses proxy env variable NO_PROXY == '127.0.0.1'
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 5000 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 5000 (#0)
> GET /r/a62a4 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.64.1
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 302 FOUND
< Content-Type: text/html; charset=utf-8
< Content-Length: 247
< Location: https://linkedin.com
< Server: Werkzeug/0.15.4 Python/3.7.7
< Date: Tue, 27 Oct 2020 09:37:12 GMT
<
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>Redirecting...</title>
<h1>Redirecting...</h1>
* Closing connection 0
<p>You should be redirected automatically to target URL: <a href="https://linkedin.com">https://linkedin.com</a>.  If not click the link.
```
