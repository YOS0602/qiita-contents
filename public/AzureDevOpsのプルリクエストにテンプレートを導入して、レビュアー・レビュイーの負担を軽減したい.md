---
title: AzureDevOpsのプルリクエストにテンプレートを導入して、レビュアー・レビュイーの負担を軽減したい
tags:
  - pullrequest
  - AzureDevOps
  - AzureRepos
private: false
updated_at: '2023-02-27T14:59:52+09:00'
id: a964e9cca91174f33832
organization_url_name: null
slide: false
ignorePublish: false
---
# 経緯

Descriptionに何も書かれていないPRのレビューを依頼されると、

- 変更点は何か
- どこを変更したのか
- 何を確認しなきゃいけないか

などを全てレビュアーが判断したり、コミッターに確認せねばなりません。
チケットが紐づいていればまだ糸口を掴めますが、「バグの修正」みたいなアバウトなチケットだと結局何したんや? 状態です。

また、誤字脱字や、`// TODO`コメントの消し忘れが残っていることも結構あり、それに対する指摘数が増えてしまってレビュアー・レビュイー双方にとって負担になる課題もありました。

そこで、PRのテンプレートを使用し、説明の充足・セルフレビューを促すことによるうっかりミスの軽減に努めようと思いました。

# PRテンプレートを用意する

https://qiita.com/07JP27/items/103f8acc9810d917ecbb

https://learn.microsoft.com/ja-jp/azure/devops/repos/git/pull-request-templates?view=azure-devops

上記を参照することで、PRテンプレートを設定する方法を理解できました。

## 実際に作ってみたテンプレート

```pull_request_template.md
# チェックリスト

- [ ] セルフレビューはしましたか?
  - 誤字・脱字などの軽微なミスはここで減らしましょう
  - デバッグログ出力や、不要なコメントは残っていませんか?
  - 逆に不足しているコメントはありませんか?
- [ ] ユニットテストは作成しましたか?
  - 対応しない場合は、タスクチケットを作りましょう
  - 理由があってユニットテストを作成しない場合、PRのDescriptionに理由を記載してください
- [ ] PRのタイトルは変更点を端的に表していますか?
- [ ] 対応チケットはPRに紐づいていますか?
  - トレーサビリティ向上のために、チケット駆動開発を意識しましょう!

# 対応内容の概要

- 変更点やできるようになることを説明してください
- 端的に言うことが難しいなら、1つのPRサイズが大きすぎるかも

# レビュアーに対して伝えたいこと

- 申し送り事項や重点的に確認してほしいところを記載します

# 参考にしたサイトなど

- [サイト名](URL)
```

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/4da84506-1a2d-10dd-8ef9-719fdb5c125f.png" width="800">

実際にしばらく使ってみて、改善点があればまた反映しようと思います。

# 2023/02/27追記

マージする際にコミットメッセージを編集しないと、`git log`した際にとても見辛いですね。
いつも私は「チェックリスト」の部分は削除するようにしてます。
Auto Completeを設定する際に、コミットメッセージも編集するようにチームの意識づけが必要かなと思います。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e5fc4cde-9656-966b-ca0f-8776f955fe96.png" width="700">

# 参照

https://qiita.com/07JP27/items/103f8acc9810d917ecbb

https://learn.microsoft.com/ja-jp/azure/devops/repos/git/pull-request-templates?view=azure-devops

https://dev.classmethod.jp/articles/pull-request-template/

https://techblog.jmdc.co.jp/entry/2022/09/09/114809
