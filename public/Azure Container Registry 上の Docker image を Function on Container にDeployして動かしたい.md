---
title: >-
  Azure Container Registry 上の Docker image を Function on Container
  にDeployして動かしたい
tags:
  - Azure
  - Docker
  - AzureFunctions
  - AzureContainerRegistry
private: false
updated_at: '2023-06-08T17:32:29+09:00'
id: 200dee18991de8c04925
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

- 下記のAzureリソースは作成してあるものとします
    - ContainerをデプロイできるFunctionApp
    - Azure Container Registry
- Dockerイメージとしてビルドするアプリケーションおよび、Dockerfileはできているものとします

この記事の内容を執筆するにあたり、技術検証段階でAzureサポートのOさんに大きなご助力をいただきました。この場をお借りして感謝申し上げます💐

# パイプラインを作成

## yamlファイルを作成

AzurePipelinesで使用できる`yaml`ファイルのテンプレートや、TIPSが紹介されているMicrosoft公式のリポジトリがあります。ご存じなかった方はスターしておくと良いかもしれません。
こちらで公開されてるテンプレートの中にこんなものがありました。

https://github.com/microsoft/azure-pipelines-yaml/blob/master/templates/docker-container-functionapp.yml

>Build a Docker image, push it to an Azure Container Registry, and deploy it to an Azure Functions app.

まさにやりたいこととドンピシャ！
ありがたく参考にさせてもらいましょう🎉

## 参考にして作ったyaml

テンプレートにはAzureリソース作成も含まれてましたが、今回は作成済みの前提なので、以下のことだけに内容を変更します。

- Dockerイメージをビルド
- Azure Container Registryにイメージをpush
- FunctionAppにデプロイするイメージを設定

<details><summary>長いので折りたたんでます🎬</summary>

```functionapp-from-acr.yml
# Docker image, Azure Container Registry, and Azure Functions app
# Build a Docker image, push it to an Azure Container Registry, and deploy it to an Azure Functions app.
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

# Copy元: https://github.com/microsoft/azure-pipelines-yaml/blob/master/templates/docker-container-functionapp.yml

# A pipeline with no CI trigger
trigger: none

resources:
  - repo: self

variables:
  pool: 'ubuntu-latest'
  Azure.ServiceConnectionId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
  FunctionApp.Name: 'docker-func-sample'
  ACR.Name: 'sampleimage'
  ACR.ImageName: '$(ACR.Name):$(Build.BuildId)'
  ACR.FullName: 'acrqiitasample.azurecr.io'

jobs:
  - job: BuildAndPushImage
    displayName: Build And Push an Docker Image
    condition: succeeded()

    pool:
      vmImage: $(pool)

    steps:
      - task: Docker@1
        displayName: 'Build an image'
        inputs:
          azureSubscriptionEndpoint: '$(Azure.ServiceConnectionId)'
          azureContainerRegistry: '$(ACR.FullName)'
          imageName: '$(ACR.ImageName)'
          command: build
          dockerFile: '$(Build.SourcesDirectory)/Dockerfile'

      - task: Docker@1
        displayName: 'Push an image'
        inputs:
          azureSubscriptionEndpoint: '$(Azure.ServiceConnectionId)'
          azureContainerRegistry: '$(ACR.FullName)'
          imageName: '$(ACR.ImageName)'
          command: push

  - job: UpdateAppServiceConfiguration
    displayName: Update AppService Configuration to Deploy an Image
    dependsOn: BuildAndPushImage
    # See https://learn.microsoft.com/ja-jp/azure/devops/pipelines/process/expressions?view=azure-devops#job-to-job-dependencies-within-one-stage
    condition: or(succeeded(), eq(dependencies.BuildAndPushImage.result, 'Succeeded'))

    pool:
      vmImage: $(pool)

    steps:
      - task: AzureFunctionAppContainer@1
        displayName: 'Azure Function App on Container Deploy: $(FunctionApp.Name)'
        inputs:
          azureSubscription: '$(Azure.ServiceConnectionId)'
          appName: $(FunctionApp.Name)
          imageName: '$(ACR.FullName)/$(ACR.ImageName)'
```

</details>

### パイプラインの中でやってることをざっくり説明

- `BuildAndPushImage` Job
    - Dockerイメージをbuildする
    - Azure Container Registry にDockerイメージをpushする
- `UpdateAppServiceConfiguration` Job
    - FunctionAppに使用するDockerイメージを設定
        - `Trying to update App Service Configuration settings. Data: {"appCommandLine":null,"linuxFxVersion":"DOCKER|acrqiitasample.azurecr.io/sampleimage:1293"}`

パイプラインを動かした後に、FunctionのBashから環境変数を見ると、Dockerイメージの情報が登録されています。

```terminal
$ echo $LINUX_FX_VERSION
DOCKER|acrqiitasample.azurecr.io/sampleimage:1293
```

## Azure DevOpsのPipelineを作成する

このあたりを参考に、`functionapp-from-acr.yml`をPipelineとしてAzure DevOpsで実行できるようにします。たぶんGUI見ればできると思います👍🏻
[最初のパイプラインの作成 - Azure Pipelines | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=java%2Ctfs-2018-2%2Cbrowser)

# PipelineからAzureリソースへの接続に使用するサービスコネクション作成

下記の接続に使用されます。

- Pipeline→ACR
- Pipeline→FunctionApp

## AzureDevOpsでの作業

[ProjectSettings]-[Pipelines]-[Service connections]にて、「New service connection」をクリック

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/76a713af-b89b-984d-13dc-cee0d44bf2bc.png" width="600">

様々なAzureリソースに対し操作を行うことになるので、`Azure Resource Manager`を選択。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/27b24fdd-c622-c8fa-96d8-7c6b81c0ccab.png" width="400">

推奨設定の `Service principal (automatic)` を選択

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e1656819-a85b-e98a-b3e8-0c3074d44bd1.png" width="400">

ここで認証を求めるポップアップが出るので、ブロック設定になってると先に進めないので注意🧱
ARMで操作するスコープとなるサブスクリプションやリソースグループをプルダウンで選択し、サービスコネクション名を入力したら「Save」をクリック。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/26a029a9-e179-63a5-ed44-4d8ada086641.png" width="400">

サービスコネクションが作成されます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/d2159d9e-278b-f587-5741-9b54c8b82cbf.png" width="350">

AzureADの「アプリの登録」画面におけるアプリケーション一覧に、作成したサービスコネクションが表示されます。隠してるところだらけで分かりづらくて申し訳ない......

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/74dacab4-f30b-fcc2-0089-4dda4a118730.png" width="820">

## 作成したサービスコネクションの情報を、yamlパイプラインに反映

作成したサービスコネクション名をyamlの`variables`に記載します。

```diff_yaml
variables:
  pool: 'ubuntu-latest'
- Azure.ServiceConnectionId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
+ Azure.ServiceConnectionId: 'DockerFuncPocServiceConnection'
```

:::note info
なお、サービスコネクションの詳細ページURLにおけるクエリパラメータ(`resourceId`)として確認できるUUIDを記載してもOKです。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/2cacce1d-2aa1-acd3-297d-401629eb5f08.png" width="770">

サービスコネクション名を頻繁に変更するニーズはあまりないと思いますが、UUIDは不変なので、そのへんの心配がいらないです。
:::

# FunctionAppがAzure Container Registryに接続するための設定

## Azure Container Registry で管理者ユーザを有効化する

コンテナレジストリにログインする情報として使用します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/da5a67eb-199f-9d80-8b00-eeb4f1d4630d.png" width="640">

## FunctionAppのアプリケーション設定に認証情報を追加

コンテナレジストリの管理者情報を、下記のアプリケーション設定に追加する。

- `DOCKER_REGISTRY_SERVER_PASSWORD`
- `DOCKER_REGISTRY_SERVER_URL` ※`https://<acrname>.azurecr.io`形式
- `DOCKER_REGISTRY_SERVER_USERNAME`

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/cd7a66b8-5ada-7d7e-1f61-35fdd3722776.png" width="700">

ここまでの設定で、ACRのDockerイメージをPullしてきて、Functionsとして動かすことができるはず。

# パイプラインを動かして動作確認

## Azure DevOpsでRun Pipeline

「Run pipeline」をクリック
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/dce7ca41-fd29-ece9-86c1-a0fd46b87f46.png" width="550">

Pipelineの全てのJobがSuccessしていることを確認します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/ed78524f-c5a8-9c64-3fa9-7376632e8d4b.png" width="260">

:::note warn
Pipeline実行時にこのようなエラーが出力された場合は、以下いずれかの原因である可能性が高いです⚡️

>There was a resource authorization issue: "The pipeline is not valid. Job BuildAndPushImage: Step Docker1 input azureSubscriptionEndpoint references service connection xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx which could not be found. The service connection does not exist or has not been authorized for use. For authorization details, refer to https://aka.ms/yamlauthz."

1. サービスコネクション名かUUIDが間違っている
2. Pipelineがサービスコネクションを使用するPermissionを持っていない

原因が2だと思われる場合、こちらの手順を参考に、PipelineにPermissionを与えてください。

ServiceConnectionの詳細ページより、メニュー内の「Security」をクリック
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/21d94ad3-a985-4320-9a1d-65b9ad086134.png" width="420">

「Pipeline permission」セクションにおいて、許可を与えたいPipelineを選択
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/1582f609-f4d4-fb77-d0b9-413ad3bdcde9.png" width="420">

:::

## ACRにPushされているかチェック

ACRのリポジトリを見ると、Pipelineの中でBuildされたImageがpushされています。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/41abefa2-19dc-9002-7318-ef79c50c27eb.png" width="1000">

## AzurePortal上で関数として読み込まれているかチェック

関数アプリの「関数」ブレードより、関数としてAzure側が識別しているかをチェックします。Dockerコンテナが正常に起動できていないと、ここに何も表示されないんです😵

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/2d5de560-eef0-5233-d793-59ca1d500b01.png" width="700">

## 実際に動作確認

関数が期待する動作をしてたらOKです💪

```terminal
$ curl -X GET "https://docker-func-poc.azurewebsites.net/api/HttpTriggerSample?name=YOS0602&code=ABCDEFG=="
Hello, YOS0602. This HTTP triggered function executed successfully.
```

# 関数が読み込めていない or Dockerコンテナがうまく起動していないなどの場合

どこでコケてるか次第ですが、FunctionがImageをPullできていない場合、後述しているマネージドID有効化とIAMロール付与を試してみると良いかもしれません。
なお、npmを利用したアプリケーションでDockerを使用していると、「ユーザー名前空間再割当てエラー」が発生して、Dockerコンテナが上手く起動しないことがあります。詳細および対策はこちらの記事を参考にしてみてください。
[App Service における Docker User Namespace remapping issues について - Japan PaaS Support Team Blog - #NPM 利用プロジェクトにおける ユーザー名前空間再割当てエラー](https://jpazpaas.github.io/blog/2023/02/15/Docker-User-Namespace-remapping-issues.html#npm-%E5%88%A9%E7%94%A8%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E5%86%8D%E5%89%B2%E5%BD%93%E3%81%A6%E3%82%A8%E3%83%A9%E3%83%BC)

## FunctionAppのマネージドIDを有効化

システム割り当てマネージドIDを有効化し、「保存」します。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/8b5449e1-04aa-22d1-84c6-dfff905b9d36.png" width="450">

「はい」をクリック。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/da7004af-2fc4-c2c7-93fe-9e6c3e42e9ab.png" width="300">

## AcrPullのIAMロールを付与

コンテナレジストリのIAMより、ロールの割り当ての追加を行います。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/3c4f0b47-8a87-de38-05ca-d065a7326ca7.png" width="390">

AcrPullロールを選択して「次へ」
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/bc8e3899-3688-955b-802b-16036eb605f2.png" width="400">

先ほど有効化したFunctionAppのマネージドIDを選択する。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e65b234d-c012-2514-d326-f8163480c1bf.png" width="630">

「レビューと割り当て」を行う。

## FunctionAppを再起動

az cliでも、Portalからでも良いです。設定が反映されるまで数分かかることもあるので待ちましょう⏱️

# 参考

https://github.com/microsoft/azure-pipelines-yaml/blob/master/templates/docker-container-functionapp.yml

https://stackoverflow.com/questions/60163440/docker-fails-to-pull-the-image-from-within-azure-app-service

https://jpazpaas.github.io/blog/2023/02/15/Docker-User-Namespace-remapping-issues.html#npm-%E5%88%A9%E7%94%A8%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E5%86%8D%E5%89%B2%E5%BD%93%E3%81%A6%E3%82%A8%E3%83%A9%E3%83%BC
