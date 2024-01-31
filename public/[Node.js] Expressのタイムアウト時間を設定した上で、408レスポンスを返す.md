---
title: '[Node.js] Expressのタイムアウト時間を設定した上で、408レスポンスを返す'
tags:
  - Node.js
  - Express.js
private: false
updated_at: '2023-01-07T16:04:20+09:00'
id: 87f8ac88f9a008029e2f
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

https://qiita.com/YOS0602/items/5a9d23037d04aacf8db6

↑自分が過去に書いた記事では、`Express`でのタイムアウト時間設定を記載しました
しかし、この設定だけだとHTTPレスポンスが返らないので、ちゃんと相手がいるシステムだと困るなあと思い、レスポンスを返せるように実装してみました

## `curl`やPostmanでtimeout設定だけしたExpressサーバのAPIを叩いてみた

```terminal
% curl http://localhost:3000/time
curl: (52) Empty reply from server
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/bd4476bf-1f1c-1056-ed59-266120bd69ac.png)

レスポンスがクライアントに返っていないことがわかります
では、`408`ステータスが返却されるようにしてみましょう↓

# 実装

タイムアウトを設定し、`408`をレスポンスする関数を作成

```timeoutSeetting.ts
import { Response } from 'express'

const TIMEOUT_MS = 1000 * 60 // 60sec

const setResponseTimeOut = (res: Response) => {
  res.setTimeout(TIMEOUT_MS, () => {
    // 408 Request Timeoutの定義に従いConnectionをcloseする
    res.setHeader('Connection', 'close')
    res.sendStatus(408)
  })
}
```

expressのルーティング定義から作成した関数を最初にコールしておく

```app.ts
app.get('/time', async (req: Request, res: Response) => {
  setResponseTimeOut(res)

  // do something...

  // 既に408を返している場合はレスポンスしない
  if (!res.headersSent) res.sendStatus(200)
})
```

# 試してみる

めんどくさいので1ファイルにどかっと書きました

```app.ts
// タイムアウト時間は1秒と設定
const TIMEOUT_MS = 1000
const setResponseTimeOut = (res: Response, next: NextFunction) => {
  res.setTimeout(TIMEOUT_MS, () => {
    // 408 Request Timeoutの定義に従いConnectionをcloseする
    res.setHeader('Connection', 'close')
    console.log(`${TIMEOUT_MS}ms経過 Timeoutレスポンスを返却します`)
    console.timeEnd('/time')
    res.sendStatus(408)
  })
}

app.get('/time', async (req: Request, res: Response, next: NextFunction) => {
  console.time('/time')
  setResponseTimeOut(res, next)
  // 3秒かかる処理
  await new Promise<void>((resolve) => {
    setTimeout(() => {
      resolve()
    }, 3000)
  })
  await new Promise<void>((resolve) => {
    setTimeout(() => {
      console.log('408を返した後も処理は続くので注意!')
      resolve()
    }, 1000)
  })
  if (!res.headersSent) res.sendStatus(200)
})
// サーバリッスンなど省略
```

## APIを叩くと...

```terminal
1000ms経過 Timeoutレスポンスを返却します
/time: 1.003s
408を返した後も処理は続くので注意!
```

設定したタイムアウト時間である1秒経過後に`408`がレスポンスされています

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/bbd677d8-8681-0352-5a84-5c0523813734.png)

# 注意点

ターミナルにも出力していますが、`408`が返った後も処理自体は継続して動いているので注意が必要です
`408`を受け取ったクライアント側が`500`エラーなんかと同じように扱って、リトライをしまくると、冪等性のないAPIだった場合データのダブりがたくさん出ます
