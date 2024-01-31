---
title: '[Azure][備忘録] サービスエンドポイント"Microsoft.Storage.Global"がAzurePortalで表示されない時の対策'
tags:
  - Azure
  - ServiceEndpoint
private: false
updated_at: '2023-05-22T17:11:21+09:00'
id: 0c329466ef44a1c935ab
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

私は画面に出るのに、他のメンバだと見れないってことが発生したので、問い合わせ内容をメモ。

2023/05/22時点の情報です。

## 本来ならこちらが表示されるはず

仮想ネットワークのサブネットより、サービスエンドポイントを選択する画面にて。
本来は「`Microsoft.Storage`」・「`Microsoft.Storage.Global`」のどちらも表示されるはず。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5f27a728-1b4c-d002-d2a1-f357b635495a.png)

他のメンバは「`Microsoft.Storage.Global`」が表示されず、キャッシュクリアやシークレットタブでのアクセスを試みても改善しなかった。

# `Microsoft.Storage.Global`サービスエンドポイントって？

https://learn.microsoft.com/ja-jp/azure/storage/common/storage-network-security?toc=%2Fazure%2Fvirtual-network%2Ftoc.json&tabs=azure-portal#azure-storage-cross-region-service-endpoints

>リージョン間サービス エンドポイントを使用すると、サブネットでは、別のリージョン内のストレージ アカウントを含めて、ストレージ アカウントとの通信にパブリック IP アドレスを使用しなくなります。 代わりに、サブネットからストレージ アカウントへのすべてのトラフィックで、ソース IP としてプライベート IP アドレスが使用されます。

リージョンが異なるストレージアカウントに対し、サービスエンドポイントを使用した特定経路通信を行いたい時に使用する。

# 問い合わせによって明らかになった原因

以下の Azure RBAC 権限が必要

- アクセス権限: `Microsoft.Network/locations/virtualNetworkAvailableEndpointServices/read`
- スコープ: サブスクリプション全体
 
従って、サブスクリプション全体に閲覧権限を持つユーザーであれば参照可能となる。動作確認も取れた。

調べたところ、こんなような権限みたい。

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/resource-provider-operations#microsoftnetwork

>`Microsoft.Network/locations/virtualNetworkAvailableEndpointServices/read`	使用可能な仮想ネットワーク エンドポイント サービスの一覧を取得します。

# Portalでは見えなくて、サブスクリプションに閲覧者権限をもらえない時のために

`az　cli`でできないか試してみる。
ただし、自分のユーザはサブスクリプションに作成者の権限を持ってしまっているので、認可の部分でエラーが出るかもしれない。

https://learn.microsoft.com/ja-jp/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update

`az network vnet subnet update`を使用したら、サブネットに`Microsoft.Storage.Global`サービスエンドポイントを追加できた。

```terminal
# 使用できるサービスエンドポイントを確認
$ az network service-endpoint list --location japaneast --output table
Name
------------------------------
Microsoft.Storage
Microsoft.Sql
Microsoft.AzureActiveDirectory
Microsoft.AzureCosmosDB
Microsoft.Web
Microsoft.KeyVault
Microsoft.EventHub
Microsoft.ServiceBus
Microsoft.ContainerRegistry
Microsoft.CognitiveServices
Microsoft.Storage.Global

# Microsoft.Storage.Globalをサブネットに追加
$ az network vnet subnet update --subscription MySubscription -g MyResourceGroup --vnet-name MyVnet --name MySubnet --service-endpoints Microsoft.Storage.Global
{
...略...
  "serviceEndpoints": [
    {
      "locations": [
        "*"
      ],
      "provisioningState": "Succeeded",
      "service": "Microsoft.Storage.Global"
    }
  ],
...略...
}
```

# 参照

https://learn.microsoft.com/ja-jp/cli/azure/network/service-endpoint?view=azure-cli-latest
