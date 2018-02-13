---
title: "単純なAPIの作成"
lang: ja
layout: page
toc: false
keywords: LoopBack
tags: [getting_started]
sidebar: ja_lb3_sidebar
permalink: /doc/ja/lb3/Create-a-simple-API.html
summary: アプリケーション生成ツールを使い、LoopBackアプリケーション・モデル・データソースを素早く作成する。
---

{% include content/ja/gs-prereqs.html %}

## 新しいアプリケーションを作成する

新しいアプリケーションを作成するには、LoopBack [アプリケーション生成ツール](Application-generator)を実行します。

`loopback-cli` を使用している場合

```
$ lb
```

`apic`を使用している場合

```
$ apic loopback
```

`slc`を使用している場合

```
$ slc loopback
```

LoopBackアプリケーション生成ツールは、フレンドリーなアスキーアートがあなたに挨拶し、アプリケーションの名前を尋ねてきます。
（`apic` や `slc` は少し異なる表示になりますが、アプリケーションの名前を要求します）

`loopback-getting-started` と入力します。
すると、生成ツールはプロジェクトを含むディレクトリの名前を尋ねます。Enterを押して既定値（アプリケーション名と同じ）を受け入れてください。

```
     _-----_
    |       |    .--------------------------.
    |--(o)--|    |  Let's create a LoopBack |
   `---------´   |       application!       |
    ( _´U`_ )    '--------------------------'
    /___A___\
     |     |
   __'.___.'__
 ´   `  |° ´ Y `
? アプリケーションの名前は何ですか? loopback-getting-started
? プロジェクトを格納するディレクトリーの名前を入力してください: loopback-getting-started
```

{% include note.html content="アプリケーションに別の名前を使うこともできます。そうする場合、チュートリアルの残りの部分を通して、\"loopback-getting-started\" を別の名前に置き換えてください。
" %}

するとツールは、作成するアプリケーションの種類を尋ねます。

`lb` または `apic` の場合、

```
? どのようなタイプのアプリケーションにしますか? (Use arrow keys)
❯ empty-server (An empty LoopBack API, without any configured models or datasources)
  hello-world (A project containing a controller, including a single vanilla Message and
    a single remote method)
  notes (A project containing a basic working example, including a memory database)
```

**Enter** を押下して、既定の選択である `empty server` （`lb` の場合は `api-server`）を受け入れます。

If using `slc`:

```
? どのようなタイプのアプリケーションにしますか? (Use arrow keys)
  api-server (A LoopBack API server with local User auth)
  empty-server (An empty LoopBack API, without any configured models or datasources)
❯ hello-world (A project containing a controller, including a single vanilla Message and
    a single remote method)
  notes (A project containing a basic working example, including a memory database)
```

矢印キーで `hello-world` を選択します。

生成ツールは、アプリケーションの土台を作りながら、以下のようなメッセージを表示します。

1.  [プロジェクトフォルダ構造](Project-layout-reference) の初期化。
2.  既定のJSONファイルの作成。
3.  既定のJavaScriptファイルの作成。
4.  依存するNodeモジュールのダウンロードとインストール（手作業で`npm install`をしたのと同様）。

## モデルを作成する

さて、最初のプロジェクトの土台ができあがりました。自動的にREST API エンドポイントが生成される _CoffeeShop_ モデルを作りましょう。

新しいアプリケーションのディレクトリに移動し、LoopBack [モデル生成ツール](Model-generator)を実行します。

```
$ cd loopback-getting-started
```

Then, using IBM API Connect developer toolkit:
```
$ apic create --type model
```

StrongLoop ツールを使用している場合、
```
$ lb model
```

生成ツールがモデル名を尋ねてくるので、**CoffeeShop** と入力します 。

```
? モデル名を入力します: CoffeeShop
```

すると、モデルを既に定義したデータソースに紐付けるかどうかを聞いてきます。

この時点では、既定のインメモリデータソースのみが利用可能です。**Enter** を押下して 選択します。

```
...
[?] CoffeeShop を付加するデータ・ソースを選択します: (Use arrow keys)
❯ db (memory)
```

次に生成ツールは、モデルに使用する基本クラスを尋ねます。ゆくゆくは、データベースにある永続的なデータソースにこのモデルを接続しますので、下矢印を押して **PersistedModel** を選んで、**Enter** を押下してください。

```
? モデルの基本クラスを選択します (Use arrow keys)
  Model
❯ PersistedModel
  ACL
  AccessToken
  Application
  Change
  Checkpoint
```

[PersistedModel ](http://apidocs.loopback.io/loopback/#persistedmodel)は、データベースのような永続的データソースに接続する全てのモデルの基本となるオブジェクトです。
モデルの継承階層についての概要は[LoopBack の核となる概念](LoopBack-core-concepts)を参照してください。

LoopBackの強力な優位性の一つが、モデルについて自動的に生成される REST API です。
生成ツールは、REST APIを公開するかどうか質問します。

再度 **Enter** を押下し、既定値を受け入れて、CoffeeShopモデルをREST経由で公開します。

```
? REST API を介して CoffeeShop を公開しますか? Y
```

LoopBackは自動的に、モデル名の _複数形_ を使って、モデルと紐付いたRESTのルートを作成します。既定では、名前を複数形に（"s"を追加）しますが、望むなら独自の複数形を指定することもできます。詳細は、[REST経由でモデルを公開する](Exposing-models-over-REST) を参照してください。

**Enter** を押下して、既定の複数形（CoffeeShops）を受け入れます。

```
? カスタム複数形 (REST URL の作成に使用します):
```

次に、モデルをサーバ限定に作成するか、サーバと[クライアントサイド LoopBack API](LoopBack-in-the-client) の両方で使用できる `/common` ディレクトリに作成するか質問されます。アプリケーションがサーバサイドのモデルしか使用しないとしても、既定の「共通」を保持しておきましょう。

```
? 共通モデルですか、あるいはサーバー専用ですか?
❯ 共通
  サーバー
```

どんなモデルにもプロパティがあります。それでは、CoffeeShop モデルに name というプロパティを定義しましょう。

プロパティの型として、**`string`** を選びます。（stringは既定の選択しなので、**Enter** を押下します）

```
では、CoffeeShop プロパティーをいくつか追加しましょう。
完了したら、空のプロパティー名を入力してください。
? プロパティー名: name
   invoke   loopback:property
? プロパティー・タイプ: (Use arrow keys)
❯ string
  number
  boolean
  object
  array
  date
  buffer
  geopoint
  (other)
```

それぞれのプロパティは、任意または必須にできます。**`y`** を入力して `name` を必須にします。

`? 必須 (y/N)`

プロパティの既定値を入力するように求められます。Enterを押下して、既定値がない状態にします。

`? デフォルト値 [なしの場合は空白のまま]: `

別のプロパティを追加するか質問されます。以下のように、"city" という必須のプロパティを追加します。

```
別の CoffeeShop プロパティーを追加しましょう。
? プロパティー名: city
? プロパティー・タイプ: string
? 必須 Yes
? デフォルト値 [なしの場合は空白のまま]:
```

モデル作成処理を終えるには、次のプロパティの名前を聞かれている時に **Enter** を押下します。

モデル生成ツールは、アプリケーションの `common/models` ディレクトリに、モデルを定義する`coffee-shop.json` と `coffee-shop.js` という２つのファイルを作成します。

{% include note.html content="LoopBack の [モデル生成ツール](Model-generator.html) は、モデル名をキャメルケース（MyModel）から、小文字をハイフンで繋いだ名前（my-model）に自動的に変換します。例えば、\"FooBar\" という名前のモデルをモデル生成ツールで作った場合、`common/models` に `foo-bar.json` と `foo-bar.js` というファイルが作られます。しかし、モデル名（\"FooBar\"）はモデルの name プロパティに保持されています。
" %}

## プロジェクトの構造を確認する

標準的なLoopBack アプリケーションの構造についての詳細は、[プロジェクト構成のリファレンス](Project-layout-reference) を参照してください。

## アプリケーションを実行する

アプリケーションを開始します。

```
$ node .
...
Web server listening at: http://0.0.0.0:3000
Browse your REST API at http://0.0.0.0:3000/explorer
```

{% include note.html content="`node` コマンドでアプリケーションを実行するのは、ローカルマシンで開発する時に適した方法です。本番環境では、スケーラビリティや信頼性のために、 [API Connect](https://developer.ibm.com/apiconnect/) や [process manager](http://strong-pm.io/) を使うことを検討してください。
" %}

ブラウザで [http://0.0.0.0:3000/](http://0.0.0.0:3000/)（システムによっては、代わりに [http://localhost:3000](http://localhost:3000/) を使う必要があるかもしれません）を開くと、以下のようなステータス情報を含む、既定のアプリケーションからの応答が表示されるでしょう。

```
{"started":"2016-09-10T21:59:47.155Z","uptime":42.054}
```

ブラウザで [http://0.0.0.0:3000/explorer](http://0.0.0.0:3000/explorer) または [http://localhost:3000/explorer](http://localhost:3000/explorer) を開いてください。StrongLoop API Explorerが表示されます。

{% include image.html file="5570638.png" alt="" %}

LoopBackを使った単純なステップを通して、CoffeeShop モデルを作成し、そのプロパティを指定し、REST経由でそれらを公開しまいた。

{% include next.html content= "
[API Explorerを使う](Use-API-Explorer.html) では、今作成したREST API をより深く探索し、その操作について学習します。
" %}
