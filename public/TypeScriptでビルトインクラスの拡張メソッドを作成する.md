---
title: TypeScriptでビルトインクラスの拡張メソッドを作成する
tags:
  - TypeScript
private: false
updated_at: '2022-12-14T18:33:38+09:00'
id: 31c8a1ef33b871a7a6ab
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

共通的に使える関数ではなく、ある特定の型に拡張してメソッドを生やしたい時ってありますよね。
JavaScriptのやり方はググったらちょくちょく出てきたんですけど、TypeScriptのナレッジがあまり見受けられなかったので備忘録を兼ねて。

## バージョン

```
Node 18.12.1
npm 8.19.2
"typescript": "^4.9.3"
```

# 実装

```StringExtension.ts
/**
 * このexportがないとglobalスコープを拡張できない
 * See https://www.web-dev-qa-db-ja.com/ja/typescript/%E3%82%B0%E3%83%AD%E3%83%BC%E3%83%90%E3%83%AB%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97%E3%81%AE%E6%8B%A1%E5%BC%B5%E3%81%AF%E3%80%81%E5%A4%96%E9%83%A8%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A2%E3%83%B3%E3%83%93%E3%82%A8%E3%83%B3%E3%83%88%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E5%AE%A3%E8%A8%80%E3%81%A7%E3%81%AE%E3%81%BF%E7%9B%B4%E6%8E%A5%E3%83%8D%E3%82%B9%E3%83%88%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%EF%BC%882669%EF%BC%89/810779808/
 */
export {}

// Stringクラスにはこんなメソッドが定義されていますよ、とコンパイラ? インタプリタ？　に教えてあげないといけない
declare global {
  interface String {
    /**
     * JavaやC#のformatメソッドのように、特定の文字列のプレースホルダを引数で渡された文字で置き換える
     *
     * 参考
     * - https://stackoverflow.com/questions/610406/javascript-equivalent-to-printf-string-format/4673436#4673436
     * - https://trueman-developer.blogspot.com/2015/11/typescriptjavascript.html
     */
    format(...args: unknown[]): string
  }
}

String.prototype.format = function (...args: unknown[]) {
  let str = this.toString()
  for (const [i, arg] of args.entries()) {
    const regExp = new RegExp(`\\{${i}\\}`, 'g')
    str = str.replace(regExp, arg as string)
  }
  return str
}
```

## 使い方

```index.ts
// 必ずimportしてあげる
import './StringExtension'

'Hello {0}'.format('World!')
```

# 参考

https://www.web-dev-qa-db-ja.com/ja/typescript/%E3%82%B0%E3%83%AD%E3%83%BC%E3%83%90%E3%83%AB%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97%E3%81%AE%E6%8B%A1%E5%BC%B5%E3%81%AF%E3%80%81%E5%A4%96%E9%83%A8%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A2%E3%83%B3%E3%83%93%E3%82%A8%E3%83%B3%E3%83%88%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E5%AE%A3%E8%A8%80%E3%81%A7%E3%81%AE%E3%81%BF%E7%9B%B4%E6%8E%A5%E3%83%8D%E3%82%B9%E3%83%88%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%EF%BC%882669%EF%BC%89/810779808/
