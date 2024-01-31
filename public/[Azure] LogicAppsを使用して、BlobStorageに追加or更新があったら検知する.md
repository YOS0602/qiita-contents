---
title: '[Azure] LogicAppsを使用して、BlobStorageに追加or更新があったら検知する'
tags:
  - Azure
  - logicapps
  - BlobStorage
private: false
updated_at: '2023-02-06T10:16:09+09:00'
id: fac1dca8cd72e92af3cc
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

BlobStorageの追加or更新検知をしたい時、今まではEventGridを使ってきましたが、LogicAppsでもできるということを知りました。
試してみたことをメモしておきます。

# 前提

- 以下は作成済みとする
    - Azureサブスクリプション
    - リソースグループ
    - LogicApps (ロジックアプリ)
    - ストレージアカウント
- スクリーンショット、UIの項目名、手順など2023/02/02時点の情報を元に記述

# 手順

## ワークフローの作成

LogicAppsの[ワークフロー]ページにアクセスし、「追加」をクリック

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5a5b8430-4310-060e-d035-89590ca1fbaa.png" width=680>

ワークフロー名を入力し、状態の種類を選択します。
私の場合は信頼性の方が低遅延より重要な要件だったので、「ステートフル」を選択。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/73564210-66d4-0787-6fbf-44f1f8e20220.png" width=400>

## デザイナーの編集

作成したワークフローをクリックし、「デザイナー」タブを選択します。
トリガーとして、「Blob name」を選んでください。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/a0c6b839-d7b4-b2c5-cae0-0c13834b5ebb.png" width="800">

「When a blob is added or updated」を選択します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/2f1a325b-2b7d-da71-2a99-703706e6938b.png" width="630">

### ストレージアカウントの接続を作成

#### 接続名

接続名には、どのストレージアカウントに対して接続しているかが分かる名前にすると良いでしょう。

#### 認証タイプ

- Storage account connection string
- ActiveDirectoryOAuth
- ManagedServiceIdentity

の中から認証方法を選べます。ここでは一番簡単な「Storage account connection string」を使用します。(セキュリティ的には一番弱いと思うので気をつけて)

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/2692f6db-67b7-8f08-0ad8-45cd05f9f5f4.png" width="580">

### Blob pathの設定

追加・更新を検知したいBlobパスを設定します。
コンテナ配下全てを検知したい時は、コンテナ名だけ記述します。
特定ディレクトリ配下のみを監視するなら、~~`container/dir1/dir2`のように記述します。~~

:::note warn
2023/02/06追記
特定ディレクトリのみを監視するには`container/dir1/dir2/{blobname}.{blobextension}`とする必要がありました。
他にも、特定拡張子のみを監視する方法などが公式ドキュメントに記載されていますので、こちらを参照ください。
[Azure Logic Apps のワークフローから Azure Blob Storage に接続する - 組み込みコネクタ トリガー](https://learn.microsoft.com/ja-jp/azure/connectors/connectors-create-api-azureblobstorage?tabs=standard#built-in-connector-trigger)
:::

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/cbadbc8f-1193-6146-84f1-fa64b6c5117c.png" width="600">

#### セキュリティ設定

このあたりの設定をONにすると、実行履歴画面から実際に検知したパスなど、入力や出力が秘匿されるようです。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/274949a9-a170-a8f1-996b-15e49edef81b.png" width="550">

### 編集内容の保存

地味に忘れやすいので注意。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/950ae84d-d39f-8a80-0f32-7d5b383694d1.png" width="480">

Blob検知の設定手順は以上になります。

# 動作確認

## BlobStorageにファイルをアップロードしてみる

`{}`とだけ記述した`sample.json`をコンテナ直下にアップロードしてみました。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/072750f2-4d70-1d2e-2382-0d5da4ae95ad.png" width="600">

## LogicAppsの実行履歴を確認

ワークフローの「概要」ページを見ると、実行履歴に「成功」状態を示す行が新たに表示されています。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/f2e29600-74c1-2c85-9395-bc72d6acd354.png" width="1000">

中を覗いてみると、新たに追加・更新したBlobが検知できていることが分かります。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/91e3d04b-d3db-92fd-95ba-56a76b209593.png" width="900">

### ワークフロー内で変数として保持することもできる

ビルトインアクションの「変数」を使用することで、

- コンテナ名
- blob名

などを保持できます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/c0b5daba-fff2-4eb9-2dc4-d362303da6fc.png" width="820">

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e9a9e167-00de-db16-5551-386417718079.png" width="900">

### 「予期しないエラー. Failed to fetch」が表示されている時は

おそらくLogicAppsに設定されたNW制限が理由で、あなたのPCからのアクセスが拒否されており、詳細が確認できないと思われます。
NWルールを見直してみてください。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/abac538f-fb64-fdfd-3d4a-b29fe43f0438.png" width="280">

## ちなみにディレクトリ配下にアップロードした時は

`dir1/dir2/sample.json`のような文字列で検知されます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/dd265847-11ff-e773-ede2-8adf2e17dd07.png" width="700">

# StorageAccountにNW制限を設定している時は

私の場合はLogicAppsをVNET統合させ、ストレージアカウントで該当サブネットからの通信を許可しています。

![LogicAppsとストレージアカウントのNW設定.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/35c668f9-9302-129c-72a6-d7c78a431f1f.png)

# 2023/02/02時点で判明しているバグについて

## 特定形式のJSONをBlobStorageに格納しても検知してくれない

- 中身が空ファイル
- 配列のカッコはじまり`[]`

以上のファイルを拡張子`.json`でアップロードしても、LogicAppsは検知してくれませんでした。
試してないだけで他にもあるのかもしれません。

## サポートに問い合わせたら

- Microsoftの意図しないバグである
- Microsoft開発部にて修正対応中とのこと
- いつ修正がデプロイされるかは未定

# [余談] 検知して、どうするか

- slackに通知する
- webhookでAPIを呼び出す

などが考えられます。
AzureFunctionsを作成して呼び出すのもいいですし、特定のHTTPエンドポイントに向けてリクエストを投げることも、LogicAppsのビルトインアクションで行うことができます。
こちらはこの記事の対象としていないので、公式ドキュメントなどを参照してください。

# 参照

https://learn.microsoft.com/ja-jp/azure/connectors/connectors-create-api-azureblobstorage?tabs=standard
