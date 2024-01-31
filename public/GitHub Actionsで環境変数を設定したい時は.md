---
title: GitHub Actionsで環境変数を設定したい時は
tags:
  - GitHub
  - GitHubActions
private: false
updated_at: '2022-06-06T10:27:22+09:00'
id: 3e23cc434acc76e4dd81
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

CICDなどで利用するGitHub Actions。
その中で、ユニットテストで接続するDBのURLや、パラメータ値などを設定したいというニーズが出てくることがあると思います。

Actionsが実行されている環境は、ユーザが管理するところではなく、直接環境変数を設定することができないので、どうやったらActionsの中で環境変数として動的に値を変えられるかを記載します。

# 手順

## Secretsの設定

1. リポジトリの[Settings]
1. [Secrets]-[Actions]
1. [New repository secret]
1. Name, Valueを入力してAdd

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/988cc568-078a-55f2-61e7-89fc8bca2efb.png)

## yml上で指定

`yml`上の`env`タグで指定することで、指定したsecretsが環境変数としてPGから読めるようになります。
こちらは、`yml`上のどのjobやstepからでも参照できます。


```.github/workflows/ci.yml
name: Node.js CI

# See: https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  DATABASE_URL: ${{secrets.TEST_DATABASE_URL}}
  SHADOW_DATABASE_URL: ${{secrets.SHADOW_DATABASE_URL}}

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm install
      - run: npm run lint
      - run: npm run build --if-present
      - run: npm test
```

### 特定のjobやstepでのみ使えるようにする

参考URLを見てもらえれば分かりますが、ある特定の箇所でのみ環境変数を有効にすることができます。
によって値を変えたいときに使えますね。

```yml
env:
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        if: ${{ env.DAY_OF_WEEK == 'Monday' }}
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
        env:
          First_Name: Mona
```


# 参考

- [GitHub Actions - 環境変数](https://docs.github.com/ja/actions/learn-github-actions/environment-variables)
