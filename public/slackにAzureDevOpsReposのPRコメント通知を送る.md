---
title: slackにAzureDevOpsReposのPRコメント通知を送る
tags:
  - Slack
  - AzureDevOps
private: false
updated_at: '2022-08-15T13:25:57+09:00'
id: e09fae9c27df1f9ee628
organization_url_name: null
slide: false
ignorePublish: false
---
# Appをslackワークスペースに追加

「Azure Repos」を追加する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/581e9524-a469-b35c-b437-1b9b713707a0.png)

# 通知を送りたいチャンネルにAppをinvite

`/invite @Azure Repos`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/7c39ddb6-58d1-368c-6dbf-f84c78b3fa83.png)

# DevOpsにサインインする

`/azrepos signin`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5e3b9270-168e-288d-88a4-3814aa4c0874.png)

「Sign in」ボタンをクリックして認証します。（ブラウザに遷移します）
6桁くらいのコードがブラウザに表示されるので、そちらをslack側で入力すると、下記のようにサインインが完了します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/57d5df3e-cd72-1693-ce80-665418933ab7.png)


# リポジトリをサブスクライブ

`/azrepos subscribe [リポジトリのURL]`

リポジトリのURLはこちらを参考に: `/azrepos subscribe https://dev.azure.com/myorg/myproject/_git/myrepo`
DevOpsのReposタブならどこ開いててもいけると思いますケド

# 通知するサブスクリプションの追加

`/azrepos subscriptions`

するとslack上でこんなメッセージが表示されますので、「Add subscription...」をクリック（動作はもっさりですのでのんびりした心でいきましょう）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e789ecfc-24ce-e93f-7984-294a6bb687ac.png)

下記をプルダウンから設定
Which event...?: `Pull request commented on`
Repository: `通知対象とするリポジトリ`(複数のリポジトリをサブスクライブしてると`Any`が選べる)
Target branch: `通知対象とするPRのマージ先ブランチの指定`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/cee454b9-4c6a-fcb8-5343-001bc2f5d20b.png)

「Save」をクリックして、下記メッセージが表示されてたら通知設定完了です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/bcb4357b-0b3a-0f27-629e-391bc44cf675.png)

# テストしてみようか

こんなコメントをしてみたら......
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/f3c7112d-591f-2d27-b3a2-f1bb44289abf.png)

slackに通知が来ましたね！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/ce14c454-75d5-7dd6-6174-9a2a98969dac.png)

# 備考

slackワークスペースへのアプリ追加は管理者じゃないとできなかったような気もする。
そのへんはググってくださいな。
