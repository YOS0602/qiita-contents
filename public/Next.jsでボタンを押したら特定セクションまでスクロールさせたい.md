---
title: Next.jsでボタンを押したら特定セクションまでスクロールさせたい
tags:
  - Next.js
private: false
updated_at: '2022-10-02T19:51:31+09:00'
id: 9d8bf425f0d74cc4f81a
organization_url_name: null
slide: false
ignorePublish: false
---
# 環境

- M2 Mac 12.5.1
- Node.js 16.17.1
- Next.js 12.3.1
- tailwind 3.1.8
- GoogleChrome 105.0.5195.125

# やりたいこと

ボタンを押したら特定セクションまでスクロールさせたい
見てもらった方が早いので、こんな感じ

https://webdesigner-go.com/portfolio-template-basic/

Qiitaの目次って言った方が伝わるかな？

# やりかた

## 1.スクロール先となる箇所に`id`を振っておく

一意に特定できるならidじゃなくても良いと思う

```index.tsx
<h1 id="section-profile" className="text-3xl pt-40 md:pt-20">
  Profile
</h1>
<Profile />
```

## 2.`Link`コンポーネントの`href`に`id`セレクタを指定する

```index.tsx
import Link from 'next/link'

<Link href="#section-profile">
  <a className="px-3 hover:underline decoration-c2 hover:decoration-4">Profile</a>
</Link>
```

# 注意点

ナビゲーションバーなどを`position: fixed;`で指定していると、スクロールされた後に要素が隠れてしまうので、必要に応じて`padding`を設定してやる
上記1.のサンプルコードの`pt-40 md:pt-20`はそのために指定している

# 参考

https://stackoverflow.com/questions/68589788/nextjs-link-to-scroll-to-a-section-in-same-page
