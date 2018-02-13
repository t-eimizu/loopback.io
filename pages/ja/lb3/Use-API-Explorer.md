---
title: "API Explorer を使う"
lang: ja
layout: page
toc: false
keywords: LoopBack
tags: [getting_started]
sidebar: ja_lb3_sidebar
permalink: /doc/ja/lb3/Use-API-Explorer.html
summary: LoopBack アプリケーションは、開発中にREST API をテストするために使える、組み込みの API Explorer を同梱しています。
---

{% include content/ja/gs-prereqs.html %}

作成したAPIを使うのはあなただけではありません。ですから、APIをあなたが文書化する必要があります。幸運なことに、LoopBackは API Explorerを提供しています。

{% include note.html content="[単純なAPIの作成](Create-a-simple-API.html) を済ませているなら、アプリケーションを実行したまま、[API Explorerの実行](#run-api-explorer) まで飛ばしてください。

直接ここに来た場合、以下のステップを実行して追いついてください。
" %}

（前の項目が終わった状態の）アプリケーションをGitHubから取得し、依存関係をインストールします。

```
$ git clone https://github.com/strongloop/loopback-getting-started.git
$ cd loopback-getting-started
$ git checkout step1
$ npm install
```

## API Explorerの実行

アプリケーションを実行します。

`$ node .`

そして、[http://localhost:3000/explorer](http://localhost:3000/explorer) に行きます。StrongLoop API Explorer がアプリケーションの持つ２つのモデル **User** と **CoffeeShop** を表示するでしょう。

{% include image.html file="5570639.png" alt="" %}

## LoopBackの組み込みモデルについて

自分で定義した CoffeeShop モデルに加えて、LoopBackは 既定で全てのアプリケーションに User モデルと、そのエンドポイントを生成します。

LoopBack は、一般的なユースケースのために、その他にも幾つかのモデルを作成します。
詳細は、[組み込みモデル](Using-built-in-models) を参照してください。

## CoffeeShop モデルを探索する

それでは、 CoffeeShop モデルを「掘り下げ」て行きましょう。**CoffeeShop** をクリックして、全てのAPIエンドポイントを表示します。

{% include image.html file="5570640.png" alt="" %}

APIエンドポイントの行を下まで見ていくと、一般的な作成・読取・更新・削除（CRUD）操作が全て網羅され、その他にも幾つか操作があることが分かるでしょう。

**POST /CoffeeShops **   **Create a new instance of the model and persist it into the data source** をクリックして、操作を展開します。
{% include image.html file="5570641.png" alt="" %}

上の図のガイドに従ってください。

「Example Value」をクリックすると、JSONの「データテンプレート」が **data** 項目にセットされ、編集することができます。

`name` プロパティと `city` プロパティに何か文字を入力して、`id` プロパティには何も入力しないでください。LoopBackは、モデルの各インスタンスが常に一意のIDを持つように、自動的に管理します。

```js
{
  "name": "Verve Coffee",
  "city": "Santa Cruz"
}
```

そうしたら、**Try it out!** ボタンをクリックします。

REST リクエストが送信されて、アプリケーションの応答が表示されるでしょう（以下は例です）。

{% include image.html file="5570642.png" alt="" %}

**Response Body** 項目には、入力したデータが、実際にデータソースに追加された内容を確認するために表示されています。

今度は、**GET /CoffeeShops** をクリックして、エンドポイントを展開します。**Try it out!** をクリックして、CoffeeShop モデルに登録したデータを取得します。
POST APIで作成したレコードが表示されます。

気が向いたら、他のリクエストも試してみましょう。**filter** 項目を使って、
[Where フィルタ](Where-filter)・[Limit filter](Limit-filter)・その他のフィルタを追加して、より複雑な [検索](Querying-data)を行うこともできます。
詳細は [データの検索](Querying-data) を参照してください。

{% include tip.html content="
API Explorer は自動的に \"filter\" をクエリ文字列に追加しますが、**filter** 項目には
[文字列化したJSON](Querying-data.html#using-stringified-json-in-rest-queries)を入力しなければなりません。また、引用符が正しい( \" )ことを確認してください。
丸くなっていたり、装飾的な引用符( “ or ” )では正しく動きません。見た目でこれらを区別するのはしばしば困難です。
" %}

既に、API Explorer の右上にある **accessToken** 項目と **Set Access Token** ボタンにお気づきかもしれません。これらは、ユーザを認証し、
アプリケーションに「ログイン」できるようにして、認証が必要な操作を行えるようにするものです。
詳細は、[User モデル認証の導入](Introduction-to-User-model-authentication) を参照してください。

{% include next.html content="[APIをデータソースに接続する](Connect-your-API-to-a-data-source.html)では、データモデルを MongoDB のようなデータベースに永続化する方法を学びます。"
%}
