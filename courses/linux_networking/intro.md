# Linuxネットワークの基礎知識

## 前提条件

- DNS、TCP、UDP、HTTPなど、TCP/IPスタックでよく使われる専門用語に関する高度な知識
- [Linuxコマンドラインの基礎](linux_basics/command_line_basics/)

## このコースで扱うこと

本コースでは、SREがどのようにシステムを最適化してWebスタックのパフォーマンスを向上させるか、またネットワークスタックのいずれかの層に問題がある場合のトラブルシューティングについて説明します。このコースでは、従来のTCP/IPスタックの各層を掘り下げ、SREがインターネットの機能を俯瞰して理解することを期待しています。

## このコースでは扱わないこと

このコースでは、基本的なことに時間を費やします。[HTTP/2.0](https://ja.wikipedia.org/wiki/HTTP/2)、[QUIC](https://ja.wikipedia.org/wiki/QUIC)、[TCP輻輳制御プロトコル](https://en.wikipedia.org/wiki/TCP_congestion_control)、[エニーキャスト](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%8B%E3%83%BC%E3%82%AD%E3%83%A3%E3%82%B9%E3%83%88)、[BGP](https://ja.wikipedia.org/wiki/Border_Gateway_Protocol)、[CDN](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84%E3%83%87%E3%83%AA%E3%83%90%E3%83%AA%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF)、[VPC](https://ja.wikipedia.org/wiki/Virtual_Private_Network)、[マルチキャスト](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%AB%E3%83%81%E3%82%AD%E3%83%A3%E3%82%B9%E3%83%88)などの概念はカバーしていません。このコースでは、このような概念を理解するための基礎知識を提供します。

## コースの全体像

このコースでは、「linkedin.comをブラウザで開くと何が起こるか？」という質問を取り上げます。具体的には、アプリケーション層のプロトコルであるDNSとHTTP、トランスポート層のプロトコルであるUDPとTCP、ネットワーキング層のプロトコルであるIPとデータリンク層のプロトコルを取り上げます。

# # コース内容
1. [DNS](linux_networking/dns/)
2. [UDP](linux_networking/udp/)
3. [HTTP](linux_networking/http/)
4. [TCP](linux_networking/tcp/)
5. [IPルーティング](linux_networking/ipr/)
