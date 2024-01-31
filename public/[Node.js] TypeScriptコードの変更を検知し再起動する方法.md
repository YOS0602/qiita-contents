---
title: '[Node.js] TypeScriptコードの変更を検知し再起動する方法'
tags:
  - Node.js
  - npm
  - TypeScript
private: false
updated_at: '2024-01-31T13:40:59+09:00'
id: e2cda3272a50184d9184
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

TypeScriptを使ってアプリケーション開発をしている時、以下のことに煩雑さを覚えたので解決方法をメモ

1. TypeScript→JavaScriptにトランスパイル
1. サーバを起動
1. コードを修正
1. サーバを止める
1. TypeScript→JavaScriptにトランスパイル
1. サーバを起動

`console.log()`を一個増やしたいなって思うだけでもこれだけの手順を踏まないといけないのは効率が悪すぎる

# `ts-node-dev`を使用する

公式はこちら

https://www.npmjs.com/package/ts-node-dev

## インストール

`npm i ts-node-dev --save-dev`
`yarn add ts-node-dev --dev`

## 使い方

`ts-node-dev [node-dev|ts-node flags] [ts-node-dev flags] [node cli flags] [--] [script] [script arguments]`

サンプル
`ts-node-dev --respawn src/index.ts`

これをnpm scriptsに登録しておくと、楽に呼び出せる

```package.json
  "scripts": {
    "dev": "ts-node-dev --respawn src/app.ts",
  }
```

### 例

```terminal
% npm run dev

> yourapp@1.0.1 dev
> ts-node-dev --respawn src/app.ts

[INFO] 17:40:21 ts-node-dev ver. 2.0.0 (using ts-node ver. 10.8.1, typescript ver. 4.7.3)
Node.js app listening on port 3000. Or Access http://localhost:3000
```

ソースをいじって保存すると勝手に再起動された

```terminal
[INFO] 17:43:55 Restarting: {PATH}/src/app.ts has been modified
Node.js app listening on port 4000. Or Access http://localhost:4000
```

## フラグ

- `--respawn` - スクリプトが終了しても引き続き監視を続ける(参考: [node-devのGitHub](https://github.com/fgnass/node-dev#command-line-options))

## 公式によると

node-devのTypeScriptサポートや、nodemonより早いらしい......
