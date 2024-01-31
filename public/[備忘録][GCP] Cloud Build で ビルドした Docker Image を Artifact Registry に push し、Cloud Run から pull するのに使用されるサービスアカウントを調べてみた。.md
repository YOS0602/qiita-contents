---
title: >-
  [備忘録][GCP] Cloud Build で ビルドした Docker Image を Artifact Registry に push し、Cloud
  Run から pull するのに使用されるサービスアカウントを調べてみた。
tags:
  - googlecloud
private: false
updated_at: '2024-01-11T16:12:44+09:00'
id: a057aaf950987223e7cc
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

デフォルトでは、Artifact Registry へのアクセスには次のサービスアカウントが使用される。

## Cloud Build から Artifact Registry への push をする時

- Cloud Build サービス アカウント
    - `PROJECT_NUMBER@cloudbuild.gserviceaccount.com`

## Cloud Run から Artifact Registry の Image を pull する時

- Cloud Run サービス エージェント
    - `service-PROJECT_NUMBER@serverless-robot-prod.iam.gserviceaccount.com`

# 背景

- Container Registry が Deprecated となるため、Artifact Registry に移行が必要だった
- 移行手順の中で、Artifact Registryリポジトリを読み書きできる権限設定をする必要があった
  - 参照： [gcr.io ドメインをサポートするリポジトリを設定する  |  Artifact Registry のドキュメント  |  Google Cloud](https://cloud.google.com/artifact-registry/docs/transition/setup-gcr-repo?hl=ja#permissions)
- 移行を担当したプロジェクトおいて、どのサービスアカウントが使用され、どんな権限が必要となるか調査した
- その調査の備忘録として記事を作成する

なお、CICDは次のような構成である。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/fcd20e98-eaa3-696d-92ae-f0134f209d4e.png" width="600" >

# Cloud Build についての調査

https://cloud.google.com/build/docs/building/store-artifacts-in-artifact-registry?hl=ja

> 1. Cloud Build とリポジトリが異なるプロジェクトにある場合、または[ユーザー指定のサービス アカウント](https://cloud.google.com/build/docs/securing-builds/configure-user-specified-service-accounts?hl=ja)を使用してビルドを実行する場合は、[Artifact Registry 書き込みロールを付与する](https://cloud.google.com/artifact-registry/docs/access-control?hl=ja#grant)を使用して、リポジトリを持つプロジェクト内のビルドサービス アカウントに追加します。
>     
>     デフォルトの Cloud Build サービス アカウントには、同じ Google Cloud プロジェクト内のリポジトリに対して次の操作を行うためのアクセス権があります。
>     
>     - アーティファクトのアップロードとダウンロード
>     - Artifact Registry に [gcr.io リポジトリ](https://cloud.google.com/artifact-registry/docs/repositories?hl=ja#gcr)を作成します。

https://cloud.google.com/build/docs/securing-builds/configure-user-specified-service-accounts?hl=ja#running_builds_from_the_command_line

**コマンドラインからのビルドの実行** 見出しより

> 次のビルド構成ファイルでは、ユーザー指定のサービス アカウントを使用してビルドを実行するように Cloud Build を構成し、ビルドログをユーザーが作成した Cloud Storage バケットに保存するよう構成しています。
>
> ```yaml
> steps:
> - name: 'bash'
>   args: ['echo', 'Hello world!']
> logsBucket: 'LOGS_BUCKET_LOCATION'
> serviceAccount: 'projects/PROJECT_ID/serviceAccounts/SERVICE_ACCOUNT'
> options:
>   logging: GCS_ONLY
> ```

Cloud Build から Artifact Registry にpush する時は、`cloudbuild.yaml` で`serviceAccount` プロパティを明示的に指定していない限りは、デフォルトであるCloud Build サービス アカウント（`PROJECT_NUMBER@cloudbuild.gserviceaccount.com`）が使われる。

# Cloud Run についての調査

https://cloud.google.com/run/docs/deploying?hl=ja#service

**新しいサービスをデプロイする**　見出しより

> デプロイ時に、Cloud Run [サービス エージェント](https://cloud.google.com/iam/docs/service-agents?hl=ja)がデプロイされたコンテナにアクセスできる必要があります（デフォルトの場合）。

https://cloud.google.com/run/docs/deploying?hl=ja#other-projects

**他の Google Cloud プロジェクトからイメージをデプロイする** 見出しより

> 正しい IAM 権限を設定されている場合は、他の Google Cloud プロジェクトのコンテナ イメージをデプロイできます。
> 
> 1. Google Cloud コンソールで、Cloud Run サービスのプロジェクトを開きます。
> 2. [**Google 提供のロール付与を含む**] を選択します。
> 3. Cloud Run [サービス エージェント](https://cloud.google.com/iam/docs/service-agents?hl=ja)のメールアドレスをコピーします。このアドレスには、**@serverless-robot-prod.iam.gserviceaccount.com** という接尾辞が付いています。
> 4. 使用する Container Registry を所有するプロジェクトを開きます。
> 5. [**追加**] をクリックして、新しいプリンシパルを追加します。
> 6. [**新しいプリンシパル**] フィールドに、先ほどコピーしたサービス アカウントのメールアドレスを貼り付けます。
> 7. Container Registry を使用している場合は、[ロールを選択] プルダウン メニューで、**[ストレージ] -> [Storage オブジェクト閲覧者]** を選択します。Artifact Registry を使用している場合は、**[Artifact Registry] -> [Artifact Registry 読み取り]** のロールを選択します。
> 8. Cloud Run サービスを含むプロジェクトに[コンテナ イメージをデプロイ](https://cloud.google.com/run/docs/deploying?hl=ja#deploying_a_new_service)します。
>     
>     **注:** セキュリティを強化するには、[コンテナ イメージを含む Cloud Storage バケットまたは Artifact Registry リポジトリにのみアクセス権を付与](https://cloud.google.com/artifact-registry/docs/access-control?hl=ja#gcp)します。
>     

Cloud Run [サービス エージェント](https://cloud.google.com/iam/docs/service-agents?hl=ja) が使用され、Artifact Registry から image がpullされる。

# 参照

https://zenn.dev/cloud_ace/articles/6c401ce3b3bccc
