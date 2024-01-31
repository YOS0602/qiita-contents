---
title: '[備忘録][TypeScript] ランダム文字列をちょっと生成したい時には'
tags:
  - TypeScript
private: false
updated_at: '2023-01-21T12:58:34+09:00'
id: b4bac0b63daeba828041
organization_url_name: null
slide: false
ignorePublish: false
---
# おことわり

ちょっとほしいな、くらいな気持ちの時にしか使わない前提です。
絶対に競合を避けたいのであれば、UUIDなどを使用してください。

# コード

```ts
const generateRandomString = (charCount = 7): string => {
  const str = Math.random().toString(36).substring(2).slice(-charCount)
  return str.length < charCount ? str + 'a'.repeat(charCount - str.length) : str
}
```

## 説明

`Math.random().toString(36).substring(2)`で生成した文字列が、引数`charCount`で指定した文字数を満たさない場合`a`で埋める。e.g. `u9bw7nr2a3caaaa`

## 実行例

`// {文字数} {生成された文字列}`

```ts
generateRandomString()
// 7 u85f9kc

generateRandomString(0)
// 11 3a2sgdmamx2 
// Math.random().toString(36).substring(2)が生成した文字列が返される

generateRandomString(1)
// 1 e

generateRandomString(20)
// 20 kq32frq5uslaaaaaaaaa
```

# ランダム部分は何文字になる？

`Math.random().toString(36).substring(2)`部分が生成する文字数をカウントするためのコードを用意。
100万回実行して、文字数の分布を調べる。

```ts
const obj: Record<string, number> = {}

for (let i = 0; i < 1_000_000; i++) {
  const rs = Math.random().toString(36).substring(2).length
  if (typeof obj[rs] === 'number') {
    obj[rs] += 1
  } else {
    obj[rs] = 1
  }
}

console.log(obj)
```

結果は以下の通り

```json
{
  "7": 6,
  "8": 193,
  "9": 7288,
  "10": 263166,
  "11": 704859,
  "12": 23804,
  "13": 668,
  "14": 14,
  "15": 2
} 
```

`a`が連続した文字列を絶対に避けたい場合、統計上、7文字を指定するのがbetter
