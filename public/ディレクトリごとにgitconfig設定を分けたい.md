---
title: ディレクトリごとにgitconfig設定を分けたい
tags:
  - Git
  - gitconfig
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---

# 背景

プライベートと仕事とGitHubアカウントを使い分けたい。
プライベートの設定を `--global` フラグで登録し、仕事はリポジトリごとに `--local` フラグで設定することもできるが、リポジトリ数が増えるにつれ管理が煩雑になっていく。

そこで、ディレクトリごとに適用される gitconfig を固定したい。

https://qiita.com/YOS0602/items/974cc61a499620feecfb#host%E5%90%8D%E3%82%92%E5%A4%89%E3%81%88%E3%81%A6%E8%A4%87%E6%95%B0%E3%81%AEazuredevops%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%82%92%E4%BD%BF%E3%81%84%E5%88%86%E3%81%91%E3%82%8B

こちらで紹介しているようなやり方で、SSH鍵の使い分けも同時に設定できると便利。

# 目指す姿

- `~/private-workspace` 配下: プライベート用の設定を使用する
- `~/companyA-workspace` 配下: 仕事用の設定を使用する

# 設定手順

## 1. グローバルの `.gitconfig` を編集する

特定ディレクトリ配下にいる時に適用するgitconfigを指定する。

```~/.gitconfig
[user]
    name = YOS0602-private
    email = yos@private.com
[includeIf "gitdir:~/companyA-workspace/**"]
    path = ~/.gitconfig.companyA
```

## 2. 特定ディレクトリ専用の gitconfig を作成する

```~/.gitconfig.companyA
[user]
    name = YOS0602-work
    email = yos@companyA.com
```

# 参照

https://zenn.dev/krabben16/articles/switch-gitconfig-by-directory
