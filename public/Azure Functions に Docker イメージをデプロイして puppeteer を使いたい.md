---
title: Azure Functions に Docker イメージをデプロイして puppeteer を使いたい
tags:
  - Docker
  - AzureFunctions
  - puppeteer
private: false
updated_at: '2023-06-08T17:34:09+09:00'
id: 8c668b14611da071469f
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

今回試したコードはGitHubで公開しています。

https://github.com/YOS0602/azure-function-docker-puppeteer

この記事の内容を執筆するにあたり、技術検証段階でAzureサポートのOさんに大きなご助力をいただきました。この場をお借りして感謝申し上げます💐

## 今回やりたいこと

WebスクレイピングやE2Eテストで利用される[Puppeteer](https://pptr.dev/)を、Azure Functionsで動かしたい。
ただし、公式で利用できると発表されている環境とは、少し異なる設定で開発したい。

公式発表: Azure Functions Linux Consumption(消費量)プラン であれば実行可能
動かしたい環境: Azure Functions Linux App Service プラン

## Azure Functions on Linux では、公式にheadless Chromiumがサポートされている

ただし、LinuxのConsumption(消費量)プランのみ。
この発表は2020年8月だが、自分が調べた感じ現在も変わっていないっぽい🤔

<iframe width="560" height="315" src="https://www.youtube.com/embed/U5qF3h-b0vU?start=770" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## 前提として、なぜDockerを使うのか

公式にpuppeteerが動作すると保証されているAuzre Functionsの価格プランは「Consumptionプラン」のみですが、「App Service プラン」を選択しています。このプランだと、puppeteerが動作するのに必要な依存ライブラリがインストールされていないので、Dockerを利用し、コンテナに依存を事前にインストールするようにします。

### 「App Service プラン」を選択する理由

- VNet統合を使いたいから

## Dockerを利用する理由

1. 従量課金である「消費量」プランではなく、「App Service プラン」を使いたいが、Chromium実行に必要な依存ライブラリがインストールされていない
2. [Azure Functions (Linux) で Puppeteer が使えるようになってた - ほりひログ](https://uncaughtexception.hatenablog.com/entry/2020/08/30/091852)で言及されているように、日本語フォント問題がある

どちらも、`package.json`の`postinstall`scriptなんかでshellを実行し、ライブラリやフォントをインストールして対策することはできると思います。ですが、PaaSであるFunctionsのランタイムに直接コマンド実行するのはなんか違くね？😑 とも思ったので。

---

では、Dockerを使ってAzure Functionsでpuppeteerを動かしていきましょう💪

# Functionsリポジトリを作成

[Azure Functions Core Tools](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-run-local?tabs=v4%2Cmacos%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools)はv4を使用しています。

```terminal
$ func version
4.0.5030
```

```terminal
$ mkdir azure-function-docker-puppeteer
$ cd azure-function-docker-puppeteer

# typescriptのFunctions雛形を作る
$ func init --worker-runtime node --docker --language typescript
Writing .funcignore
Writing package.json
Writing tsconfig.json
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing /Users/user/Desktop/azure-function-docker-puppeteer/.vscode/extensions.json
Writing Dockerfile
Writing .dockerignore

# "sample"という名前の関数を、HTTP Triggerで作成する
$ func new --template "HTTP trigger" --name sample --authlevel function
Select a number for template:HTTP trigger
Function name: [HttpTrigger] Writing /Users/user/Desktop/azure-function-docker-puppeteer/sample/index.ts
Writing /Users/user/Desktop/azure-function-docker-puppeteer/sample/function.json
The function "sample" was created successfully from the "HTTP trigger" template.
```

依存パッケージをインストールします📦
テンプレートで用意されてるパッケージのバージョンが少し古いので、そのアップデートも含みます。

```terminal
$ npm i puppeteer
$ npm i -D typescript @types/node @azure/functions azure-functions-core-tools
```

puppeteerの動作確認をするためのコードを記述します✍️

```sample/index.ts
import { AzureFunction, Context, HttpRequest } from '@azure/functions';
import puppeteer, { Browser } from 'puppeteer';

const httpTrigger: AzureFunction = async function (
  context: Context,
  req: HttpRequest
): Promise<void> {
  let browser: Browser;

  try {
    browser = await puppeteer.launch({
      headless: 'new',
      args: ['--no-sandbox'],
      defaultViewport: {
        width: 1200,
        height: 900,
      },
      // WindowsPCでデバッグするときはコメントアウトを外す
      // See https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-windows
      // ignoreDefaultArgs: ['--disable-extensions'],
    });
  } catch (error) {
    context.log.error(error);
    throw new Error('Failed to launch puppeteer browser.');
  }

  try {
    const url = req.query.url || 'https://google.com/';

    const page = await browser!.newPage();
    await page.setExtraHTTPHeaders({
      'Accept-Language': 'ja-JP',
    });

    await page.goto(url);
    const screenshotBuffer: string | Buffer = await page.screenshot();

    context.res = {
      body: screenshotBuffer,
      headers: {
        'content-type': 'image/png',
      },
    };
  } catch (error) {
    context.log.error(error);
    throw new Error('Error!!');
  } finally {
    await browser!.close();
  }
};

export default httpTrigger;
```

## ローカルでFunctionsを起動してみる

まずはローカルで起動してみる。
※まだDockerは使用しません。

```terminal
azure-function-docker-puppeteer $ npm start

> azure-function-docker-puppeteer@1.0.0 prestart
> npm run build

> azure-function-docker-puppeteer@1.0.0 build
> tsc

> azure-function-docker-puppeteer@1.0.0 start
> func start

Azure Functions Core Tools
Core Tools Version:       4.0.5198 Commit hash: N/A  (64-bit)
Function Runtime Version: 4.21.1.20667

Functions:

        sample: [GET,POST] http://localhost:7071/api/sample

For detailed output, run func with --verbose flag.
[2023-06-08T07:12:32.943Z] Worker process started and initialized.
```

ブラウザで`http://localhost:7071/api/sample`にアクセスすると、Googleのトップ画面が表示されます🖥️

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/c64595b4-eacc-a9f3-2b7f-c12027570b51.png" width="600">

確認できたら、`Cmd` + `C` or `Ctrl` + `C` で起動しているfunctionを停止しておきましょう。

# Dockerfileを作成

https://github.com/puppeteer/puppeteer/blob/puppeteer-v20.5.0/docs/troubleshooting.md#running-puppeteer-in-docker

Puppeteer公式のDocsには`Running Puppeteer in Docker`の見出しで、Dockerfileが公開されていますが、そのまま使ってもFunctionsとして上手く動かすことができなかったので編集しています。
作成したDockerfileはこちら↓

```Dockerfile:Dockerfile
# To enable ssh & remote debugging on app service change the base image to the one below
# FROM mcr.microsoft.com/azure-functions/node:4-node16-appservice
FROM mcr.microsoft.com/azure-functions/node:4-node18-appservice

ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true \
    DEBCONF_NOWARNINGS=yes

WORKDIR /home/site/wwwroot
COPY . /home/site/wwwroot
RUN cd /home/site/wwwroot

RUN apt-get update \
    && apt-get install -y wget gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# If running Docker >= 1.13.0 use docker run's --init arg to reap zombie processes, otherwise
# uncomment the following lines to have `dumb-init` as PID 1
# ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_x86_64 /usr/local/bin/dumb-init
# RUN chmod +x /usr/local/bin/dumb-init
# ENTRYPOINT ["dumb-init", "--"]

# Uncomment to skip the chromium download when installing puppeteer. If you do,
# you'll need to launch puppeteer with:
#     browser.launch({executablePath: 'google-chrome-stable'})
# ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

RUN npm install \
    # NPM 利用プロジェクトにおける ユーザー名前空間再割当てエラーに対する対策
    # See https://jpazpaas.github.io/blog/2023/02/15/Docker-User-Namespace-remapping-issues.html#npm-%E5%88%A9%E7%94%A8%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E5%86%8D%E5%89%B2%E5%BD%93%E3%81%A6%E3%82%A8%E3%83%A9%E3%83%BC
    && find /home/site/wwwroot/node_modules ! -user root | xargs --no-run-if-empty chown root:root

CMD ["google-chrome-stable"]

RUN npm run build
```

# FunctionAppにデプロイ

デプロイに Azure container registry (ACR)を使用するなら、私が以前書いたこちらの記事を参考にできるかもしれません。

https://qiita.com/YOS0602/items/200dee18991de8c04925

今回は、AzurePipelinesで以下のような流れでデプロイする前提とします。

1. DockerImageをbuildする
2. buildしたimageをACRにpushする
3. FunctionAppにデプロイ設定する

今回紹介するやり方だと、FunctionApp内でimageをpullし、コンテナを起動するのはAzureが勝手にやってくれます。

## pipeline YAMLを作成する

<details><summary>長いので折りたたんでいます✂️</summary>

```azure-pipelines.yml
trigger: none

resources:
  - repo: self

# Pipelineが以下の変数を参照できるように設定してください。
# - AzureServiceConnectionId: Service Connection Name or UUID
# - FunctionAppName: The resource name of Azure Functions
# - ACRRepositoryName: The repository name of ACR. This is also used as an Docker image name
# - ACRResourceName: Sub domain name of ACR server
variables:
  pool: 'ubuntu-latest'
  Azure.ServiceConnectionId: $(AzureServiceConnectionId)
  FunctionApp.Name: $(FunctionAppName)
  ACR.Name: $(ACRRepositoryName)
  ACR.ImageName: '$(ACR.Name):$(Build.BuildId)'
  ACR.FullName: '$(ACRResourceName).azurecr.io'

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

## Pipelineを動かす

Pipelinesが成功して、しばらく待てばAzure Functionsリソースでimageがpullされ、コンテナが起動しているはずです。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/3b75c4b7-1207-ab13-0f98-e103d9c5db82.png" width="450">

# Functionsで動作確認

デプロイできたら、実際のAzure FunctionsリソースのHTTPエンドポイントをコールして動作確認してみましょう。

`/api/sample?url=https://qiita.com/YOS0602`(※function keyは省略)にアクセスすると、日本語が文字化けせず表示されていますね🧑‍💻

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/63514d9e-6ddd-10a0-95bd-03aee5813d1c.png" width="780">

興味本位でアラビア語のサイトを試してみたけど表示されました。豆腐にならなくて良かった😌

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/cd63aaf3-958f-4514-d1c9-18129ed02c76.png" width="650">

# 参考記事

https://uncaughtexception.hatenablog.com/entry/2020/08/30/091852

https://uncaughtexception.hatenablog.com/entry/2022/12/09/191520

https://qiita.com/georgeOsdDev@github/items/636314b488286b6024a4

https://anthonychu.ca/post/azure-functions-headless-chromium-puppeteer-playwright/

https://qiita.com/kouphasi/items/991c1b9da6d685b9cc36

https://qiita.com/QUANON/items/59c468adfff0278f20bb

https://qiita.com/m-tmatma/items/10a02d60ae7b2cc6f32b

https://github.com/puppeteer/puppeteer/blob/puppeteer-v20.5.0/docs/troubleshooting.md#running-puppeteer-in-docker

https://jpazpaas.github.io/blog/2023/02/15/Docker-User-Namespace-remapping-issues.html#npm-%E5%88%A9%E7%94%A8%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E5%86%8D%E5%89%B2%E5%BD%93%E3%81%A6%E3%82%A8%E3%83%A9%E3%83%BC
