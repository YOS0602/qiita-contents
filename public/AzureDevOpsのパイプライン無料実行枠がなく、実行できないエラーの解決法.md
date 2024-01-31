---
title: AzureDevOpsのパイプライン無料実行枠がなく、実行できないエラーの解決法
tags:
  - Azure
  - AzureDevOps
private: false
updated_at: '2022-08-31T13:51:25+09:00'
id: 0c105bc15052aa7204df
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Azure PipelinesでCIパイプラインを作成し、テストを兼ねて実行しようとしたところ、「Microsoft Hosted環境でのパイプライン無料実行枠が購入されていない、または許可されていない」旨のエラー。
動かせるところを目標とし、解決策を記載する。

# 事象が起きたAzure DevOps環境について

- 作成： Portalからではなく[サービス概要ページ](https://azure.microsoft.com/ja-jp/services/devops/)から
- Billing設定、Azure ADの紐付けは後から
- Organizationの中にProjectは1つのみ
- パイプライン実行自体はOrganizationの中で初めて

# 出力されたエラーメッセージ

```terminal
##[error]No hosted parallelism has been purchased or granted. To request a free parallelism grant, please fill out the following form https://aka.ms/azpipelines-parallelism-request
Pool: Azure Pipelines
Image: ubuntu-latest
Started: Today at 0:45
Duration: 19m 0s

Job preparation parameters
```

# 原因調査

## OrganizationのBilling設定を確認

問題なく支払い先のAzureサブスクリプションは登録されている。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/a4031ad4-1254-235f-1b08-bb6b89841401.png)

`Paid parallel jobs`が`0`であるせいかとも考えたが、他のパイプライン実行を問題なく動かせるDevOps Organization設定を見ると、同じように`0`設定だった。

エラーメッセージの言う通り、「無料枠」としてパイプラインが動かせる時間が確保（Microsoftから許可）されているかいないか、という単純なことらしい。

# 解決策

2通りの方法がある。

## その1、無料枠をリクエストする

パイプライン実行ログに記載されている通り、無料枠リクエストする[フォーム](https://aka.ms/azpipelines-parallelism-request)に必要事項を記入し登録する。
2、3営業日かかるので、急ぎの場合は[その2](#その2パイプライン実行に支払いを設定する)のやり方で。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/434b3f21-b4c8-79ee-3bbb-07f14747dec5.png)

### メール返答

こんなメールがMicrosoftから来ます。


>Hi [XXXXXXXXXX],
>We’ve received your request to increase free parallelism in Azure DevOps.
>Please note that your request was Completed
>
>Request Details:
>\-----------------------------------------------------------------------------
>Name: [XXXXXXXXXX]
>Email: [EMAIL@ADDRESS.com]
>Organization Name: [Your Azure DevOps URL]
>Parallelism Type: Private
>\-----------------------------------------------------------------------------
>
>Request Free Parallelism for your organization: [Azure DevOps Parallelism Request Form](https://aka.ms/azpipelines-parallelism-request)
>Useful information:
>Azure DevOps Documentation: [Configure and pay for parallel jobs](https://docs.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops&tabs=ms-hosted)
>Azure DevOps Blog post [Change in Azure Pipelines Grant for Private Projects](https://devblogs.microsoft.com/devops/change-in-azure-pipelines-grant-for-private-projects/)
>Azure DevOps Blog post [Change in Azure Pipelines Grant for Public Projects](https://devblogs.microsoft.com/devops/change-in-azure-pipelines-grant-for-public-projects/)

このメールを受け取った後にパイプラインをRunしたら正常に動きました。

## その2、パイプライン実行に支払いを設定する

[OrganizationのBilling設定を確認](#organizationのbilling設定を確認)項で貼った画像の`Paid parallel jobs`入力欄を`1`以上の値とする。
料金（月額¥4,000くらい）はかかりますが、一番手っ取り早い。
