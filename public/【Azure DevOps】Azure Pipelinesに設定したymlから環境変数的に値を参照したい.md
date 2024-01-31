---
title: 【Azure DevOps】Azure Pipelinesに設定したymlから環境変数的に値を参照したい
tags:
  - 環境変数
  - Pipeline
  - AzureDevOps
private: false
updated_at: '2023-01-07T00:12:51+09:00'
id: f8481555c5047d8aea8d
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

CICDのために欠かせないパイプライン
シークレットとして扱いたい文字列や、可変にしたい設定値などがよくある
ローカルマシンやサーバであれば環境変数として設定できるが、CIを動かしているAgentではどのように可変でプライベートな値を設定するのかを記載する

## 前提

- Pipelines/Libraryを使用した方法を記載する
    - `Variable groups`を使用
    - `Secure files`はここでは取り上げない
- KeyVaultを使用する方法もあるが、やっとことないのでここでは取り上げない
    - 参考
        - [Azure Pipelines で Azure Key Vault シークレットを使用する](https://learn.microsoft.com/ja-jp/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=yaml)
        - [パイプラインで Azure Key Vault シークレットを使用する](https://learn.microsoft.com/ja-jp/azure/devops/pipelines/release/key-vault-in-own-project?view=azure-devops&tabs=portal)
- 使用するAgentはMicrosoftHosted
- Pipelineは既に作成済みとする

# LibraryにVariable groupsを作成

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/4272668c-2281-fd56-9f77-55dae908681a.png)

「+ Variable group」をクリック

## Variablesを定義

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/7d23abd8-a107-0ce2-d45a-653484321d54.png)

1. groupに名前をつける
2. 環境変数として使用したい値をkey valueの形式で追加する

# yamlに値を参照するための記述を追加

```ci.yml
# variable groupから環境設定を読み込み
variables:
  - group: qiita-env

steps:
  - bash: echo $(Build.SourceBranch)
    displayName: 'echo working branch name'

  - script: |
      npm test
    displayName: 'unit test'
    env:
      SECRET: $(SECRET) # $(variable)の形式でLibraryに定義した値にアクセスできる
```

# 参照

- [変数グループ内のシークレット変数を参照する](https://learn.microsoft.com/ja-jp/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#reference-secret-variables-in-variable-groups)

# 余談

## LibraryにアクセスできるPipelineを制限する

デフォルトだと、OrganizationかProjectか分からないが、ほぼ全てのPipelineがLibrary/Variable groupにアクセスできてしまう
セキュリティ向上のためにも、アクセスできるPipelineを限定しておくと良い

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/178dca0c-f66d-af60-f8b1-a788c6d45ccb.png)
