---
title: '[備忘録] Azure Storage Account (Blob) のアクセス権について'
tags:
  - Azure
  - BlobStorage
  - StorageAccount
private: false
updated_at: '2023-07-06T14:03:16+09:00'
id: 3cd1272824aeef6cc7c7
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

AzureにおけるStorage Account(主にBlob Storage)のアクセスをどう制御するかについて、調査したことを備忘として残しておく。
検証に使用したリソースは削除済みであるため、画像に写り込んでいるURLやアクセスキーなどは無効となっている。
検証日: 2023/07/06

# 完全にPublicにしたい時

コンテナーを作成するときに、パブリックアクセスレベルを「BLOB」に設定する。（「コンテナー」でもOKだが、コンテナ内のBLOBを一覧表示できるようになる）

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/8a4d89b9-0278-98f5-7cde-bab840cc09d2.png" width="400">

BLOBパスを示したURLだけで、ファイルにアクセスできるようになる。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/86a5ee44-08e1-1e13-b4c4-2dbc2c3d54fe.png" width="650">

# Privateにしたい時

前提として、コンテナーは「プライベート」のアクセスレベルで作成する。じゃないとパブリックで誰でも閲覧できる状態になってしまう。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/386b5f36-0cdb-66d9-a5aa-59170e58165f.png" width="310">

## 手っ取り早くアクセスできるようにしたい時

アクセスキーを使用する。

デメリットとしては、何でもできてしまうこと。
ストレージアカウント内のBlob、File、Queue、Tableは全て読取も書込も可能で、削除することもできる。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/86651671-22ff-b6cb-72aa-7c74a8ef2f5a.png" width="800">

かなり危ないので、次に紹介するSASを使用するのがベター。

## 制限付きアクセスを付与する

>Shared Access Signature (SAS) を使用すると、ストレージ アカウント内のリソースへのセキュリティで保護された委任アクセスが可能になります。 SAS を使用すると、クライアントがデータにアクセスする方法をきめ細かく制御できます。 

[Shared Access Signatures (SAS) でデータの制限付きアクセスを付与する - Azure Storage | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-sas-overview) より引用

以下のことを制御できる。

- どのリソースにアクセスできるか
    - Blobであればコンテナー単位
- リソースに対し何の操作ができるか
    - 読取・書込・削除など
- SAS有効期間

[SAS を使用する際のベスト プラクティス](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-sas-overview#best-practices-when-using-sas) には目を通しておく。

SASには下記3種類あるが、サービス SASおよびアカウント SASは、ストレージアカウントキー(アクセスキー)に依存する。アクセスキーは漏洩したらアウトであり、頻繁に更新が推奨されているが、管理コストが増えてしまう。公式でも、warning noteに示すように、AzureAD認証を用いた「ユーザー委任 SAS」の使用を呼びかけている。

1. ユーザー委任 SAS
2. サービス SAS
3. アカウント SAS

:::note warn
>セキュリティのベスト プラクティスとして、より侵害されやすいアカウント キーを使用するのではなく、可能な限り Azure AD 資格情報を使用することをお勧めします。 アプリケーション設計で、BLOB ストレージへのアクセスのため、Shared Access Signature が必要な場合は、セキュリティを強化するために、可能な限り、Azure AD 資格情報を使用してユーザー委任 SAS を作成してください。 詳細については、「Azure Storage でデータへのアクセスを承認する」をご覧ください。

[公式ドキュメント](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-sas-overview#account-sas) より引用
:::

ストレージアカウントキー(アクセスキー)を使用しないなら、安全のためにも無効化しておくと良い。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/16e4b15d-1dfd-a614-dc72-9f1ae670ca9d.png" width="600">

### アカウントSASを利用する場合の手順メモ

推奨されてはいないが、使わざるを得ない時もありそうな気がするのでメモ

#### アクセスポリシーの作成

アクセスポリシーを先に作成し、アカウントSASに紐づけることで、有効期限やアクセス許可を柔軟に変更することができる。

アクセスポリシーを作成したいコンテナのメニューから、「アクセス ポリシー」を選択

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/7378c6ed-83fe-5166-db73-2c0d42501de1.png" width="600">

許可するアクセス種類や有効期限を設定する。
(実質)無期限に設定することもできそうだ。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/191c614b-0f56-4c3a-dda8-b5f557238bc9.png" width="490">

#### SASの発行

SASを作成したいコンテナのメニューから、「SAS の生成」を選択

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/f59e04c1-6247-2043-5ceb-24a8322006a5.png" width="550">

- アカウントSAS、ユーザー委任SASのどちらを作成するか
- アカウントキーはどちらを使うのか
- どのアクセスポリシーを使用するか

などを指定できる。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/71b332d7-8e98-8a92-488d-19275be03e18.png" width="520">

#### 動作確認

Public設定時は、下記のようなリソースパスを示したURLでBLOBを参照できていたが、ResourceNotFoundエラーが返却されている。
`https://qiitasample.blob.core.windows.net/private-container/dir1/hoge1.txt`

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/30f7b3c2-a825-12df-1f17-568fe667bc4c.png)

URLにSASトークンを付与してやると、BLOBを参照することができた。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/3108d3e3-0f56-15d8-f9a5-0c2bb00b4552.png)

## ネットワークトラフィックを制限する

[ストレージアカウント]-[セキュリティとネットワーク]-[ネットワーク]ブレード-[ファイアウォールと仮想ネットワーク]タブ

こちらより、特定のネットワークを介した通信に制限するよう設定することができる。
設定できるのは以下。

- 特定の仮想ネットワーク/サブネット
- 特定IPアドレス
    - 2023/07/06ではIPv4のみサポート
- アクセス許可を持つシステム割り当てマネージド ID

### プライベートエンドポイントを使用する

[ストレージアカウント]-[セキュリティとネットワーク]-[ネットワーク]ブレード-[プライベートエンドポイント接続]タブ

トラフィックを制限してプライベート向けに限定することで、リソースへのアクセス安全性を高めることができる。詳細は以下の公式ドキュメント参照。

[プライベート エンドポイントを使用する - Azure Storage | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-private-endpoints?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json)

なお、サブリソースごとに個別のプライベートエンドポイントを作成する必要がある。

- BLOB (blob、blob_secondary)
- Table (table、table_secondary)
- Queue (queue、queue_secondary)
- File (file、file_secondary)
- Web (web、web_secondary)
- Dfs (dfs、dfs_secondary)

# Refference

https://learn.microsoft.com/ja-jp/azure/storage/common/storage-sas-overview

https://learn.microsoft.com/ja-jp/azure/private-link/private-endpoint-overview
