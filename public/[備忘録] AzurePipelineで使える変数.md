---
title: '[備忘録] AzurePipelineで使える変数'
tags:
  - Azure
  - AzurePipelines
private: false
updated_at: '2022-09-23T15:59:16+09:00'
id: 0820d9d80808046b645d
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

`$(Build.ArtifactStagingDirectory)`とか`$(Build.BuildId)`とか
いつもなんやこれって思ってた。

# predefined variablesという名前らしい

ここに一覧載ってる。

https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml

日本語版はこちら

https://learn.microsoft.com/ja-jp/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml

# おまけ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/0a2702a3-4f67-61da-5212-ba449a0912c9.png)

リンク先ページの関連ファイルはPipeline yamlを書く上でかなり役立つので一読されたし。

## たとえば自分で変数を定義する方法とか

この例は変数を変数に入れ直してるだけだが・・・

```yaml
variables:
  - name: branchName
    value: $(Build.SourceBranchName)

steps:
  - task: PublishBuildArtifacts@1
    displayName: 'Publish Artifact'
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)/$(branchName)-$(Build.BuildId).zip'
      ArtifactName: 'Artifact_sample'
      publishLocation: 'Container'
```
