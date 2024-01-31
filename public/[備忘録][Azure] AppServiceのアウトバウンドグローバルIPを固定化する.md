---
title: '[備忘録][Azure] AppServiceのアウトバウンドグローバルIPを固定化する'
tags:
  - Azure
  - natgateway
  - AppService
private: false
updated_at: '2022-12-02T18:50:18+09:00'
id: 13563a577ec6986ab709
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

サポートの方にきいてみた

> Web App からのアウトバウンド通信の送信元 IP を固定化する方法について
> NATゲートウェイを使用する以外に方法があるのでしょうか？

# 公式のおすすめ

NAT Gatewayを利用してアウトバウンドIPを固定化

[Azure公式ドキュメント - 静的送信 IP を取得する](https://learn.microsoft.com/ja-jp/azure/app-service/overview-inbound-outbound-ips#get-a-static-outbound-ip)

>仮想ネットワーク NAT ゲートウェイを使用して、静的パブリック IP アドレス経由でトラフィックを送信すると、アプリからの送信トラフィックの IP アドレスを制御できます。 リージョン VNet 統合は、Standard、Premium、PremiumV2 および PremiumV3 App Service プランのみで使用できます。 このセットアップの詳細については、NAT ゲートウェイの統合に関するページを参照してください。
 
## やり方を適当に説明

用意するリソース

- AppServiceプラン(VNet統合できる、Standard、Premium、PremiumV2 および PremiumV3のいずれか)
    - AppService
- NAT Gateway
- PublicIP
- VNet
    - 1つのサブネット

1. NAT GatewayにPublicIPを紐づける
2. サブネットにNAT Gatewayを紐づける
3. AppServiceをVNet統合する
    - 用意したサブネットに紐づける
    - この時、必ず「ルート全て」設定をONにする(参考: [アプリケーション ルーティング](https://learn.microsoft.com/ja-jp/azure/app-service/overview-vnet-integration#application-routing))
        - じゃないとアウトバウンド通信全てがSNATされず、IP固定化できない通信が発生する

# 一応他の方法もあるらしい

理論上は NAT Gateway の代わりに NVA (仮想ネットワークアプライアンス) で SNAT を実施することで、送信元 IP を固定できるらしい。
(構成例)
App Service ---> VNet ---> NVA ---> Internet

現時点(2022/12)で上記構成は公開されている情報はなく、Azureサポートもおすすめはしてないみたい。
「特別な理由がない限りは、NAT Gateway と組み合わせて送信元 IP を固定化する方法をご検討いただければと存じます。」ですって。

公式で明言されていない以上、動作確認や性能テストは自分達で全て担保する必要があるので、やるメリットは少なそう。

# NAT Gatewayを使うデメリット

## ゾーン冗長化できない

NAT Gatewayはゾーン冗長に対応しておらず、特定アベイラビリティゾーンへのデプロイとなってしまう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/c83d0996-8720-863f-10a4-392f1078c179.png)

しかし、サブネットに紐づけられるNAT Gatewayは1つだけ。
つまり、NAT Gatewayをデプロイしたアベイラビリティゾーンで障害が発生した場合、AppServiceプランがゾーン冗長化されていようとも、もろとも影響を受けてしまう。

フィードバックも上がってるみたいなので、いずれ改善されるかもしれません。望み薄。

https://feedback.azure.com/d365community/idea/3c5a1b30-8b26-ec11-b6e6-000d3a4f0789

ちなみにPublicIPも冗長化に対応してる。あとはNAT Gatewayだけ。
