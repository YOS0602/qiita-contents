---
title: Macを新しく買った時にやること
tags:
  - Mac
  - homebrew
private: false
updated_at: '2024-07-15T16:20:22+09:00'
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
  - 左上: Mission Control
  - 左下: アプリケーションウィンドウ
  - 右上: なし
  - 右下: 画面をロック

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

[macOS（またはLinux）用パッケージマネージャー — Homebrew](https://brew.sh/ja/)

## pathを通す

https://qiita.com/howaito01/items/242ba23e8a8fb00dac05

## インストールパッケージ

- [Google Chrome](https://www.google.com/chrome/)
  - `brew install --cask google-chrome`
- [Clipy](https://github.com/Clipy/Clipy)
  - Clipboard extension app for macOS.
- [Rectangle](https://rectangleapp.com/)
  - Move and resize windows in macOS
- [AltTab](https://alt-tab-macos.netlify.app/)
    - `brew install --cask alt-tab`
    - windows風にウィンドウを切り替えられる
- [drawio](https://www.diagrams.net/)
    - `brew install --cask drawio`
    - アーキ図とか描くやつ
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
    ```bash
    brew tap azure/functions
    brew install azure-functions-core-tools@4
    ```
- [VS Code](https://code.visualstudio.com/)
    - `brew install --cask visual-studio-code`
- [Microsoft Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/)
    - `brew install azure-cli`
- [Amazon AWS command-line interface](https://aws.amazon.com/cli/)
    - `brew install awscli`
- [gcloud CLI](https://cloud.google.com/sdk/docs/install?hl=ja)
- [git](https://git-scm.com/)
    - `brew install git`
    - pathを通す
        - [【MacOS】Homebrew経由でGitをインストールする方法 - シェルにHomebrew経由のGitパスを通す](https://blog.cloud-acct.com/posts/u-homebrew-git-install/#%E7%8F%BE%E7%8A%B6%E3%82%92%E6%95%B4%E7%90%86%E3%81%97%E3%81%BE%E3%81%97%E3%82%87%E3%81%86)
- [slack](https://slack.com/intl/ja-jp/)
    - `brew install --cask slack`
- [Rancher Desktop](https://rancherdesktop.io/)
- [Notion](https://www.notion.so/ja-jp/desktop)

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
    wd = diff --word-diff
    gr = grep
    f = fetch -p
    p = pull
    pu = push
    cp = cherry-pick -n
    cpc = cherry-pick
    ns = log --name-status
    lo = log --oneline
    # すっきりしたコミットログ
    loggr = log --graph --date=short --decorate=short --abbrev-commit --all --pretty=format:'%C(yellow)%H %C(reset)%cd %C(cyan)%cn %C(red)%d %C(reset)%s'
    # 上記に変更ファイルを足したもの
    loggrf = log --graph --date=short --decorate=short --name-status --abbrev-commit --all --pretty=format:'%C(yellow)%H %C(reset)%cd %C(cyan)%cn %C(red)%d %C(reset)%s'
    # いい感じのグラフでログを表示
    graph = log --graph --date=short --decorate=short --pretty=format:'%Cgreen%h %Creset%cd %Cblue%cn %Cred%d %Creset%s'
    # 上の省略形
    gr = log --graph --date=short --decorate=short --pretty=format:'%Cgreen%h %Creset%cd %Cblue%cn %Cred%d %Creset%s'
[user]
    name = XXX
    email = XXX@hoge.com
[core]
    excludesfile = /Users/user/.gitignore_global
    ignorecase = false
[init]
    defaultBranch = main
[pull]
    rebase = false
```

# Cursor (VSCode) 拡張機能

`cursor` 部分を `code` に置換すれば、VSCodeに拡張機能をinstallできる。

```sh
cursor --install-extension davidanson.vscode-markdownlint         # markdownlint
cursor --install-extension yzhang.markdown-all-in-one             # Markdown All in One
cursor --install-extension bierner.markdown-preview-github-styles # Markdown Preview Github Styling
cursor --install-extension vscode-icons-team.vscode-icons         # vscode-icons
cursor --install-extension esbenp.prettier-vscode                 # Prettier - Code formatter
cursor --install-extension inferrinizzard.prettier-sql-vscode     # Prettier SQL VSCode
cursor --install-extension streetsidesoftware.code-spell-checker  # Code Spell Checker
cursor --install-extension ExodiusStudios.comment-anchors         # Comment Anchors
cursor --install-extension ms-vscode-remote.remote-containers     # Dev Containers
cursor --install-extension ms-azuretools.vscode-docker            # Docker
cursor --install-extension mechatroner.rainbow-csv                # Rainbow CSV
cursor --install-extension janisdd.vscode-edit-csv                # Edit csv
cursor --install-extension dsznajder.es7-react-js-snippets        # ES7+ React/Redux/React-Native snippets
cursor --install-extension dbaeumer.vscode-eslint                 # ESLint
cursor --install-extension GitHub.github-vscode-theme             # GitHub Theme
cursor --install-extension HashiCorp.terraform                    # HashiCorp Terraform
cursor --install-extension ecmel.vscode-html-css                  # HTML CSS Support
cursor --install-extension oderwat.indent-rainbow                 # indent-rainbow
cursor --install-extension christian-kohler.path-intellisense     # Path Intellisense
cursor --install-extension VisualStudioExptTeam.vscodeintellicode # IntelliCode
cursor --install-extension andys8.jest-snippets                   # Jest Snippets
cursor --install-extension firsttris.vscode-jest-runner           # Jest Runner
cursor --install-extension MS-vsliveshare.vsliveshare             # Live Share
cursor --install-extension redhat.vscode-yaml                     # YAML
cursor --install-extension bradlc.vscode-tailwindcss              # Tailwind CSS IntelliSense
cursor --install-extension foxundermoon.shell-format              # shell-format
cursor --install-extension eamodio.gitlens                        # GitLens
cursor --install-extension GraphQL.vscode-graphql-syntax          # GraphQL: Syntax Highlighting
cursor --install-extension tomoki1207.pdf                         # vscode-pdf
cursor --install-extension GitHub.vscode-pull-request-github      # GitHub Pull Requests
cursor --install-extension foxundermoon.shell-format              # shell-format
```
