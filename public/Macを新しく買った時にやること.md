---
title: Macを新しく買った時にやること
tags:
  - Mac
  - homebrew
private: false
updated_at: '2023-07-02T16:41:27+09:00'
id: d723e3f35b7a738ef97e
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

開発端末としてMacを新しくした時に、色々とやることをメモしておく。全て自分のため。
気が向いたらインストールパッケージもうちょっとカテゴリ分けしようかな。

# 端末設定

- ライブ変換をOFF

## システム環境設定

### デスクトップとスクリーンセーバ

- 時計と一緒に表示→ON
- ホットコーナー
    - 右上:　画面をロック
    - 右下: クイックメモ

### Dockとメニューバー

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/196f3a4e-2965-f865-a71d-7c287ca17d91.png" width="600">

#### コントロールセンター

- Bluetooth: メニューバーに表示
- バッテリー
    - <img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/d402ab7d-cc13-8a29-8b3e-e149a753782f.png" width="180">
- ファストユーザスイッチ: メニューバーに表示
- 時計
    - <img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/751ebfa3-3400-2e01-8907-1d0f8fd87f4b.png" width="300">
- Spotlight: メニューバーに表示→OFF

### ソフトウェア・アップデート

- Macを自動的に最新の状態に保つ

### サウンド

- 通知音
    - Pebble
- 起動時にサウンドを再生→OFF

### キーボード

- F1、F2などのキーを標準のファンクションキーとして使用→ON

#### 修飾キー

- Caps Lock→Commandキー

#### 入力ソース

- 候補表示: ヒラギノ角ゴシック W3
- 数字を全角入力→OFF

### トラックパッド

#### ポイントとクリック

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/df792740-507f-7d50-630f-29c0b21d8cbf.png" width="620">

#### スクロールとズーム

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5d0f9ff2-d0d7-e6b8-3e41-c9047d9201a2.png" width="420">

#### その他のジェスチャ

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/4bb54c37-49e8-147a-384c-aea39f4a2ffa.png" width="350">

### ディスプレイ

#### NightShift

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/6b4dbd73-d122-0ca5-038c-1fee0060cfd5.png" width="500">

## Finder

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/818ce2c1-e04b-7b87-5e56-58f8a9ee0141.png" width="450">

# Homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

## pathを通す

https://qiita.com/howaito01/items/242ba23e8a8fb00dac05

## インストールパッケージ

- [Google Chrome](https://www.google.com/chrome/)
    - `brew install --cask google-chrome`
- [Maccy](https://maccy.app/)
    - `brew install --cask maccy`
    - 複数のクリップボード履歴を保持する
    - 設定メモ
        - ショートカット: Shift + Command + V
        - ログイン時に実行
        - 自動的にアップデートを確認
        - 自動的に貼り付け
        - 保管サイズ: 20
- [Spectacle](https://www.spectacleapp.com/)
    - `brew install --cask spectacle`
    - Windowsっぽくウィンドウをリサイズする
    - デフォルト
        - <img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/9b14ae71-98e9-3ed8-163a-f6082ba0411f.png" width="530">
    - 設定
        - <img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e8d959fa-edb6-4a01-8d76-0c6e651097f3.png" width="530">
- [AltTab](https://alt-tab-macos.netlify.app/)
    - `brew install --cask alt-tab`
    - windows風にウィンドウを切り替えられる
- [drawio](https://www.diagrams.net/)
    - `brew install --cask drawio`
    - アーキ図とか描くやつ
- [plantuml](https://plantuml.com/ja/)
    - `brew install plantuml`
    - `brew install graphviz` クラス図を書くならこっちも
    - [openjdk — Homebrew Formulae](https://formulae.brew.sh/formula/openjdk)より、pathを通すためにシンボリックリンクを貼る必要がある

        ```
        sudo ln -sfn $HOMEBREW_PREFIX/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
        ```
- [dbeaver-community](https://dbeaver.io/)
    - `brew install --cask dbeaver-community`
    - DBクライアント
- [Authy](https://authy.com/)
    - `brew install --cask authy`
    - MFAコードジェネレータ
- [postman](https://www.postman.com/)
    - `brew install --cask postman`
    - API tool
- [pyenv](https://github.com/pyenv/pyenv)
    - `brew install pyenv`
    - Pythonバージョン管理
- [sqlite](https://sqlite.org/index.html)
    - `brew install sqlite`
- [volta](https://volta.sh)
    - `brew install volta`
    - Node.jsバージョン管理
- [JDK from Oracle](https://www.oracle.com/java/technologies/downloads/)
    - `brew install --cask oracle-jdk`
- [azure-data-studio](https://docs.microsoft.com/ja-jp/sql/azure-data-studio/?view=sql-server-ver16)
    - `brew install --cask azure-data-studio`
- [Azure Functions Core Tools](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-run-local?tabs=v4%2Cmacos%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools)
    ```
    brew tap azure/functions
    brew install azure-functions-core-tools@4
    ```
- [VMware Horizon Client](https://www.vmware.com/jp.html)
    - `brew install --cask vmware-horizon-client`
- [VS Code](https://code.visualstudio.com/)
    - `brew install --cask visual-studio-code`
- [Microsoft Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/)
    - `brew install azure-cli`
- [Amazon AWS command-line interface](https://aws.amazon.com/cli/)
    - `brew install awscli`
- [git](https://git-scm.com/)
    - `brew install git`
    - pathを通す
        - [【MacOS】Homebrew経由でGitをインストールする方法 - シェルにHomebrew経由のGitパスを通す](https://blog.cloud-acct.com/posts/u-homebrew-git-install/#%E7%8F%BE%E7%8A%B6%E3%82%92%E6%95%B4%E7%90%86%E3%81%97%E3%81%BE%E3%81%97%E3%82%87%E3%81%86)
- [slack](https://slack.com/intl/ja-jp/)
    - `brew install --cask slack`
- [Go](https://go.dev/)
    - `brew install go`
    ```terminal
    # 環境変数の設定
    cd $HOME
    mkdir go
    export GOPATH=/Users/[username]/go
    ```
- [.NET](https://dotnet.microsoft.com/ja-jp/download)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
    - `brew install --cask docker`
- [Rancher Desktop](https://rancherdesktop.io/)

# アプリケーション

- [Logicool Options](https://www.logicool.co.jp/ja-jp/software/options.html)
    - Logicool製品のカスタマイズ
- Kindle
- kobo
- [DaVinci Resolve](https://www.blackmagicdesign.com/jp/products/davinciresolve)

# Git設定

```~/.gitignore_global
.DS_Store
```

```~/.gitconfig
[alias]
	st = status
	co = checkout
    cob = checkout -b
	b = branch
	ba = branch -a
	bd = branch -d
	bdd = branch -D
	cm = commit
	df = diff
	gr = grep
	f = fetch -p
	p = pull
	cp = cherry-pick -n
	cpc = cherry-pick
    pu = push
	ns = log --name-status
	wd = diff --word-diff
	# すっきりしたコミットログ
	loggr = log --graph --date=short --decorate=short --abbrev-commit --all --pretty=format:'%C(yellow)%H %C(reset)%cd %C(cyan)%cn %C(red)%d %C(reset)%s'
	# 上記に変更ファイルを足したもの
	loggrf = log --graph --date=short --decorate=short --name-status --abbrev-commit --all --pretty=format:'%C(yellow)%H %C(reset)%cd %C(cyan)%cn %C(red)%d %C(reset)%s'
    # いい感じのグラフでログを表示
    gr = log --graph --date=short --decorate=short --pretty=format:'%Cgreen%h %Creset%cd %Cblue%cn %Cred%d %Creset%s'
[user]
	name = XXX
	email = XXX@hoge.com
[core]
	excludesfile = /Users/user/.gitignore_global
	ignorecase = false
[init]
	defaultBranch = main
```
