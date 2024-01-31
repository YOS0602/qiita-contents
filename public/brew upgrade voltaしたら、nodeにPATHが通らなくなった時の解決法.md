---
title: brew upgrade voltaしたら、nodeにPATHが通らなくなった時の解決法
tags:
  - Node.js
  - homebrew
  - npm
  - volta
private: false
updated_at: '2022-05-20T21:42:10+09:00'
id: 852130953e19cc18d551
organization_url_name: null
slide: false
ignorePublish: false
---
# 環境

- macOS 12.3.1
- Homebrew 3.4.11
- Volta 1.0.5 -> 1.0.7

# Volta

Voltaはクロスプラットフォームで使えるNodeとnpmのバージョン管理ツールです。かなり高速で、使い勝手も良いので、未導入の方は是非。

https://docs.volta.sh/guide/

# upgrade

voltaを最新にアップグレード

```terminal
$ brew upgrade volta
```

# command not found: node

nodeはおろかnpmも。焦る。

```terminal
$ node -v
zsh: command not found: node

$ npm -v
zsh: command not found: npm
```

けど、voltaだけはPATHが通っている状態

```terminal
$ volta -v
1.0.7
```

# 解決策

voltaをsetupすれば直りました。
もしかしたらupgradeした時にターミナルに出力されてたのかも。

```terminal
$ volta setup
success: Setup complete. Open a new terminal to start using Volta!
```

ターミナルを再起動したらnodeのPATH通った！

```terminal
$ node -v
v16.13.0
```
