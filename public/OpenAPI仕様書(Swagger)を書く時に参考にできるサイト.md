---
title: OpenAPI仕様書(Swagger)を書く時に参考にできるサイト
tags:
  - swagger
  - OpenAPI
private: false
updated_at: '2022-09-05T18:37:51+09:00'
id: 0b5dfcb14559d25b3608
organization_url_name: null
slide: false
ignorePublish: false
---
# そもそもopenAPI仕様書ってこんなものってのを理解する

- [OpenAPI (Swagger) 超入門 - Qiita](https://qiita.com/teinen_qiita/items/e440ca7b1b52ec918f1b)

# Swagger公式

サンプル見たりするのに使うかな。
正直見にくい。

- [OpenAPI Guide](https://swagger.io/docs/specification/basic-structure/)

# GitHubのドキュメント

これが一番使うかな。
`yaml`や`json`は階層構造になってて、どんどん深掘りしていける。
`components`配下の`parameters`オブジェクトで指定できる項目は？
`in`に指定できるコードは？
など検索できるのが良い。

- [OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md)

# HTTPステータスコード一覧

APIですから、ステータスコードに意味をしっかり持たせたいですよね。
- [HTTP レスポンスステータスコード](https://developer.mozilla.org/ja/docs/Web/HTTP/Status)

# 個人的に参考にしたサイト

`components`配下の書き方、`$ref`での汎用性の持たせ方など、大変参考になりました。

- [OpenAPI Specification 3.0.3を使ったいい感じのCRUD APIのサンプルを作る](https://www.blog.danishi.net/2022/01/22/post-5758/)

# SwaggerとOpenAPI

`openapi: 3.0.3`
`swagger: 2.0`
どっちを書くかで思ったよりフォーマットが違ってくる。
なんか最近は`openapi`にしといた方が良さげなんかなあ。
