# IPルーティングとデータリンク層
ここでは、クライアントから出発したパケットがどのようにしてサーバーに到達するのか、またその逆はどのようにして行われるのかを説明します。パケットがIP層に到達すると、トランスポート層は送信元ポートと送信先ポートを入力します。IP/ネットワーク層は、宛先IP（DNSから発見します）を入力し、ルーティングテーブルで宛先IPへのルートを検索します。

```bash
#Linux の route -n コマンドは、デフォルトのルーティングテーブルを表示します。
route -n
```

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
```

宛先のIPとGenmaskをビットごとにANDし、その答えがテーブルの宛先部分（Destination）であれば、そのゲートウェイとインターフェイスがルーティングに選ばれます。ここで、linkedin.comのIPアドレス108.174.10.10は255.255.255.0とAND演算すると、得られる答えは108.174.10.0であるので、ルーティングテーブルのどの宛先とも一致しません。次に、Linuxは宛先IPと0.0.0.0をANDすると、0.0.0.0が得られます。これはデフォルトの行に一致します。

ルーティングテーブルは、Genmaskに設定された1のオクテットが多い順に処理され、マッチするものがなければgenmask 0.0.0.0がデフォルトルートとなります。
この操作の結果、Linuxはパケットをeth0経由でネクストホップの172.17.0.1に送る必要があると考えました。パケットの送信元IPは、インターフェースeth0のIPとして設定されます。
さて、パケットを172.17.0.1に送るために、Linuxは172.17.0.1のMACアドレスを調べなければなりません。MACアドレスは、IPアドレスとMACアドレスの変換を保存している内部ARPキャッシュを見て算出します。キャッシュが見つからない場合、Linuxは内部ネットワーク内でARPリクエストをブロードキャストし、誰が172.17.0.1を持っているかを尋ねます。IPの所有者は、カーネルにキャッシュされているARP応答を送信し、カーネルは送信元のMACアドレスをeth0のMACアドレスに、送信先のMACアドレスを先ほど得た172.17.0.1に設定して、パケットをゲートウェイに送信します。パケットが実際のサーバーに到達するまで、各ホップで同様のルーティングルックアッププロセスが行われます。トランスポート層とその上の層は、エンドサーバーでのみ機能します。中間ホップでは、IP/ネットワーク層までしか関与しません。

![説明のためのスクリーンショット](images/arp.gif)

0.0.0.0というゲートウェイがありますが、これはレイヤー3（ネットワーク層）のホップを必要としないことを意味しています。送信元と送信先の両方が同じネットワーク内にあります。カーネルは、送信先のMACアドレスを把握し、送信元と送信先のMACアドレスを適切に設定して、レイヤー3のホップを介さずに送信先に届くようにパケットを送信しなければなりません。

他のモジュールに続いて、SREのユースケースでこのセッションを締めくくりましょう。

## SREの役割における応用
1. 一般的にルーティングテーブルはDHCPによって作成されますが、これを弄るのは良くありません。ルーティングテーブルを弄らなければならない理由があるかもしれませんが、どうしても必要な場合にのみその方法をとります。
2. 「No route to host」というエラーは、宛先ホストのマックアドレスが見つからないことや、宛先ホストがダウンしていることを意味します。
3. まれに、ARPテーブルを見ることで、同じIPが誤って2つのホストに割り当てられているIPコンフリクトが発生し、それが予期せぬ動作を引き起こしているかどうかを理解することができます。
