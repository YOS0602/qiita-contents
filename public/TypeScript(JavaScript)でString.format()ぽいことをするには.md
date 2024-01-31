---
title: TypeScript(JavaScript)でString.format()ぽいことをするには
tags:
  - JavaScript
  - TypeScript
private: false
updated_at: '2023-01-01T17:30:13+09:00'
id: 8eadf8f7743ebdc5946c
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

コードだけさっさと知りたい方は[TypeScriptでの実装](#typescriptでの実装)までジャンプしてください

## 経緯

以下にあるように、C#やJava, Pythonには当たり前のように組み込まれている`String.format()`はTypeScriptには存在しない(らしい)
[テンプレートリテラル](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals)と呼ばれるものを使えば似たようなことはできるが、ログ出力なんかを考えると、`format()`みたいなのが使いたい時があるんですよね

```js
const name = 'taro', age = 21
console.log(`name: ${name}, age: ${age}`)
// -> name: taro, age: 21
```

https://qiita.com/yanchi4425/items/e24769f1ede8a983dca0

上記記事だと`String`クラスの`prototype`拡張としてメソッドを定義しているので、共通利用関数として使用できるように作ってみた

## C#の例

```csharp.cs
using System;
public class Hello{
    public static void Main(){
        Console.WriteLine(String.Format("name: {0}, age: {1}","taro", 21));
        // -> name: taro, age: 21
    }
}
```

## pythonの例

```python.py
print('name: {0}, age: {1}'.format('taro', 21))
# -> name: taro, age: 21
```

## Javaの例

```Java.java
public class Main {
    public static void main(String[] args) throws Exception {
        System.out.println(String.format("name: %s, age: %d", "taro", 21));
        // -> name: taro, age: 21
    }
}
```

# TypeScriptでの実装

```typescript.ts
/**
 * JavaやC#のformatメソッドのように、特定の文字列のプレースホルダを引数で渡された文字で置き換える
 *
 * @param str 置換前文字列 プレースホルダを`{0}`, `{1}`の形式で埋め込む
 * @param ...args 第2引数以降で、置換する文字列を指定する
 *
 * 参考
 * - https://stackoverflow.com/questions/610406/javascript-equivalent-to-printf-string-format/4673436#4673436
 * - https://trueman-developer.blogspot.com/2015/11/typescriptjavascript.html
 *
 * ### Sample
 * ```ts
 * format('{0}とは、{1}までに身に付けた{2}の{3}である。', '常識', '18歳', '偏見', 'コレクション')
 * format('{0}とは、{1}までに身に付けた{2}の{3}である。', ...['常識', '18歳', '偏見', 'コレクション'])
 * ```
 * →`'常識とは、18歳までに身に付けた偏見のコレクションである。'`
 */
export const format = (str: string, ...args: unknown[]): string => {
  for (const [i, arg] of args.entries()) {
    const regExp = new RegExp(`\\{${i}\\}`, 'g')
    str = str.replace(regExp, arg as string)
  }
  return str
}
```

## JavaScriptでも使える

型定義さえ外せば、JavaScriptとしても動かせる

```javascript.js
const format = (str, ...args) => {
  for (const [i, arg] of args.entries()) {
    const regExp = new RegExp(`\\{${i}\\}`, 'g')
    str = str.replace(regExp, arg)
  }
  return str
}
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/dfde10e5-36ee-2b44-0a36-605cb98b3179.png)

ChromeのConsoleタブで動かしてみた

# 簡単な使い方

```ts
format('{0}とは、{1}までに身に付けた{2}の{3}である。', '常識', '18歳', '偏見', 'コレクション')
// -> '常識とは、18歳までに身に付けた偏見のコレクションである。'
```

## 第1引数

文字列を渡す
置き換えたい箇所を`{0}`形式で文字列に埋め込む

## 第2引数以降

順番に置き換える文字列や数値を記述する

# `jest`でのテストコード

こちらから仕様を読み取ってもらえたら嬉しいです
`it.each`でやればよかった......

```format.spec.ts
it('format', () => {
    // stringではなくnumberを渡されても正常に動作する
    expect(format('age: {0}', 25)).toBe('age: 25')
    // 複数の置換があっても動作する
    expect(format('{0} {1} へのリクエストが失敗しました。', 'GET', '/users')).toBe(
      'GET /users へのリクエストが失敗しました。'
    )
    expect(
      format('{0}とは、{1}までに身に付けた{2}の{3}である。', '常識', '18歳', '偏見', 'コレクション')
    ).toBe('常識とは、18歳までに身に付けた偏見のコレクションである。')
    // null・undefinedチェック
    expect(format('null: {0}, undefined: {1}', null, undefined)).toBe(
      'null: null, undefined: undefined'
    )
    // ブランク文字列チェック
    expect(format('blank: {0}', '')).toBe('blank: ')
    // プレースホルダと置換文字列の数が一致していない場合の確認
    expect(format('{0} is dead, but {1} is alive! {0} {2}', 'ASP', 'ASP.NET')).toBe(
      'ASP is dead, but ASP.NET is alive! ASP {2}'
    )
    expect(format('{0}べきか、{1}べきか', '生きる', '死ぬ', '問題')).toBe(
      '生きるべきか、死ぬべきか'
    )
    // 序数の順序が逆
    expect(format('やはらかに 柳あをめる北上の {1} {0}', '泣けと如くに', '岸辺目に見ゆ')).toBe(
      'やはらかに 柳あをめる北上の 岸辺目に見ゆ 泣けと如くに'
    )
    // プレースホルダが複数ある場合
    expect(
      format(
        '{0}にて、アニメ「{1}×{1}」が{2}より毎週水曜午前1時59分（火曜深夜）から「Anichu」枠にて再放送することが決定いたしました',
        '日本テレビ',
        'HUNTER',
        '4月'
      )
    ).toBe(
      '日本テレビにて、アニメ「HUNTER×HUNTER」が4月より毎週水曜午前1時59分（火曜深夜）から「Anichu」枠にて再放送することが決定いたしました'
    )
    // 配列をスプレッド演算子で展開した場合
    expect(
      format(
        '{0}とは、{1}までに身に付けた{2}の{3}である。',
        ...['常識', '18歳', '偏見', 'コレクション']
      )
    ).toBe('常識とは、18歳までに身に付けた偏見のコレクションである。')
    // 置換文字を渡されなくても正常に動作する
    expect(format('生きるべきか、死ぬべきか')).toBe('生きるべきか、死ぬべきか')
    expect(format('生きるべきか、死ぬべきか', [])).toBe('生きるべきか、死ぬべきか')
    expect(format('生きるべきか、死ぬべきか', null)).toBe('生きるべきか、死ぬべきか')
    expect(format('生きるべきか、死ぬべきか', undefined)).toBe('生きるべきか、死ぬべきか')
})
```
