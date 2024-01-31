---
title: '[Node.js] 既に存在するPostgresから、sequelize用のmodelsファイルを生成したい'
tags:
  - sequelize
private: false
updated_at: '2023-02-09T10:38:42+09:00'
id: 891acdb1f159ea3fda28
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Node.jsの老舗ORMである`sequelize`を使用するには、公式によると下記2通りの方法で`Model`を作成しないといけません。

>Models can be defined in two equivalent ways in Sequelize:
>
>- Calling sequelize.define(modelName, attributes, options)
>- Extending Model and calling init(attributes, options)

https://sequelize.org/docs/v6/core-concepts/model-basics/#model-definition

しかし、既に実装済みのDBから、すべてのテーブル分Model定義を頑張って作るのは骨が折れます。
しかも、DDLでは`VARCHAR`なのに、Modelでは`INT`にしてしまった、なんて乖離も起きそうです。
ならリバースエンジニアリングできんかや？　と探してみたら、`sequelize-auto`なるライブラリで実現できそうだったので、使い方をメモしておきます。

# 環境

```text
macOS 13.1
node 18.12.1
npm 8.19.2
"pg": "^8.9.0",
"pg-hstore": "^2.3.4",
"sequelize": "^6.28.0",
"sequelize-auto": "^0.8.8"
```

# Reference

https://sequelize.org/docs/v6/other-topics/migrations/

# 必要になるライブラリのインストール

## sequelize-autoのインストール

```terminal
npm install sequelize-auto
```

## Postgresアクセスのためのライブラリをインストール

`sequelize`も必要になります。

```terminal
npm install sequelize pg pg-hstore
```

## TypeScript系

TypeScriptで実装したいので。

```teminal
npm i typescript @types/node ts-node
```

versionはこちら

```text
"@types/node": "^18.13.0",
"ts-node": "^10.9.1",
"typescript": "^4.9.5"
```

# 設定ファイルの準備

https://github.com/sequelize/sequelize-auto#usage

https://sequelize.org/api/v6/class/src/sequelize.js~sequelize

上記の公式を見ながら、設定ファイルを準備

```config.json
{
  "database": "",
  "host": "",
  "username": "",
  "password": "",
  "dialect": "postgres",
  "schema": "public",
  "dialectOptions": {
    "ssl": true
  },
  "directory": "./src/models",
  "port": 5432,
  "caseModel": "p",
  "caseProp": "o",
  "caseFile": "p",
  "lang": "ts",
}
```

## ちょっとだけ説明

`sequelize`の設定と、`sequelize-suto`の設定が混在しています。

- `database`や`host`なんかはDBに接続するための情報
- `caseModel`や`lang`なんかは、sequelize-autoの設定

## ちょっと詰まったところ

Postgresの`sslmode`を`require`としていたのですが、下記を設定しないと上手く接続できないのでご注意を。

```json
{
  "dialectOptions": {
    "ssl": true
  },
}
```

# コマンド発行

```terminal
npx sequelize-auto --config "./config.json"
```

これで、指定した出力先にModelファイルが出力されるはずです。

# (おまけ)sequelizeから、作成したModelを読み込む

https://github.com/sequelize/sequelize-auto#example

公式にも載ってますが。

```index.ts
import dotenv from 'dotenv';
dotenv.config();
import { Sequelize } from 'sequelize';
import { initModels } from './models/init-models';

const connectionString = process.env.CONNECTION_STRING || '';
const sequelize = new Sequelize(connectionString, {
  benchmark: true,
  logging: (sql: string, timing?: number) => {
    console.log(`${timing}ms`);
  },
});

const models = initModels(sequelize);

export { models };
```

```ts
import { models } from './index'
```

してやることで、すべてのModelにアクセスすることができ、依存関係をきれいに整理することができます。
