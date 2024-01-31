---
title: '[Azure SDK for JavaScript] ContainerClient.listBlobsFlat が上手く動かず、blob一覧が取得できない'
tags:
  - Azure
  - BlobStorage
private: false
updated_at: '2023-01-20T22:22:25+09:00'
id: 51b200ccdb2f5e5d0898
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

- `@azure/core-http`の特定バージョンにバグが含まれていたのが原因
    - 私の環境では`2.3.0`が入っていて、このバージョンだと上手く動かない模様
- 現在は修正されているため、`@azure/storage-blob`や`package-lock.json`をクリーンアップし、`@azure/core-http`を`2.3.1`以上にする

# 環境

```txt
MacOS:　13.1
Node.js: v18.12.1
npm: 8.19.2
@azure/storage-blob: 12.12.0
```

# 関連Issue

GitHubのIssuesを参照

https://github.com/Azure/azure-sdk-for-js/issues/23762#issuecomment-1309194107

## 適当に流れをざっくり説明(違ってたらごめん)

- `ContainerClient`.`listBlobsFlat`やら`listBlobsByHierarchy`なんかの一覧表示系メソッドが動かへんのやけど
    - ワイも！
- `@azure/core-http`の`2.3.0`がどうやら原因らしい
- `@azure/core-http`の問題があるバージョンはnpmで配信されないよう`deprecate`したで
- bugfixした`@azure/core-http@2.3.1`をリリースしたで！

まあこんな感じですかね。
Issueの発行からCloseまでが24時間以内で終わってるんだからすごいものです。

# 実施した対応内容

## `package-lock.json`の削除

`@azure/core-http`の`2.3.0`が入っていたので、一旦`package-lock.json`を削除

```package-lock.json
{
  "dependencies": {
......略......
    "@azure/core-http": {
      "version": "2.3.0",
      "resolved": "https://registry.npmjs.org/@azure/core-http/-/core-http-2.3.0.tgz",
......略......
```

## 再度`npm install`

依存関係を入れ直します

## `@azure/core-http`のアップデートを確認

`package-lock.json`を確認し、`@azure/core-http`が`2.3.1`となっているのを確認

```package-lock.json
{
  "dependencies": {
......略......
    "@azure/core-http": {
      "version": "2.3.1",
      "resolved": "https://registry.npmjs.org/@azure/core-http/-/core-http-2.3.1.tgz",
......略......
```

## 再度動作確認

Blobが一覧表示されるようになりました

# keyword

同じ問題で困ってる人たちが検索でこのページを見つけられるように

- listBlobsFlat
- listBlobsByHierarchy
- not working
- Azure Blob Storage

# 参照

https://learn.microsoft.com/en-us/javascript/api/@azure/storage-blob/containerclient?view=azure-node-latest

https://github.com/Azure/azure-sdk-for-js/issues/23762
