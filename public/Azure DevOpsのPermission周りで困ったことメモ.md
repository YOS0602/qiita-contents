---
title: Azure DevOpsのPermission周りで困ったことメモ
tags:
  - AzureDevOps
private: false
updated_at: '2023-04-19T12:22:11+09:00'
id: 3616cc9060184f549bd9
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

AzureDevOpsのPermission周りの設定、世界でいちばん難しい。
循環参照ありの階層構造になってて、テキトーにやってると人間の管理できる範囲を簡単に凌駕してくる。

今後のためにも、困ったこととその対応をメモします。

※随時追加予定です。

# Teamを作った時は

[Project Settings]-[Permissions]-[Group]tabに、Team名のgroupがあるはず。
それに全員追加しておく。

チームメンバー一律で設定できるので、これをやっておくと楽です。
Teamの中で権限を分けたい（例えば、会社Aの人と会社Bで権限を分けたり、新人さんだけは別にしたり）場合は、Groupをそれぞれ作り、TeamのPermission Group配下にメンバーとして登録すれば良いかと。

# Repositoryが見えないんだけど

[Project Settings]-[Repositories]-[Security]tab
該当するユーザもしくはGroupに、`Read`のPermissionをつける。

`Contribute`や`Create branch`できないと開発がままならないので、一緒につけてあげよう。

参考

https://blog.beachside.dev/entry/2022/02/22/190000

# Boardsのチケットが何も見えないんだけど

[Project Settings]-[Project configuration]-[Areas]tab

権限をつけたいAreaのSecurity設定に進み、`View work items in this node`をつけます。
`Edit work item comments in this node`や`Edit work items in this node`は最低限ないと、、、という感じですね。

参考

https://learn.microsoft.com/ja-jp/azure/devops/organizations/security/set-permissions-access-work-tracking?view=azure-devops#create-child-nodes-modify-work-items-under-an-area-or-iteration-path
