---
html: software-ecosystem.html
parent: introduction.html
blurb: どのようなXRP Ledgerソフトウェアがあり、どのように組み合わされているのか、その概要を知ることができます。
labels:
  - コアサーバ
---
# ソフトウェアエコシステム

XRP Ledgerは、価値のインターネットを実現するソフトウェアプロジェクトの、深く階層的なエコシステムの本拠地となっています。 XRP Ledgerと相互作用する全てのプロジェクト、ツール、ビジネスをリストアップすることは不可能なので、このページではいくつかのカテゴリーを挙げ、このウェブサイトで文書化されているいくつかの中心的なプロジェクトに焦点を当てます。
![XRPLエコシステム](img/ecosystem-apps-and-services.svg)

## スタックレベル

- [_コアサーバ_](#コアサーバ)は、常にトランザクションを中継し処理するピアツーピアのネットワークであるXRP Ledgerの基盤となるものです。

- [_クライアントライブラリ_](#クライアントライブラリ)は、プログラムコードに直接インポートされるハイレベルなソフトウェアで使用され、XRP Ledgerにアクセスするためのメソッドを含んでいます。

- [_ミドルウェア_](#ミドルウェア)は、XRP Ledgerのデータへの間接アクセスを可能にします。多くの場合、この層のアプリケーションには、独自のデータストレージと処理が存在します。

- [_アプリとサービス_](#アプリとサービス)は、XRP Ledgerでのユーザレベルのやり取りや、さらに上位のアプリやサービスに対する基盤を提供します。


### コアサーバ

XRP Ledgerの中心であるピアツーピアネットワークは、コンセンサスとトランザクションプロセスのルールを実行するために、信頼性が高く、効率のよいサーバを必要とします。XRP Ledger財団では、このサーバソフトウェアのリファレンス実装である[**`rippled`**](xrpl-servers.html)(発音は「リップルディー」)を公開しています。このサーバは、[一般利用が可能なオープンソースライセンス](https://github.com/XRPLF/rippled/blob/develop/LICENSE.md)の下で使用できるため、誰でもこのサーバの自身のインスタンスを検証し、変更することができます。また、いくつかの制限の下でそれを再公開することができます。

![コアサーバ](img/ecosystem-peer-to-peer.svg)

各コアサーバは、（[テストネットワーク](parallel-networks.html)に従うように構成されていない限り）同じネットワークに同期され、ネットワーク全体のあらゆる通信にアクセスできます。ネットワーク上の各サーバは、最近のトランザクションと、それらのトランザクションで行われた変更の記録とともに、XRP Ledger全体の最新の状態データの完全なコピーを保持します。また、各サーバは各トランザクションを単独で処理すると同時に、そのトランザクションの結果が残りのネットワークに一致するか検証します。サーバは、より多くの[レジャー履歴](ledger-history.html)を保持したり、[バリデータ](rippled-server-modes.html#バリデータ)としてコンセンサスプロセスに参加するように構成することができます。

コアサーバは、ユーザがデータを調べたり、サーバを管理したり、トランザクションを送信したりするために、[HTTP / WebSocket API](http-websocket-apis.html)を公開します。また、HTTP / WebSocket APIを提供するものの、ピアツーピアネットワークに直接接続せず、トランザクションを処理したりコンセンサスに参加したりしないサーバも存在します。Reportingモードで動作する`rippled`サーバやClioサーバなどのこれらのサーバは、トランザクションを処理するためにP2Pモードのコアサーバに依存しています。


### クライアントライブラリ

ライブラリは、通常HTTP / WebSocket APIを通じてXRP Ledgerにアクセスする際の共通作業の一部をシンプルにするものです。これらは、データを様々なプログラミング言語にとってより身近で便利な形に変換し、一般的な操作の実装を備えています。クライアントライブラリの中には、XRP Ledger財団によって公式にメンテナンスされているものもあれば、コミュニティの他のエンティティによってメンテナンスされているものもあります。

![クライアントライブラリ](img/ecosystem-client-libraries.svg)

ほとんどのクライアント・ライブラリのコアな機能の一つは、トランザクションをローカルで署名することであり、ユーザは秘密鍵をネットワーク上で送信する必要をなくします。

多くのミドルウェアサービスは、内部でクライアントライブラリを使用しています。

現在利用可能なクライアントライブラリについては、[クライアントライブラリ](client-libraries.html)を参照してください。


### ミドルウェア

ミドルウェアサービスは、一方ではXRP Ledger APIを利用し、もう一方では独自のAPIを提供するプログラムです。抽象化層を提供して、いくつかの一般的な機能をサービスとして提供することで上位のアプリケーションを容易に構築できるようにします。

![ミドルウェア](img/ecosystem-middleware.svg)

クライアントライブラリは、インポートしたプログラムとともに新たにインスタンス化され、シャットダウンされますが、ミドルウェアサービスは通常、無期限に稼働し続け、独自のデータベース（リレーショナルSQLデータベースなど）や設定ファイルを持つことがあります。クラウドサービスとして提供されているものもあり、価格や利用方法にさまざまな制限があります。


### アプリとサービス

最上層は、最もエキサイティングなことが起こる場所です。アプリとサービスは、XRP Ledgerに接続するための手段をユーザとデバイスに提供します。私設取引所、トークン発行者、マーケットプレイス、分散型取引所へのインターフェース、ウォレットなどのサービスは、XRPやあらゆる種類のトークンを含む様々な資産を売買・取引するためのユーザインターフェースを提供します。その他にも、さらに上位に重ねた追加サービスなど、多くの可能性が存在します。

![アプリとサービス](img/ecosystem-apps-and-services.svg)

このレイヤーで構築できるいくつかの例については、[ユースケース](use-cases.html)をご覧ください。

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}