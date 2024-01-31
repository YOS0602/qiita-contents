---
title: 'position: fixed指定したら横幅短くなったし、要素が隠れてしまった時の対処法'
tags:
  - CSS
  - fixed
private: false
updated_at: '2022-10-02T19:38:28+09:00'
id: b7656eff931893767514
organization_url_name: null
slide: false
ignorePublish: false
---
# 前提

- M2 Mac 12.5.1
- Node.js 16.17.1
- Next.js 12.3.1
- tailwind 3.1.8
- GoogleChrome 105.0.5195.125

SPAとかSSRとか関係なく、HTML/CSSの話なので環境が違っても役立つ話かと思います
ただし、`tailwind`を使用しているので、CSSの書き方は読み替えお願いします

https://tailwindcss.com/docs/installation

# 理想

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/241f1136-0c46-328f-0d1e-f1354898e1f5.png)

# 現実

`fixed`を指定したあとは...

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/fad4ec69-bdff-94c6-fffb-66a6ce05cb11.png)

# やったこと

- ヘッダ部分に`position: fixed;`を追加

```diff_tsx
// navbar.tsx
import Link from 'next/link'

export const Navbar = (): JSX.Element => {
  return (
-    <header className="text-center bg-c1 text-c5 py-4">
+    <header className="fixed text-center bg-c1 text-c5 py-4">
      <div className="flex flex-wrap flex-row text-3xl justify-center">
        <Link href="#section-profile">
          <a className="px-3 hover:underline decoration-c2 hover:decoration-4">Profile</a>
        </Link>
// ...略...
      </div>
    </header>
  )
}
```

# 解決策

## 横幅が短くなってしまう

- `position: fixed;`を指定しているところに`width: 100%;`を追加

```diff_tsx
// navbar.tsx
import Link from 'next/link'

export const Navbar = (): JSX.Element => {
  return (
-    <header className="fixed text-center bg-c1 text-c5 py-4">
+    <header className="fixed w-full text-center bg-c1 text-c5 py-4">
      <div className="flex flex-wrap flex-row text-3xl justify-center">
        <Link href="#section-profile">
          <a className="px-3 hover:underline decoration-c2 hover:decoration-4">Profile</a>
        </Link>
// ...略...
      </div>
    </header>
  )
}
```

## 要素がヘッダーの下に隠れてしまう

- ヘッダーの高さ以上の十分な`padding`をとる

```diff_tsx
// index.tsx
// ...importは省略...

const Home: NextPageWithLayout = () => {
  return (
-    <div className="container text-center">
+    <div className="container text-center py-52 sm:py-40 md:py-20">
      <Head>
        <title>サンプル</title>
        <meta name="description" content="サンプル" />
        <link rel="icon" href="/favicon.ico" />
      </Head>
      <h1 id="section-profile" className="text-3xl">
        Profile
      </h1>
      <div className="py-10">なまえ: じゅげむじゅげむ</div>
    </div>
  )
}
```

# 参考

https://teratail.com/questions/175904
