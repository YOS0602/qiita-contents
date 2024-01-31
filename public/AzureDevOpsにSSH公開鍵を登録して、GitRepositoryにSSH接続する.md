---
title: AzureDevOpsにSSH公開鍵を登録して、GitRepositoryにSSH接続する
tags:
  - Git
  - SSH
  - AzureDevOps
private: false
updated_at: '2023-02-10T09:51:49+09:00'
id: 974cc61a499620feecfb
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

自分はGitRepositoryにSSHで接続する派です。
Windowsマシンを使ってた頃、複数の接続先を持ちすぎたのか、頻繁にPWを聞かれてうんざりしてからSSH派になりました。

AzureDevOpsのSSH鍵有効期限は「1年」です。(2023/02/10時点)
毎年やらなきゃいけないことをその度に調べ直すのは嫌なので、メモしておきます。

なお、AzureDevOpsへの登録方法を記述しますが、GitHubでもノリは同じです。

# 環境

```text
macOS 13.1
```

# 公開鍵・秘密鍵の作成

## sshフォルダに移動

```terminal
$ cd ~/.ssh
```

## 鍵ペアの生成

```terminal
$ ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/Users/[userName]/.ssh/id_rsa): key_azure_devops
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in key_azure_devops
Your public key has been saved in key_azure_devops.pub
```

`Enter file in which to save the key (/Users/[userName]/.ssh/id_rsa):`で鍵の名前を、
`Enter passphrase (empty for no passphrase):`でパスフレーズを指定できます。

## 鍵が作成されたかの確認

```terminal
$ ls -l
```

- `key_azure_devops`
- `key_azure_devops.pub`

の2ファイルが確認できると思います。
`.pub`拡張子が「公開鍵」となり、拡張子なしファイルが「秘密鍵」となります。
公開鍵をAzureDevOpsに登録することになりますが、秘密鍵は自分以外に知られてはなりません。

# AzureDevOpsに鍵を登録する

※記事で紹介するAzureDevOpsのUIは、2023/02/10時点のものです。

## SSH公開鍵管理ページにアクセス

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/3170cc0c-e98c-ed9c-caec-af8c38d28386.png" width="320">

## 公開鍵の登録

「+ New Key」をクリック。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/faf8bb18-9164-6f02-dfd9-631a76245ed1.png" width="450">

Name: 公開鍵やマシンの名前
Public Key　Data: 公開鍵データ

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/8f3a7961-3cf1-88b6-b2de-3045639077ad.png" width="450">

公開鍵の中身をコピーする時、マウスを使ったドラッグだとミスの元なので、下記コマンドを使用するようにしましょう。

mac

```terminal
$ pbcopy < ~/.ssh/key_azure_devops.pub
```

windows

```terminal
$ clip < ~/.ssh/key_azure_devops.pub
```

## SSH configファイルでどの鍵を使用するか指定

`~/.ssh/config`を開き、下記を追記します。
`IdentityFile`には、自分で作成した鍵のパスを記述してください。

```~/.ssh/config
# Azure DevOps
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/key_azure_devops
  User git
  Port 22
  TCPKeepAlive yes
  IdentitiesOnly yes
```

### Host名を変えて、複数のAzureDevOpsアカウントを使い分ける

任意の`Host`名と、個別の`IdentityFile`を指定することで、複数のAzureDevOpsアカウント(GitHubも)があっても簡単にSSH接続情報を使い分けられます。

```~/.ssh/config
# Azure DevOps
Host work_devops
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/key_azure_devops
  User git
  Port 22
  TCPKeepAlive yes
  IdentitiesOnly yes

# private Azure DevOps
Host private_devops
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/key_private_azure_devops
  User git
  Port 22
  TCPKeepAlive yes
  IdentitiesOnly yes
```

ただし、`clone`してくる時や`remote add`する時など、デフォルトのコマンドラインではなく、自分で定義した`Host`名を使用する必要があります。

デフォルト
`git clone git@ssh.dev.azure.com:v3/[Organization]/[Project]/[RepositoryName]`

変更後
`git clone git@[Host名]:v3/[Organization]/[Project]/[RepositoryName]`

上記の`private_devops`を使うなら、
`git clone git@private_devops:v3/[Organization]/[Project]/[RepositoryName]`となります。

# 参照

https://qiita.com/shizuma/items/2b2f873a0034839e47ce

https://qiita.com/wakahara3/items/52094d476774f3a2f619

https://atmarkit.itmedia.co.jp/ait/articles/1908/02/news015.html

https://zenn.dev/taichifukumoto/articles/how-to-use-multiple-github-accounts
