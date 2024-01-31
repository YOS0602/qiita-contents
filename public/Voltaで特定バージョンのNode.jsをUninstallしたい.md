---
title: Voltaで特定バージョンのNode.jsをUninstallしたい
tags:
  - Node.js
  - volta
private: false
updated_at: '2022-11-23T18:04:12+09:00'
id: 04579e9933b90cce576a
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

[Volta公式のUninstallコマンドに関するリファレンス](https://docs.volta.sh/reference/uninstall)には、packageのUninstallについては言及があるものの、Node自体については特にナシ
Nodeバージョンを増やしすぎて、`volta list all`した時にわんさか出てくるのも嫌なので、削除方法をメモ

2019年3月にはIssueが発行されているが、まだVoltaビルトインの機能としては対応されていないみたい
[Support volta uninstall for node and yarn #327](https://github.com/volta-cli/volta/issues/327)

# 方法

以下パスにあるバージョン番号のディレクトリを削除

- Mac - `~/.volta/tools/image/node/`
- Win - `%LOCALAPPDATA%\Volta\tools\image\node\`

`rm -rf ~/.volta/tools/image/node/x.x.x`
Finderで消しに行ってもいいが、コマンドで実行すると楽

## 例

```terminal
$ volta list node
⚡️ Node runtimes in your toolchain:

    v16.13.0
    v16.17.1 (default)
    v18.12.1

$ rm -rf ~/.volta/tools/image/node/16.13.0

$ volta list node                         
⚡️ Node runtimes in your toolchain:

    v16.17.1 (default)
    v18.12.1 
```

# 参考

- [VoltaでNodeをアンインストールする](https://blog.70-10.net/2021/09/29/volta-uninstall-node/)
- [Unclear how to remove Node.js version #855](https://github.com/volta-cli/volta/issues/855#issuecomment-713218171)
