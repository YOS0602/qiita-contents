---
title: '[Azure Functions] function.jsonのscriptFileプロパティは、気をつけないとはまる'
tags:
  - Node.js
  - Azure
  - AzureFunctions
private: false
updated_at: '2022-09-21T23:29:40+09:00'
id: fca4a02de6c8f2afd0a4
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

困ったことが。
TimerTriggerのAzureFunctionsをローカルデバッグしていて、ある特定のタイミングで必ず別プロセスが起動してしまう。
別の言い方をすると、Functionsが二重で起動してしまう。
しかも後から立ち上がった方はブレークポイントで止まってくれない。(デバッグ用のポート(デフォルト9229)が既に使われてるから当たり前だけど

`runOnStartup`はデバッグのために`true`にしてたけど、こんなことなるの不便すぎるしどう考えても変だ。

# 環境

OS: Mac OS 12.6(Windows10端末でも再現性あり)
IDE: VSCode
Node.js: 16.14.0
npm: 8.3.1
azure function core tools: 4.0.4483

# 疑ったもの

## Azurite

ローカルデバッグ時のストレージアカウントエミュレータ
インストールし直してみたり、クリーンしたけど変わらず

## azure function core tools

これも再インストール。けど違いそう。

## そもそもFunctionsとはそういうものという説

Azureって、特にFunctionsってなんかクセが強いので。
けど、試しに`func init`でAzureFunctionsのサンプルを作ってみたけど、これは二重起動なんてしない。

# 原因

試しに`func init`で作ってみたのが功を奏して原因判明。
`function.json`の`scriptFile`をちょっとカスタマイズしていたのが原因っぽい。

# とった対応

こっちがカスタマイズしてた方(ダブル起動しちゃう)

```function.json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 0 7-9 * * *",
      "runOnStartup": false
    }
  ],
  "scriptFile": "./index.js"
}
```

こっちが直した方

```diff_json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 0 7-9 * * *",
      "runOnStartup": false
    }
  ],
- "scriptFile": "./index.js"
+ "scriptFile": "../dist/TimerFunc/index.js"
}
```

まあこれ、`func new`したらできるやつと一緒に戻しただけなんだけどね。

## `tasks.json`も修正

デバッグ用に用意してたファイルも修正。
今までやらんくて良いことしてたんやなあ。

```diff_json
    {
      "type": "func",
      "label": "launch debug",
-     "command": "host start --script-root \"./dist\" --javascript",
+     "command": "host start",
      "problemMatcher": "$func-node-watch",
      "isBackground": true,
      "dependsOn": "npm build (functions)"
    }
```

# 余談: `scriptFile`をいじったら、AzurePortal上で関数として認識されなくなった

まあ当たり前ですけど。
今までは`./index.js`のように、直下にあるファイルを指定してたけど、それが`../dist/TimerFunc/index.js`に変わったので、
Functionsの実行ファイルを見つけられなくなったんだね。
なので、`function.json`から、`scriptFile`に記載してある実行ファイルをパス通りに配置するようデプロイするなり
CICDを見直すなりする必要がある。

これが一番しんどかった。
