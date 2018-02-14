---
title: "APIをデータソースに接続する"
lang: ja
layout: page
toc: false
keywords: LoopBack
tags: [getting_started]
sidebar: ja_lb3_sidebar
permalink: /doc/ja/lb3/Connect-your-API-to-a-data-source.html
summary: LoopBackでは、データモデルを様々なデータソースに永続化することが、コードを書くことなしに簡単に行なえます。
---

{% include content/ja/gs-prereqs.html %}

前のセクションから持ってきたアプリケーションを、MySQLに接続します。

{% include note.html content="
チュートリアルのここまでのステップを済ませているなら、[データソースの追加](#add-a-data-source)に進みます。

直接ここに来た場合、以下のステップを実行して追いついてください。
" %}

（前の項目が終わった状態の）アプリケーションをGitHubから取得し、依存関係をインストールします。

```
$ git clone https://github.com/strongloop/loopback-getting-started.git
$ cd loopback-getting-started
$ git checkout step1
$ npm install
```

## データソースの追加

それでは、[データソース生成ツール](Data-source-generator)を使って、データソースを定義していきます。

```
$ lb datasource
```

生成ツールは、データソースの名前を尋ねます。

```
? データ・ソース名を入力してください:
```

**mysqlDs** と入力して、**Enter** を押下します。

次に、データソースの種類を聞かれます。

```
? mysqlDs のコネクターを選択します: (Use arrow keys)
> In-memory db (StrongLoop でサポートされます)
  In-memory key-value connector (StrongLoop でサポートされます)
  IBM Object Storage (StrongLoop でサポートされます)
  IBM DB2 (StrongLoop でサポートされます)
  IBM DashDB (StrongLoop でサポートされます)
  IBM MQ Light (StrongLoop でサポートされます)
  IBM Cloudant DB (StrongLoop でサポートされます)
(Move up and down to reveal more choices)
```

下矢印キーを押下して、**MySQL (StrongLoop でサポートされます)** を選び、**Enter** を押下します。

すると、データソースの接続設定を尋ねてきます。
MySQLの場合、すべての設定をURL形式で入力することも、個別に入力することもできます。

```
Connector-specific configuration:
? Connection String url to override other settings (eg: mysql://user:pass@host/db):
```
**Enter** を押下して、URL接続文字列をスキップし、個別に設定を入力します。

{% include important.html content="もし、利用可能なMySQLデータベースサーバを持っている場合、そちらを使ってください。\"getting_started\" という名前で新しいデータベースを作ります。別の名前が良ければ違う名前でも構いません。`datasources.json` の `mysqlDs.database` プロパティとそれが一致するようにしてください（下参照）。

もしそうでない場合、`demo.strongloop.com` で稼働している StrongLoop MySQL サーバを利用できます。しかし、これは共有のリソースであることに注意してください。二人のユーザが同時にサンプルデータを作成するスクリプトを実行して、競合状態に陥る可能性が、少ないですがあります。この理由から、もしMySQLを持っていれば、そちらを使うようにお勧めしています。
" %}

StrongLoop MySQL サーバを使う場合、以下のように設定を入力します。
自分のMySQLサーバを使う場合、あなたのサーバに合ったホスト名・ポート番号・ログイン資格情報を入力します。

```
? host: demo.strongloop.com
? port: 3306
? user: demo
? password: L00pBack
? database: getting_started
? Install loopback-connector-mysql@^2.2 Yes
```

コネクタのインストールを聞かれた時、**Enter** を押下して、生成ツールに `npm install loopback-connector-mysql --save` を実行させます。
また、`server/datasources.json` ファイルに、以下のようなデータソース定義が追加されます。「mysqlDs」が追加したデータソースで、「db」というインメモリデータソースも、同様に既定で存在しています。

{% include code-caption.html content="/server/datasources.json" %}
```javascript
{
  "db": {
    "name": "db",
    "connector": "memory"
  },
  "mysqlDs": {
      "host": "demo.strongloop.com",
      "port": 3306,
      "url": "",
      "database": "getting_started",
      "password": "L00pBack",
      "name": "mysqlDs",
      "user": "demo",
      "connector": "mysql"
    }
}
```

## CoffeeShop モデルを MySQL に接続する

MySQLデータソースを作成し終わり、CoffeeShopモデルがあります。あとはこれらを接続するだけです。
LoopBackアプリケーションは、モデルとデータソースを紐付けるのに、[model-config.json](model-config.json) ファイルを使います。
`/server/model-config.json` を開き、CoffeeShop エントリを探します。

{% include code-caption.html content="/server/model-config.json" %}
```javascript
...
  "CoffeeShop": {
    "dataSource": "db",
    "public": true
  }
  ...
```

`dataSource` プロパティを `db` から `mysqlDs` に変更します。これで、CoffeeShopモデルが、作ったばかりのMySQLデータソースに紐付けられます。

{% include code-caption.html content="/server/model-config.json" %}
```javascript
...
  "CoffeeShop": {
    "dataSource": "mysqlDs",
    "public": true
  }
  ...
```

## テストデータの追加と表示

CoffeeShop モデルが 既にLoopBackにあります。どうやってMySQLデータベースに対応するテーブルを作成しますか？

SQL文を直接実行しようとすることもできますが、LoopBackは、_auto-migration_ と呼ばれる、それを自動的に行うNode APIを提供しています。
詳細は、[モデルからデータベーススキーマを作る](Creating-a-database-schema-from-models) を参照してください。

`loopback-getting-started` モジュールは、auto-migration のデモを行う `create-sample-models.js` スクリプトを含んでいます。
もし、初めから進めてきた（そして、このモジュールを複製していない）場合、下か [GitHub から](https://github.com/strongloop/loopback-getting-started/blob/master/server/boot/create-sample-models.js) コピーする必要があります。
アプリケーションの `/server/boot` ディレクトリにスクリプトを配置すると、アプリケーションの開始時に実行されます。

{% include note.html content="以下の auto-migration スクリプトは、LoopBackがアプリケーションを起動する時に最初に実行する _起動スクリプト_ の例です。アプリケーションが開始時に行う必要がある初期化やその他のロジックのために、起動スクリプトを使います。詳しくは[起動スクリプトの定義](Defining-boot-scripts) を参照してください。
" %}

{% include code-caption.html content="/server/boot/create-sample-models.js" %}
```javascript
module.exports = function(app) {
  app.dataSources.mysqlDs.automigrate('CoffeeShop', function(err) {
    if (err) throw err;

    app.models.CoffeeShop.create([{
      name: 'Bel Cafe',
      city: 'Vancouver'
    }, {
      name: 'Three Bees Coffee House',
      city: 'San Mateo'
    }, {
      name: 'Caffe Artigiano',
      city: 'Vancouver'
    }], function(err, coffeeShops) {
      if (err) throw err;

      console.log('Models created: \n', coffeeShops);
    });
  });
};
```

これは、データソースにテストデータを保存します。

{% include note.html content="auto-migrationコマンドを含む起動スクリプトは、アプリケーションを起動するたび、_毎回_ 実行されます。[`automigrate()`](http://apidocs.loopback.io/loopback-datasource-juggler/#datasource-prototype-automigrate) は、テーブルを重複して作成しないように、最初にテーブルを削除します。
" %}

アプリケーションを実行します。

```
$ node .
```

コンソールには、以下のように表示されます。

```
...
Browse your REST API at http://0.0.0.0:3000/explorer
Web server listening at: http://0.0.0.0:3000/
Models created: [ { name: 'Bel Cafe',
    city: 'Vancouver',
    id: 1 },
  { name: 'Three Bees Coffee House',
    city: 'San Mateo',
    id: 3 },
  { name: 'Caffe Artigiano',
    city: 'Vancouver',
    id: 2 } ]
```

API Explorer も使えます。

1.  [http://0.0.0.0:3000/explorer/](http://0.0.0.0:3000/explorer/) (OSやブラウザによっては、[http://localhost:3000/explorer,](http://localhost:3000/explorer) を使う必要があります) を開きます。
2.  **GET  /CoffeeShops  Find all instance of the model matched by filter...** をクリックします。
3.  **Try it out!** をクリックします。
4.  上のスクリプトで作成された３つのコーヒーショップのデータが表示されます。

{% include next.html content= "
[API を拡張する](Extend-your-API.html)では、モデルに独自のメソッドを追加する方法を学習します。
" %}
