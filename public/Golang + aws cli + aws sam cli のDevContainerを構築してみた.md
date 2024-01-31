---
title: Golang + aws cli + aws sam cli のDevContainerを構築してみた
tags:
  - Go
  - aws-cli
  - sam-cli
  - devcontainer
private: false
updated_at: '2023-10-01T15:15:03+09:00'
id: b2e71280575ce493e7a9
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

やろうとおもったきっかけ

https://www.udemy.com/course/bitcoin-lambda-golang/

このUdemy講座を見ていて、Golang、aws cli、aws sam cli をインストールするセクションがあります。
講座ではシンプルにするためにローカル環境にhomebrewでインストールしてましたが、環境を汚したくなかったので、DevContainerとして構築できないかとやってみました。

:::note warn
筆者はGolangもAWSもコンテナも初学者です。
記載する内容が正しい保証は一切なく、ベストプラクティスでもありません。
至らぬ点がありましたらコメントで教えていただけますと幸いです。
:::

# 環境

- macOS 13.6
- Docker 24.0.2-rd
- VSCode 1.82.2

# 各種ファイルの用意

作成したファイルは下記3つです。
分けずに作れそうだなあとも思いつつ、そこは放置。。。

1. `Dockerfile`
2. `docker-compose.yml`
3. `devcontainer.json`

## .devcontainer/Dockerfile

- Golangイメージで下記cliを使えるようにしたコンテナを作成する
    - aws cli
    - aws sam cli

```.devcontainer/Dockerfile
FROM golang:1.21-bullseye

# Clean up old resources
RUN rm -Rf /workspace/aws 

RUN mkdir -p /workspace/aws
WORKDIR /workspace/aws

# Install dependencies
RUN apt-get update -y && apt-get install -y zip unzip sudo pip

# Install aws cli
# See https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN sudo ./aws/install --update

# Install aws sam cli
# See https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/install-sam-cli.html
RUN pip install --upgrade aws-sam-cli
```

## docker-compose.yml

- bindするvolumeを指定

```.devcontainer/docker-compose.yml
version: '3'
services:
  golang:
    build: .
    image: golang-and-aws-cli
    container_name: 'golang-bitcoin-lambda'
    ports:
      - 8080:80
    volumes:
      - type: bind
        source: '..'
        target: '/workspace/app'
      # 認証情報をバインドしておく
      # See https://qiita.com/yh1224/items/80a652562291cf37c718
      - type: bind
        source: '~/.aws'
        target: '/root/.aws'
        read_only: true
    tty: true
    restart: always
    # enable the tini init process to handle signals and clean up Zombie processes if you do not have an alternative set up
    # See https://github.com/microsoft/vscode-dev-containers/blob/main/script-library/docs/docker-in-docker.md#feature-use
    init: true
```

## devcontainer.json

```.devcontainer/devcontainer.json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "golang",
  "workspaceFolder": "/workspace/app",
  "features": {
    "docker-in-docker": {
      "version": "latest"
    }
  },
  "customizations": {
    "vscode": {
      "extensions": ["golang.go", "amazonwebservices.aws-toolkit-vscode"]
    }
  }
}
```

# VSCodeで開発コンテナーを立ててみる

必要なものが全部揃ってますね。

```bash
root:/workspace/app# go version
go version go1.21.1 linux/arm64

root:/workspace/app# aws --version
aws-cli/2.13.22 Python/3.11.5 Linux/6.1.30-0-virt exe/aarch64.debian.11 prompt/off

root:/workspace/app# sam --version
SAM CLI, version 1.97.0

root:/workspace/app# docker --version
Docker version 23.0.6+azure-2, build ef23cbc4315ae76c744e02d687c09548ede461bd
```

# 参考

https://qiita.com/yh1224/items/80a652562291cf37c718

https://qiita.com/frozenbonito/items/b1de3980ee0553fb1c2f
