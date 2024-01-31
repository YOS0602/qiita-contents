---
title: あれ？ Voltaを使ってるんだけど「node」にPATHが通ってないぞ？
tags:
  - Node.js
  - volta
private: false
updated_at: '2022-10-02T08:50:55+09:00'
id: c694780773b091b4218c
organization_url_name: null
slide: false
ignorePublish: false
---
毎回忘れててググっているのでメモ

# 環境

MacOS 12.5.1
Volta 1.0.8

# 事象

`node`コマンドが通らない

```terminal
% node -v
zsh: command not found: node
```

# 解決策

PATHを通す

```~/.zshrc
︙
export VOLTA_HOME="$HOME/.volta"
export PATH="$VOLTA_HOME/bin:$PATH"
```

# 参考

https://zenn.dev/taichifukumoto/articles/how-to-use-volta#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%95%E3%82%8C%E3%81%9F%E3%81%93%E3%81%A8%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B
