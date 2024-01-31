---
title: '[Azure Functions] リモートビルドするように設定してデプロイしたら、CICDのzipデプロイが失敗するようになった'
tags:
  - Azure
  - AzureFunctions
  - AppService
private: false
updated_at: '2022-09-28T12:02:44+09:00'
id: 79cbf6329ee2e89d4cb7
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに結論

- funcコマンドデプロイ時にアプリケーション設定が自動で書き換えられていた
    - リモートビルド(Functionsランタイムでビルド)する設定になっていた
- 組んでいるCICDでは、AzureDevOpsのパイプラインを用い、MicrosoftHostedAgentでビルドした資産をzipデプロイする
    - ビルド後の資産にはビルド設定ファイルが含まれておらず、リモートビルドしようとして落ちていた
- なのでリモートビルドするためのアプリケーション設定をOFFったら成功するようになった

# 環境

- FunctionsランタイムOS：Linux
- アプリケーションランタイム：Node16
- コーディング言語：Typescript

# 失敗するようになった経緯

- これまではCICD(AzureDevOpsのReleasesパイプライン)は成功していた
- 色々あって、VSCodeを使ったデプロイ(リモートビルド)をやりたくなったので下記コマンドを実行
    - `func azure functionapp publish $appName --build remote`

その時参考にしたサイト

https://dev.to/azure/running-headless-chromium-in-azure-functions-with-puppeteer-and-playwright-2fgk

- 試したいことは終わったので、CICDのデプロイを動かしたところ失敗

## 失敗したリリースパイプラインのログ

抜粋し、一部改変

```log
2022-09-28T01:53:58.1244834Z ##[section]Starting: Deploy Azure Function App
...
2022-09-28T01:54:04.4089082Z Package deployment using ZIP Deploy initiated.
2022-09-28T01:56:43.2424115Z Running oryx build...
2022-09-28T01:56:43.7017455Z Command: oryx build /tmp/zipdeploy/extracted -o /tmp/build/expressbuild --platform nodejs --platform-version 16 -i /tmp/8daa0f468b80c00
...
2022-09-28T01:56:43.7045398Z Running 'npm run build'...
2022-09-28T01:56:43.7049424Z > XXXXXXXXX@1.0.0 build
2022-09-28T01:56:43.7050004Z > tsc --project ./tsconfig.build.json
2022-09-28T01:56:43.7050186Z 
2022-09-28T01:56:43.7050815Z error TS5058: The specified path does not exist: './tsconfig.build.json'.
```

`npm run build`をしようとして、`tsconfig.build.json`がないため落ちてる

# 原因

それもそのはず、CICDではビルドしたArtifactがzipデプロイされており、TS→JSにトランスパイルされているので`tsconfig.build.json`やら`tsconfig.json`などの設定ファイルはデプロイ資産に入っていない

## なぜリモートビルドが動いたのか？

`func azure functionapp publish $appName --build remote`
こちらのコマンドを使用した際の引数でリモートビルドを指定するかどうかで動きが変わるものと思っていたが、どうやらAppServiceのアプリケーション設定を書き換えてしまうようだ。
前後比較したわけではないので定かではないが、見覚えのないアプリケーション設定がいくつか追加されていた。

https://learn.microsoft.com/ja-jp/azure/azure-functions/run-functions-from-deployment-package#general-considerations

>プロジェクトでリモート ビルドを使用する必要がある場合は、WEBSITE_RUN_FROM_PACKAGE アプリ設定を使用しないでください。 代わりに、SCM_DO_BUILD_DURING_DEPLOYMENT=true デプロイ カスタマイズ アプリ設定を追加します。 Linux の場合は、さらに ENABLE_ORYX_BUILD=true 設定を追加します。 詳細については、「[リモート ビルド](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-deployment-technologies#remote-build)」を参照してください。

こちらを見ると、下記2つがリモートビルドのために書き換えられたor追加されたのだと思われる。

- `SCM_DO_BUILD_DURING_DEPLOYMENT=true`
- `ENABLE_ORYX_BUILD=true`

# CICDを元通り動くようにする

リモートビルドのためのアプリケーション設定を、どちらも`false`に変えてあげるだけ

- `SCM_DO_BUILD_DURING_DEPLOYMENT=false`
- `ENABLE_ORYX_BUILD=false`

# 参考サイト

https://logico-jp.io/2022/02/21/disable-to-build-applications-during-deploying-to-app-service/

https://learn.microsoft.com/ja-jp/azure/azure-functions/deployment-zip-push#deployment-customization

https://github.com/microsoft/Oryx/blob/main/doc/hosts/appservice.md#nodejs
