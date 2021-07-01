# HTTP

ここまでの内容では、linkedin.comのIPアドレスしかわかりませんでした。linkin.comのHTMLページはHTTPプロトコルで提供されており、ブラウザはこれをレンダリングします。ブラウザは、上記で判明したサーバーのIPアドレスにHTTPリクエストを送信します。
リクエストには、GET、PUT、POSTという動詞に続いて、パス、クエリパラメータに加え、クライアントに関する情報や、受け入れ可能なコンテンツなどのクライアントの能力を示すキーバリューペアの行と、（通常、POSTまたはPUTの場合）ボディがあります。

```bash
# コンテナで以下を実行して、ヘッダーを見てみましょう。
curl linkedin.com -v
```
```bash
* Connected to linkedin.com (108.174.10.10) port 80 (#0)
> GET / HTTP/1.1
> Host: linkedin.com
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 09 Nov 2020 10:39:43 GMT
< X-Li-Pop: prod-esv5
< X-LI-Proto: http/1.1
< Location: https://www.linkedin.com/
< Content-Length: 0
< 
* Connection #0 to host linkedin.com left intact
* Closing connection 0
```

ここで、1行目のGETは動詞、/はパス、1.1はHTTPプロトコルのバージョンを表しています。そして、クライアントの機能といくつかの詳細をサーバーに伝えるキーバリューペアがあります。サーバは、HTTPバージョン、[ステータスコードとステータスメッセージ](https://ja.wikipedia.org/wiki/HTTP%E3%82%B9%E3%83%86%E3%83%BC%E3%82%BF%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%89)で応答します。ステータスコードの2xxは成功を、3xxはリダイレクトを、4xxはクライアント側のエラーを、5xxはサーバー側のエラーを意味します。

HTTP/1.0とHTTP/1.1の違いについて説明します。

```bash
#ターミナルで次のように入力します
telnet www.linkedin.com 80
#telnet STDINの最後に空の改行を入れて、以下の内容をコピー＆ペーストします。
GET / HTTP/1.1
HOST:linkedin.com
USER-AGENT: curl

```


これによってサーバーからの応答が得られ、www.linkedin.comへのベースとなる接続が次の問い合わせに再利用できるため、次の入力を待ちます。TCPを使ってみると、そのメリットがよくわかります。しかし、HTTP/1.0では、この接続は応答の後すぐに閉じられるため、問い合わせのたびに新しい接続を開かなければなりません。HTTP/1.1では、開いている接続では同時に1つのリクエストしかできませんが、接続は複数のリクエストに次々と再利用できます。HTTP/1.1と比較したHTTP/2.0の利点の1つは、同じ接続で複数のリクエストを処理することが可能なことです。ここでは、一般的なHTTPに範囲を限定しているため、各プロトコルバージョンの複雑さには触れていませんが、コース終了後には簡単に理解できるようになっているはずです。

HTTPは**ステートレスプロトコル**と呼ばれています。ここでは、ステートレスとは何を意味するのかを説明します。例えば、私たちがlinkedin.comにログインしたとすると、クライアントからlinkedin.comへの各リクエストは、ユーザーのコンテキストを持たず、各ページ/リソースごとにユーザーにログインを促すことは意味がありません。このようなHTTPの問題を解決するのが、*COOKIE*です。ユーザーがログインすると、セッションが作成されます。このセッション識別子は、*SET-COOKIE*ヘッダーを介してブラウザに送信されます。ブラウザは、サーバーが設定した有効期限までCOOKIEを保存し、今後linkedin.comへのリクエストごとにクッキーを送信します。クッキーの詳細については、[こちら](https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies)をご覧ください。クッキーはパスワードと同様に重要な情報であり、HTTPはプレーンテキストプロトコルであるため、中間者はパスワードやクッキーのいずれかを捕捉することができ、ユーザーのプライバシーを侵害する可能性があります。同様に、DNSの項で述べたように、linkedin.comのIPを偽装すると、ユーザーがlinkedinのパスワードを入力して悪意のあるサイトにログインするというフィッシング攻撃を受ける可能性があります。この2つの問題を解決するために、HTTPSが導入され、HTTPSを義務づける必要があります。

HTTPSは、サーバー証明書と、クライアントとサーバー間のデータの暗号化を提供しなければなりません。サーバー管理者は、秘密鍵・公開鍵ペアと証明書要求を生成する必要があります。この証明書要求は、証明書要求を証明書に変換する認証局によって署名されなければなりません。サーバー管理者は、証明書と秘密鍵を更新してウェブサーバーに送る必要があります。証明書には、サーバーに関する詳細（サービスを提供するドメイン名、有効期限など）と、サーバーの公開鍵が含まれています。秘密鍵はサーバーの秘密であり、秘密鍵を失うとサーバーが提供する信頼を失うことになります。クライアントが接続して、HELLOと送信したとします。サーバーは証明書をクライアントに送ります。クライアントは、証明書の有効期限内であるか、信頼できる機関によって署名されているか、証明書に記載されているホスト名がサーバーと同じであるかなど、証明書の有効性を確認します。この検証により、サーバーが正しいものであり、フィッシングがないことが確認されます。それが検証されると、クライアントはサーバーの公開鍵でネゴシエーションを暗号化して、サーバーと対称的な鍵と暗号をネゴシエートします。秘密鍵を持っているサーバー以外の誰もこのデータを理解することはできません。ネゴシエーションが完了すると、その対称鍵とアルゴリズムがさらに暗号化に使用され、対称鍵とアルゴリズムを知っているのはクライアントとサーバーだけなので、それ以降は復号化することができます。非対称暗号化アルゴリズムから対称暗号化アルゴリズムへの切り替えは、一般的に対称暗号化は非対称暗号化よりもリソースを消費しないため、クライアントデバイスのリソースに負担をかけないためです。

```bash
#ターミナルで以下を実行すると、Subject Name(ドメイン名)、Issuer details、Expiry dateなどの証明書の詳細が表示されます。
curl https://www.linkedin.com -v 
```
```bash
* Connected to www.linkedin.com (13.107.42.14) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
} [230 bytes data]
* TLSv1.2 (IN), TLS handshake, Server hello (2):
{ [90 bytes data]
* TLSv1.2 (IN), TLS handshake, Certificate (11):
{ [3171 bytes data]
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
{ [365 bytes data]
* TLSv1.2 (IN), TLS handshake, Server finished (14):
{ [4 bytes data]
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
} [102 bytes data]
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.2 (OUT), TLS handshake, Finished (20):
} [16 bytes data]
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
{ [1 bytes data]
* TLSv1.2 (IN), TLS handshake, Finished (20):
{ [16 bytes data]
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=US; ST=California; L=Sunnyvale; O=LinkedIn Corporation; CN=www.linkedin.com
*  start date: Oct  2 00:00:00 2020 GMT
*  expire date: Apr  2 12:00:00 2021 GMT
*  subjectAltName: host "www.linkedin.com" matched cert's "www.linkedin.com"
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert SHA2 Secure Server CA
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fb055808200)
* Connection state changed (MAX_CONCURRENT_STREAMS == 100)!
  0 82117    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Connection #0 to host www.linkedin.com left intact
HTTP/2 200 
cache-control: no-cache, no-store
pragma: no-cache
content-length: 82117
content-type: text/html; charset=utf-8
expires: Thu, 01 Jan 1970 00:00:00 GMT
set-cookie: JSESSIONID=ajax:2747059799136291014; SameSite=None; Path=/; Domain=.www.linkedin.com; Secure
set-cookie: lang=v=2&lang=en-us; SameSite=None; Path=/; Domain=linkedin.com; Secure
set-cookie: bcookie="v=2&70bd59e3-5a51-406c-8e0d-dd70befa8890"; domain=.linkedin.com; Path=/; Secure; Expires=Wed, 09-Nov-2022 22:27:42 GMT; SameSite=None
set-cookie: bscookie="v=1&202011091050107ae9b7ac-fe97-40fc-830d-d7a9ccf80659AQGib5iXwarbY8CCBP94Q39THkgUlx6J"; domain=.www.linkedin.com; Path=/; Secure; Expires=Wed, 09-Nov-2022 22:27:42 GMT; HttpOnly; SameSite=None
set-cookie: lissc=1; domain=.linkedin.com; Path=/; Secure; Expires=Tue, 09-Nov-2021 10:50:10 GMT; SameSite=None
set-cookie: lidc="b=VGST04:s=V:r=V:g=2201:u=1:i=1604919010:t=1605005410:v=1:sig=AQHe-KzU8i_5Iy6MwnFEsgRct3c9Lh5R"; Expires=Tue, 10 Nov 2020 10:50:10 GMT; domain=.linkedin.com; Path=/; SameSite=None; Secure
x-fs-txn-id: 2b8d5409ba70
x-fs-uuid: 61bbf94956d14516302567fc882b0000
expect-ct: max-age=86400, report-uri="https://www.linkedin.com/platform-telemetry/ct"
x-xss-protection: 1; mode=block
content-security-policy-report-only: default-src 'none'; connect-src 'self' www.linkedin.com www.google-analytics.com https://dpm.demdex.net/id lnkd.demdex.net blob: https://linkedin.sc.omtrdc.net/b/ss/ static.licdn.com static-exp1.licdn.com static-exp2.licdn.com static-exp3.licdn.com; script-src 'sha256-THuVhwbXPeTR0HszASqMOnIyxqEgvGyBwSPBKBF/iMc=' 'sha256-PyCXNcEkzRWqbiNr087fizmiBBrq9O6GGD8eV3P09Ik=' 'sha256-2SQ55Erm3CPCb+k03EpNxU9bdV3XL9TnVTriDs7INZ4=' 'sha256-S/KSPe186K/1B0JEjbIXcCdpB97krdzX05S+dHnQjUs=' platform.linkedin.com platform-akam.linkedin.com platform-ecst.linkedin.com platform-azur.linkedin.com static.licdn.com static-exp1.licdn.com static-exp2.licdn.com static-exp3.licdn.com; img-src data: blob: *; font-src data: *; style-src 'self' 'unsafe-inline' static.licdn.com static-exp1.licdn.com static-exp2.licdn.com static-exp3.licdn.com; media-src dms.licdn.com; child-src blob: *; frame-src 'self' lnkd.demdex.net linkedin.cdn.qualaroo.com; manifest-src 'self'; report-uri https://www.linkedin.com/platform-telemetry/csp?f=g
content-security-policy: default-src *; connect-src 'self' https://media-src.linkedin.com/media/ www.linkedin.com s.c.lnkd.licdn.com m.c.lnkd.licdn.com s.c.exp1.licdn.com s.c.exp2.licdn.com m.c.exp1.licdn.com m.c.exp2.licdn.com wss://*.linkedin.com dms.licdn.com https://dpm.demdex.net/id lnkd.demdex.net blob: https://accounts.google.com/gsi/status https://linkedin.sc.omtrdc.net/b/ss/ www.google-analytics.com static.licdn.com static-exp1.licdn.com static-exp2.licdn.com static-exp3.licdn.com media.licdn.com media-exp1.licdn.com media-exp2.licdn.com media-exp3.licdn.com; img-src data: blob: *; font-src data: *; style-src 'unsafe-inline' 'self' static-src.linkedin.com *.licdn.com; script-src 'report-sample' 'unsafe-inline' 'unsafe-eval' 'self' spdy.linkedin.com static-src.linkedin.com *.ads.linkedin.com *.licdn.com static.chartbeat.com www.google-analytics.com ssl.google-analytics.com bcvipva02.rightnowtech.com www.bizographics.com sjs.bizographics.com js.bizographics.com d.la4-c1-was.salesforceliveagent.com slideshare.www.linkedin.com https://snap.licdn.com/li.lms-analytics/ platform.linkedin.com platform-akam.linkedin.com platform-ecst.linkedin.com platform-azur.linkedin.com; object-src 'none'; media-src blob: *; child-src blob: lnkd-communities: voyager: *; frame-ancestors 'self'; report-uri https://www.linkedin.com/platform-telemetry/csp?f=l
x-frame-options: sameorigin
x-content-type-options: nosniff
strict-transport-security: max-age=2592000
x-li-fabric: prod-lva1
x-li-pop: afd-prod-lva1
x-li-proto: http/2
x-li-uuid: Ybv5SVbRRRYwJWf8iCsAAA==
x-msedge-ref: Ref A: CFB9AC1D2B0645DDB161CEE4A4909AEF Ref B: BOM02EDGE0712 Ref C: 2020-11-09T10:50:10Z
date: Mon, 09 Nov 2020 10:50:10 GMT

* Closing connection 0
```

ここで、システムは信頼する認証局のリストを/etc/ssl/cert.pemに持っています。curlは、証明書のサブジェクト部分のCNセクションを見ることで、証明書がwww.linkedin.comのものであることを検証します。また、証明書の有効期限を確認することで、証明書が期限切れでないことを確認します。また、/etc/ssl/cert.pemにある発行者DigiCertの公開鍵を使用して、証明書の署名を検証します。これが完了すると、www.linkedin.comの公開鍵を使って、対称鍵を持つ暗号TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384をネゴシエートします。最初のHTTPリクエストを含むその後のデータ転送は、同じ暗号と対称鍵を使用します。


