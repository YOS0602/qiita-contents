---
title: AppServiceの接続文字列に騙されるな！
tags:
  - Azure
  - AppService
  - AzurePortal
private: false
updated_at: '2023-01-06T23:15:26+09:00'
id: ae86ff0443598899e32e
organization_url_name: null
slide: false
ignorePublish: false
---
# やりたいこと

環境変数として参照できるように、データベースの接続文字列をApp Serviceに設定したい。

# 簡単やん！

## Azure Portalの設定

[App Service]-[構成]-[接続文字列]からちょちょいのちょいやん！

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/edac8bb6-b608-b96a-5f82-27bc2f8fc6e6.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/6feb797f-56ac-e29b-ada7-772b3b51b202.png)


## ファイルでの記述

んで、こうしてやればええんやろ

```js
// javascriptの例
process.env.DATABASE_URL
```

# 参照できない

`DATABASE_URL`が見つからない旨のエラーが出てアプリが落ちてしまう。

## なぜか

公式ドキュメントを見ると、AppServiceに設定する接続文字列について以下の記述があります。

https://docs.microsoft.com/ja-jp/azure/app-service/configure-common?tabs=portal#configure-connection-strings

>実行時に、接続文字列は、前に次の接続の種類が付加された環境変数として使用できます。
SQLServer: SQLCONNSTR_
MySQL: MYSQLCONNSTR_
SQLAzure: SQLAZURECONNSTR_
カスタム: CUSTOMCONNSTR_
PostgreSQL: POSTGRESQLCONNSTR_

選択した種類に応じて、キーの頭に特定の文字列が付与されてたからです。

## 解決策

だから、さっきの`DATABASE_URL`は、選択したDB種類が「SQLAzure」なので、こうしないといけなかったんです。

```js
process.env.SQLAZURECONNSTR_DATABASE_URL
```

`.NET`系はうまいことやってくれる(ことが多い)みたいですが、JavaとかNodeとかだと、気を遣って書式をAzureに合わせてあげないといけません。

# おわりに

僕はアプリケーション設定に登録する派です。
