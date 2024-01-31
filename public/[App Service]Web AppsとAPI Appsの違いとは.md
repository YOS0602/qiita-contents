---
title: '[App Service]Web AppsとAPI Appsの違いとは'
tags:
  - Azure
  - WebApps
  - AppService
private: false
updated_at: '2022-05-17T14:49:05+09:00'
id: 0e4b34f74c9e8acb1c7e
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

```txt
App Service
  |-web apps
  |-API apps
```

上記の関係前提で喋ってます。間違ってたら指摘ください。

## やりたいこと

SpringBootとか、Expressを使用して、Azure上にAPIサーバを作りたい

## 記事を書いた経緯

やりたいことの実現方法を調べていると、AzureにはWeb AppsとAPI Appsなるサービスがあることを知りました。ただ、ぱっと見は名前とアイコン以外に違いはあるのかよく分からなかったので、ちょっと書き残しておきます。

# 公式ドキュメント

[Azureのドキュメント](https://docs.microsoft.com/ja-jp/azure/?product=web)では、API AppsもApp Serviceも、下記[App Service のドキュメント](https://docs.microsoft.com/ja-jp/azure/app-service/)にリンクします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/2c4ce245-9cae-e2dd-f45d-cb9431c18a46.png)

https://docs.microsoft.com/ja-jp/azure/app-service/

# 結論

できることは一緒
※2022/5/17時点の意見

私はAPI appsを使ったことがないので、ネットで調べた情報だけで判断してますが、それぞれに「強み」はあれど、できることは「一緒」なのかなあ、というのが結論です。

## 個人的にわかったこと

### 本当に違いがなさそうだなあ

[API AppsのOverviewページ](https://azure.microsoft.com/ja-jp/services/app-service/api/#documentation)のドキュメントリンクは、すべてApp Serviceのものに繋がってる

### 認証の種類

AzureADは知ってたけど、下記プロバイダのサポートもあったのは初めて知った。

* [Facebook](https://developers.facebook.com/docs/facebook-login)
* [Google](https://developers.google.com/identity/choose-auth)
* [Twitter](https://developer.twitter.com/en/docs/basics/authentication)
* [OpenID Connect](https://openid.net/connect/)

## 想定される使い分け

[参考](#参考)2より
>When your business application has got multiple UI components to be compatible with Mobile and desktop interfaces, the UI applications can be hosted in the Mobile App and Web App respectively. However, the underlying business logic needs to be severed from a common source to maintain consistency. In this scenario hosting the business tier in an Azure API App will be the best solution.

モバイル・デスクトップのインターフェースに適応した、複数のUIコンポーネントを持つフロントエンドアプリケーションは、それぞれMobile AppとWeb Appにホストすることができる。しかし、ビジネスロジックは一貫性を保つために切り離す必要があり、このようなシナリオではビジネス層をAzure API Appにホストするのがベストプラクティスとなる。

うーーん……それこそWeb Appsでもいいのでは😂

# 参考

1. [Azure API App vs Web App](https://www.serverless360.com/blog/azure-api-app-vs-web-app)
2. [What is the difference between an API App and a Web App?](https://intellipaat.com/community/1183/what-is-the-difference-between-an-api-app-and-a-web-app#:~:text=An%20API%20(Application%20programming%20interface,easily%20with%20the%20other%20programs. )
2. [Azure App Service および Azure Functions での認証と承認](https://azure.microsoft.com/ja-jp/services/app-service/api/#documentation)
