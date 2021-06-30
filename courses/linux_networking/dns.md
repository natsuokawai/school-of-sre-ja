# DNS
ドメイン名とは、ウェブサイトの名前を人間が読めるようにしたものです。インターネットはIPアドレスしか理解できませんが、支離滅裂な数字を覚えるのは現実的ではないため、代わりにドメイン名が使われます。これらのドメイン名は、DNSのインフラによってIPアドレスに変換されます。誰かがブラウザで[www.linkedin.com](https://www.linkedin.com)を開こうとすると、ブラウザは[www.linkedin.com](https://www.linkedin.com)をIPアドレスに変換しようとします。この処理をDNS解決といいます。このプロセスを描いた簡単な疑似コードは次のようになります。

```python
ip, err = getIPAddress(domainName)
if err:
    print("unknown Host Exception while trying to resolve:%s".format(domainName))
```

では、getIPAddress関数の中で何が起こっているのかを理解してみましょう。ブラウザは自分自身のDNSキャッシュを持っていて、ドメイン名とIPアドレスのマッピングがすでに存在するかどうかをチェックします。そのようなマッピングが存在しない場合、ブラウザはgethostbynameシステムコールを呼び出して、与えられたドメイン名に対応するIPアドレスを見つけるようにオペレーティングシステムに依頼します。

```python
def getIPAddress(domainName):
    resp, fail = lookupCache(domainName)
    If not fail:
        return resp
    else:
        resp, err = gethostbyname(domainName)
        if err:
            return null, err
        else:
            return resp
```

ここで、[gethostbyname](https://man7.org/linux/man-pages/man3/gethostbyname.3.html)関数が呼び出されたときに、オペレーティングシステムのカーネルが何をするかを理解しましょう。Linuxのオペレーティングシステムは、[/etc/nsswitch.conf](https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html)というファイルを見ます。

```bash
hosts:      files dns
```

この行は、OSが最初にファイル（/etc/hosts）を検索し、/etc/hostsに一致するものがない場合にDNSプロトコルを使用して解決することを意味します。

ファイル/etc/hostsは次のような形式になっています。


IPAddress FQDN [FQDN].*

```bash
127.0.0.1 localhost.localdomain localhost
::1 localhost.localdomain localhost
```

このファイルの中のドメインにマッチするものがあれば、そのIPアドレスがOSから返されます。このファイルに一行追加してみましょう。

```bash
127.0.0.1 test.linkedin.com
```

そして、test.linkedin.comにpingを実行します。

```bash
ping test.linkedin.com -n
```

```bash
PING test.linkedin.com (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.036 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.037 ms

```

前述のように、/etc/hostsにマッチするものがない場合、OSはDNSプロトコルを使ってDNS解決を試みます。Linuxシステムは、/etc/resolv.confに記載されている最初のIPに対してDNSリクエストを行います。応答がない場合は、resolv.conf内の後続のサーバーに要求が送られます。これらのresolv.conf内のサーバーをDNSリゾルバと呼びます。DNSリゾルバは、[DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)によって設定されるか、管理者によって静的に設定されます。
[Dig](https://linux.die.net/man/1/dig)はユーザー空間のDNSシステムで、DNSリゾルバを作成してリクエストを送り、受け取ったレスポンスをコンソールに表示します。

```bash
# DNSリクエストを取得するには、次のコマンドをシェルで実行します
sudo tcpdump -s 0 -A -i any port 53
# 別のシェルからdigリクエストを行います
dig linkedin.com
```

```bash 
13:19:54.432507 IP 172.19.209.122.56497 > 172.23.195.101.53: 527+ [1au] A? linkedin.com. (41)
....E..E....@.n....z...e...5.1.:... .........linkedin.com.......)........
13:19:54.485131 IP 172.23.195.101.53 > 172.19.209.122.56497: 527 1/0/1 A 108.174.10.10 (57)
....E..U..@.|.	....e...z.5...A...............linkedin.com..............3..l.

..)........
```


パケットキャプチャを見ると、172.23.195.101:53（これは/etc/resolv.confのリゾルバ）にlinkedin.comのリクエストが行われ、172.23.195.101からlinkedin.comのIPアドレス108.174.10.10のレスポンスを受信していることがわかります。

それでは、DNSリゾルバがどのようにlinkedin.comのIPアドレスを見つけようとするのかを理解してみましょう。DNSリゾルバはまずキャッシュを調べます。ネットワーク上の多くのデバイスがlinkedin.comというドメイン名を照会できるため、名前解決の結果がすでにキャッシュに存在している可能性があります。キャッシュミスがあれば、DNS解決処理を開始します。DNSサーバーは「linkedin.com」を「.」、「com.」、「linkedin.com.」に分解し、「.」からDNS解決を開始します。「.」はルートドメインと呼ばれ、これらのIPはDNSリゾルバソフトウェアに知られています。DNSリゾルバは、ルートドメインのネームサーバに問い合わせて、「com.」の詳細について応答できる適切なネームサーバを探します。「com.」の権威DNSサーバーのアドレスが返されます。ここでDNS解決サービスは、「com.」の権威DNSサーバーに連絡して、「linkedin.com」の権威DNSサーバーを取得します。「linkedin.com」の権威DNSサーバーが判明すると、リゾルバはLinkedinのネームサーバーに連絡して「linkedin.com」のIPアドレスを提供します。このプロセスの全体像は、次のように実行するとわかります。

```bash
dig +trace linkedin.com
```


```bash
linkedin.com.		3600	IN	A	108.174.10.10
```
このDNSレスポンスには5つのフィールドがあり、最初のフィールドがリクエストで、最後のフィールドがレスポンスです。2番目のフィールドはTime to Liveで、DNSレスポンスがどれくらい有効かを秒単位で表しています。このケースでは、linkedin.comのマッピングは1時間有効です。このようにして、リゾルバとアプリケーション（ブラウザ）はキャッシュを維持しています。1時間を超えてlinkedin.comへのリクエストがあった場合、マッピングのTTLが切れているため、キャッシュミスとして扱われ、すべての処理をやり直さなければなりません。
4番目のフィールドは、DNSレスポンス/リクエストのタイプを示します。DNSクエリの種類には次のようなものがあります。
A, AAAA, NS, TXT, PTR, MX, CNAME
- Aレコードは、ドメイン名のIPV4アドレスを返します。
- AAAAレコードは、ドメイン名のIPV6アドレスを返します。
- NSレコードは、ドメイン名の権威DNSサーバーを返します。
- CNAMEレコードは、ドメイン名のエイリアスです。一部のドメインは他のドメイン名を指しており、後者のドメイン名を解決すると、前者のドメイン名のIPとしても使用されるIPが得られます。例: www.linkedin.comのIPアドレスは、2-01-2c3e-005a.cdx.cedexis.netと同じです。
- 簡潔にするために、他のDNSレコードタイプについては説明しませんが、これらのレコードのRFCは[こちら][(https://en.wikipedia.org/wiki/List_of_DNS_record_types](https://ja.wikipedia.org/wiki/DNS%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%BF%E3%82%A4%E3%83%97%E3%81%AE%E4%B8%80%E8%A6%A7))にあります。

```bash
dig A linkedin.com +short
108.174.10.10


dig AAAA linkedin.com +short
2620:109:c002::6cae:a0a


dig NS linkedin.com +short
dns3.p09.nsone.net.
dns4.p09.nsone.net.
dns2.p09.nsone.net.
ns4.p43.dynect.net.
ns1.p43.dynect.net.
ns2.p43.dynect.net.
ns3.p43.dynect.net.
dns1.p09.nsone.net.

dig www.linkedin.com CNAME +short
2-01-2c3e-005a.cdx.cedexis.net.
```
このようなDNSの基礎知識をもとに、SREがDNSを使用するケースを見てみましょう。

## SREの役割における応用

このセクションでは、SREがDNSから得られる一般的なソリューションのいくつかを取り上げます。

1. すべての企業は、イントラネットやデータベースなどの内部サービス、wikiなどの内部アプリケーションのために、内部DNSインフラを持たなければなりません。そのため、インフラチームがこれらのドメイン名のためにDNSインフラを維持する必要があります。このDNSインフラは、単一障害点にならないよう、最適化されスケールしなければなりません。内部DNSインフラの障害は、マイクロサービスのAPIコールの障害やその他の連鎖的な影響を引き起こす可能性があります。
2. DNSは、サービスを発見するためにも使用できます。例えば、ホスト名serviceb.internal.example.comは、example.com社の内部でservicebを実行しているインスタンスをリストアップすることができます。クラウドプロバイダーは、DNSディスカバリーを有効にするオプションを提供しています（[例](https://docs.aws.amazon.com/whitepapers/latest/microservices-on-aws/service-discovery.html#dns-based-service-discovery)）。
3. DNSは、クラウド事業者やCDN事業者がサービスを拡張するために使用します。Azure/AWSでは、ロードバランサーにIPアドレスではなくCNAMEが与えられます。エイリアスドメイン名のIPアドレスを変更することにより、スケーリング時にロードバランサーのIPアドレスを更新します。これが、このようなエイリアスドメインのAレコードが1分程度の短命である理由の一つです。
4. DNSは、企業が地理的に分散している場合に、クライアントが自分の所在地に近いIPアドレスを取得して、HTTP コールをより迅速に応答できるようにするためにも使用できます。
5. SREは、DNSインフラには検証がないため、これらの応答が偽装される可能性があることを理解する必要があります。DNS、HTTPS（後述）のような他のプロトコルによって保護されています。DNSSECは、偽造または操作された DNSレスポンスから保護します。
6. 古くなったDNSキャッシュは問題となりえます。一部の[アプリ](https://stackoverflow.com/questions/1256556/how-to-make-java-honor-the-dns-caching-timeout)では、APIの呼び出しに期限切れのDNSレコードを使用している場合があります。これは、SREがメンテナンスを行う際に注意しなければならないことです。
7. DNSロードバランシングとサービスディスカバリーはTTLを理解する必要があり、 DNSレコードに変更が加えられた後、TTLまで待って初めてサーバーをプールから削除することができます。これが行われないと、TTL前にサーバーが削除され、トラフィックの一部が失敗します。
