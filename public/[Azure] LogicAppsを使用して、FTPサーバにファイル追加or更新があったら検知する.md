---
title: '[Azure] LogicAppsを使用して、FTPサーバにファイル追加or更新があったら検知する'
tags:
  - Azure
  - ftp
  - sftp
  - logicapps
private: false
updated_at: '2023-02-09T16:28:30+09:00'
id: cae759e5f82c9f31827d
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

ファイル連係を行うFTPサーバを定期的に監視し、ファイル追加や更新を検知したい要件がありました。
AzureのLogicAppsサービスを使用することで実現できたので、対応方法をメモしておきます。

もしかしたら自分がとった方法や設定値以外に、もっと効率の良いやり方があるかもしれませんが、とりあえずできたことをメモする、というのをターゲットにしています。

なお、今回の要件です。

- FTPサーバにはSFTPプロトコルで接続する
- ログインにはPW認証を用い、SSH認証は行わない

# 前提

- 以下は作成済みとする
    - Azureサブスクリプション
    - リソースグループ
    - LogicApps (ロジックアプリ)
    - 監視するFTPサーバ
- スクリーンショット、UIの項目名、手順など2023/02/09時点の情報を元に記述

# 手順

## ワークフローの作成

LogicAppsの[ワークフロー]ページにアクセスし、「追加」をクリック

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5a5b8430-4310-060e-d035-89590ca1fbaa.png" width=680>

ワークフロー名を入力し、状態の種類を選択します。
私の場合は信頼性の方が低遅延より重要な要件だったので、「ステートフル」を選択。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/d61aec48-2e24-4bd3-c011-0360b5df793c.png" width=400>

## デザイナーの編集

作成したワークフローをクリックし、「デザイナー」タブを選択します。
トリガーとして、「Azure」タブにある、「SFTP - SSH」を選んでください。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/b5e4e68a-f660-f867-373d-209e3def6c74.png" width="550">

:::note warn
以下のように、似たようなコネクタがたくさんあるので、間違えないように!
今回手順で使っているのは3ですが、僕は1でやろうとしてずっと上手くいかず悩んでました。

1. 「ビルトイン」-「SFTP」
    - [公式ドキュメント](https://learn.microsoft.com/ja-jp/azure/logic-apps/connectors/built-in/reference/sftp/)
2. 「Azure」-「SFTP」
    - こちらは **非推奨** となっていますので注意
    - [公式ドキュメント](https://learn.microsoft.com/ja-jp/connectors/sftp/)
3. 「Azure」-「SFTP - SSH」
    - [公式ドキュメント](https://learn.microsoft.com/ja-jp/connectors/sftpwithssh/)
:::

### トリガーの選択

「ファイルが追加または変更されたとき」を選択

:::note info
トリガーについての説明はこちらに載っています。
[ファイルが追加または修正されたとき](https://learn.microsoft.com/ja-jp/connectors/sftpwithssh/#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%8C%E8%BF%BD%E5%8A%A0%E3%81%BE%E3%81%9F%E3%81%AF%E4%BF%AE%E6%AD%A3%E3%81%95%E3%82%8C%E3%81%9F%E3%81%A8%E3%81%8D)

>この操作では、ファイルがフォルダー内で追加または変更されたときにフローをトリガーします。 トリガーは、ファイルのメタデータとファイルのコンテンツの両方を取得します。 トリガーは、ファイルの最終変更時刻に依存します。 サード パーティのクライアントによってファイルが作成されている場合は、クライアントで最終変更時刻の保存を無効にする必要があります。 50 MB 超えるファイルは、トリガーによってスキップされます。 サブフォルダーでファイルが追加および更新されても、トリガーは起動しません。 サブフォルダーをトリガーする必要がある場合は、複数のトリガーを作成する必要があります。

注意ポイントとしては下記でしょうか。

- ファイルの最終変更日時を見てトリガーする
- 50MBを超えるファイルはトリガーされない
- サブフォルダーは監視対象とならない
:::

### 接続の作成

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/74dfd16d-c003-8cb8-d233-dc5cd45904a0.png" width="680">

#### 接続名

接続名には、どのサーバに対して接続しているかが分かる名前にすると良いでしょう。

#### Host server address

接続するFTPサーバのIPアドレスまたはホストを記載します。
Publicアクセスになると思われるので、サーバはPublicIPを持つ必要があります。

:::note info
LogicAppsをVNet統合してたらPrivateIPでも接続できるんですかね？
検証はしてません。
:::

#### User name

サーバにログインするユーザ名を記載します。

#### Password

サーバにログインするパスワードを記載します。

#### SSH private key

今回はPW認証を使うので入力しません。

#### SSH private key passphrase

今回はPW認証を使うので入力しません。

#### Port number

ポート番号を記載します。

#### Disable SSH host key validation

今回はPW認証を使うので、チェックを入れます。

#### SSH host key finger-print

今回はPW認証を使うので入力しません。

#### Root folder path

どこをルートフォルダとして見るかのパスでしょうか？
今回は未入力にしました。

### パラメーター設定

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/a57f113d-bc4f-1c5d-427c-6b7f9b46a395.png" width="680">

#### フォルダー

監視するフォルダーパスを入力します。

#### ファイルのコンテンツを含める

ファイル内容に関してLogicAppsワークフロー内で何かしらの処理を行いたいなら「はい」にします。
今回の要件は、検知したあと別サービスをwebhookで呼ぶのが目的であり、ファイル内容を知る必要がないので「いいえ」としました。

#### コンテンツタイプの推測

拡張子に基づいてコンテンツ タイプを推測してくれます。

#### 項目を確認する頻度

ポーリング頻度を設定します。

### 編集内容の保存

地味に忘れやすいので注意。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/75a293fc-7cbd-064b-3c71-f41ad1b6afa2.png" width="300">

検知の設定手順は以上になります。

# 動作確認

実際にFTPサーバの該当ディレクトリにファイルを追加したところ、実行履歴より検知ができていることを確認できました。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/c69745f3-b54e-f434-7319-24a8371cd278.png" width="480">

# 参照

https://learn.microsoft.com/ja-jp/connectors/sftpwithssh/

https://learn.microsoft.com/ja-jp/connectors/sftp/

https://learn.microsoft.com/ja-jp/azure/logic-apps/connectors/built-in/reference/sftp/
