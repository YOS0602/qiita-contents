---
title: '[Node.js] Expressサーバーのタイムアウト時間を変更する'
tags:
  - JavaScript
  - Node.js
  - Express
private: false
updated_at: '2023-01-06T22:58:43+09:00'
id: 5a9d23037d04aacf8db6
organization_url_name: null
slide: false
ignorePublish: false
---
2023/01/06追記
この記事のやり方だとレスポンスが返らないので、その対策をした記事を書きました

https://qiita.com/YOS0602/items/87f8ac88f9a008029e2f

# デフォルト

Expressでは、リクエストのタイムアウトデフォルト値が**2分**に設定されています。

# タイムアウト時間を設定

以下のように、オプションとして`timeout`が用意されています。そちらの値をミリ秒で指定することで、タイムアウト時間を変更することができます。

```app.js
const express = require("express")
const app = express()
...
const appServer = app.listen(80, () => { })
appServer.timeout = 1000 * 60 * 5 // 5min
```

# 参考

※2022/5/19追記：見れなくなっちゃってますね
[Node.js サーバーのタイムアウトの時間を変更する](http://backham.me/blog/2019/10/24/node-js-%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%81%AE%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%A2%E3%82%A6%E3%83%88%E3%81%AE%E6%99%82%E9%96%93%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B/)
