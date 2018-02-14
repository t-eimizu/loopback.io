---
title: "APIを拡張する"
lang: ja
layout: page
toc: false
keywords: LoopBack
tags: [getting_started]
sidebar: ja_lb3_sidebar
permalink: /doc/ja/lb3/Extend-your-API.html
summary: In LoopBack, a Node function attached to a custom REST endpoint is called a <i>remote method</i>.
summary: LoopBackでは、Node function attached to a custom REST endpoint is called a <i>remote method</i>.
---

{% include content/ja/gs-prereqs.html %}

このセクションでは、APIに独自のリモートメソッドを追加していきます。

{% include note.html content="
チュートリアルのここまでのステップを済ませているなら、[リモートメソッドの追加](#add-a-remote-method)に進みます。

直接ここに来た場合、以下のステップを実行して追いついてください。
" %}

（前の項目が終わった状態の）アプリケーションをGitHubから取得し、依存関係をインストールします。

```
$ git clone https://github.com/strongloop/loopback-getting-started.git
$ cd loopback-getting-started
$ git checkout step2
$ npm install
```

## リモートメソッドの追加

以下の手順に従います。

1.  アプリケーションの `/common/models` ディレクトリを開きます。`coffee-shop.js` と `coffee-shop.json` ファイルがあることがわかります。

    {% include important.html content="
    LoopBack の [モデル生成ツール](Model-generator.html) は、モデル名をキャメルケース（MyModel）から、小文字をハイフンで繋いだ名前（my-model）に自動的に変換します。例えば、\"FooBar\" という名前のモデルをモデル生成ツールで作った場合、`common/models` に `foo-bar.json` と `foo-bar.js` というファイルが作られます。しかし、モデル名（\"FooBar\"）はモデルの name プロパティに保持されています。
    " %}
2.  `coffee-shop.js` を好きなエディタで開きます。既定では、空の関数が存在しています。

    ```js
    module.exports = function(CoffeeShop) {};
    ```

3.  以下のコードを関数に追加して、モデルの振る舞いにリモートメソッドを追加します。

    ```js
    module.exports = function(CoffeeShop) {
      CoffeeShop.status = function(cb) {
        var currentDate = new Date();
        var currentHour = currentDate.getHours();
        var OPEN_HOUR = 6;
        var CLOSE_HOUR = 20;
        console.log('今は %d 時です。', currentHour);
        var response;
        if (currentHour >= OPEN_HOUR && currentHour < CLOSE_HOUR) {
          response = '営業中です。';
        } else {
          response = 'すみません。営業時間外です。毎日6:00～20:00に営業しています。';
        }
        cb(null, response);
      };
      CoffeeShop.remoteMethod(
        'status', {
          http: {
            path: '/status',
            verb: 'get'
          },
          returns: {
            arg: 'status',
            type: 'string'
          }
        }
      );
    };
    ```

    これは「status」というシンプルなリモートメソッドを定義しています。引数はなく、現在時刻によって「営業中です」または「すみません。営業時間外です」というJSONのメッセージを返すものです。

    もちろん、練習としてリモートメソッドでもっと面白くて複雑なこともできます。データベースに保存する前に入力データを操作するなどです。リモートメソッドのルートを変更したり、複雑な引数や戻り値を定義することもできます。詳細は[リモートメソッド](Remote-methods)を参照してください。

4.  ファイルを保存します。

{% include note.html content="
リモートメソッドを全員に公開したくない場合、アクセス制御リスト（ACL）を使ってアクセスを制限するのが簡単です。[リモートメソッドへのACL追加](Remote-methods#adding-acls-to-remote-methods) を参照してください。
" %}

## リモートメソッドを試す

1.  アプリケーションの最上位ディレクトリに戻り、アプリケーションを実行します。

    `$ node .`

2.  [http://localhost:3000/explorer](http://localhost:3000/explorer) へ行き、API Explorer を表示します。そして、CoffeeShopsをクリックして、新しいREST エンドポイント `GET /CoffeeShop/status` を確認します。
    {% include image.html file="5570643.png" alt="" %}

3.  **Try it out!** をクリックします。
    リモートメソッドを呼出した結果が表示されるでしょう。
    ```js
    {
      "status": "営業中です。"
    }
    ```

LoopBack にリモートメソッドを追加するのは、なんて簡単なんでしょう！

詳細は、 [Remote methods](Remote-methods) を参照してください。

## リモートメソッドで作成・読取・更新・削除のメソッドを実行する

`status` リモートメソッドは平凡でしたが、リモートメソッドは、標準的なモデルの作成・読取・更新・削除メソッドを使って、データ処理や検証を行うこともできます。
これはシンプルな例です（これは、`loopback-getting-started` リポジトリにはありません）。

```js
module.exports = function(CoffeeShop) {
...
  CoffeeShop.getName = function(shopId, cb) {
    CoffeeShop.findById( shopId, function (err, instance) {
        var response = "Name of coffee shop is " + instance.name;
        cb(null, response);
        console.log(response);
    });
  }
...
  CoffeeShop.remoteMethod (
        'getName',
        {
          http: {path: '/getname', verb: 'get'},
          accepts: {arg: 'id', type: 'number', http: { source: 'query' } },
          returns: {arg: 'name', type: 'string'}
        }
    );
}
```

リモートメソッドにアクセスするには、以下のようにします。

`http://0.0.0.0:3000/api/CoffeeShops/getname?id=1`

以下のような応答が得られるでしょう。

```js
{
  "name": "Name of coffee shop is Bel Cafe"
}
```

{% include next.html content="[静的webページの追加](Add-a-static-web-page.html)では、静的なクライアント用の資源（HTML/CSS, images, JavaScriptなど）を提供するためにExpressのミドルウェアを追加します。"
%}
