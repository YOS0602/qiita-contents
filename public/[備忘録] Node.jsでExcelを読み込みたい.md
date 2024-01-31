---
title: '[備忘録] Node.jsでExcelを読み込みたい'
tags:
  - Node.js
  - Excel
  - SheetJS
private: false
updated_at: '2023-01-07T02:05:35+09:00'
id: 97a224eb1e996e017727
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

自動化ツールの一環で、Node.jsを使ってExcelを読み込みたかった
ちょこっとSheetJSをさわったのでメモ

## おことわり

- この記事ではエクセルの内容を読むだけで、セルに書き込んだりブックを新しく作ったりしません
- せっかくなのでTypeScriptで書いてみました
    - `ts-node`使ったり、いちいちjsにトランスパイルしたりしないと実行できないので、たぶんcommonJSか`.mjs`拡張子で作って`node`で起動するのがツール利用観点では楽かも

# 環境

```
macOS 12.6
node 18.12.1
SheetJS(xlsx) 0.19.1
typescript 4.9.4
@types/node 18.11.18
ts-node 10.9.1
```

`npm i -D typescript ts-node @types/node`

# 使用ライブラリ

[SheetJS](https://docs.sheetjs.com/)
npmでのパッケージ名は`xlsx`

[NodeJSでのInstallationページ](https://docs.sheetjs.com/docs/getting-started/installation/nodejs#installation)に下記のように記載がある通り、npmで配布されているのはバージョン`0.18.5`で止まっているそうな

:::note alert
Older releases are technically available on the public npm registry as xlsx, but the registry is out of date. The latest version on that registry is 0.18.5
This is a known registry bug
:::

公式曰く、npmからinstallしたパッケージはuninstallして、cdnで配布されている新しいのを入れ直してくれだとよ

# Install

`npm i https://cdn.sheetjs.com/xlsx-0.19.1/xlsx-0.19.1.tgz`
`yarn add https://cdn.sheetjs.com/xlsx-0.19.1/xlsx-0.19.1.tgz`

# データを読んでみるExcelを適当に作る

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/13b3bd21-1be6-1267-ceee-fdc6a2e542f8.png)

# ファイルの読み込み

```index.ts
import os from 'node:os';
import path from 'node:path';
import * as XLSX from 'xlsx';

const filepath = path.join(os.homedir(), '/Desktop/prices.xlsx');
const wb = XLSX.readFile(filepath, {
  type: 'array', // Input data encoding bufferやbase64なども選択可能
  sheets: ['Sheet1'], // 読み込むシート
  cellHTML: false, // rich textをHTMLとして読み込むか フロントエンドで表示したい時は使えそう
});
const ws = wb.Sheets['Sheet1'];
```

## もし`mjs`で作ってるなら

[公式Docs](https://docs.sheetjs.com/docs/getting-started/installation/nodejs#usage)に言及がある通り、native node modulesをloadできないのでsetしてあげる必要がある
Docsは`import * as XLSX from 'xlsx/xlsx.mjs';`って書いてあるけど、`/xlsx.mjs`つけるとうまく動かなかった

```js:index.mjs
import * as XLSX from 'xlsx'
import fs from 'node:fs'
/* load 'fs' for readFile and writeFile support
See https://docs.sheetjs.com/docs/getting-started/installation/nodejs#esm-import */
XLSX.set_fs(fs)
```

## `ws`の中身は...

```ts
{
  '!ref': 'B2:E8',
  B2: {
    t: 's',
    v: 'products',
    r: '<t>products</t><phoneticPr fontId="1"/>',
    w: 'products'
  },
/////// 略 ///////
  B8: {
    t: 's',
    v: 'chair',
    r: '<t>chair</t><phoneticPr fontId="1"/>',
    w: 'chair'
  },
  C8: { t: 'n', v: 6, w: '6' },
  D8: { t: 'n', v: 3840, w: '3840' },
  E8: { t: 'n', v: 23040, f: 'C8*D8', w: '23040' }
}
```

- 空白セルは勝手に無視してくれる
- `t`とか`w`とか
    - 参照: [Parsing Options](https://docs.sheetjs.com/docs/api/parse-options/)

| key | description |
|:-:|:-|
| t | The Excel data type for a cell.<br>b Boolean, n Number, e error, s String, d Date, z Stub |
| v | The raw value of the cell.  Can be omitted if a formula is specified |
| f | Cell formula (if applicable)<br>`cellFormula`optionで設定可能 |
| F | Range of enclosing array if formula is array formula (if applicable) |
| w | Formatted text (if applicable)<br>`cellText`optionで設定可能 |
| h | HTML rendering of the rich text (if applicable)<br>`cellHTML`optionで設定可能 |
| z | Number format string associated with the cell<br>`cellNF`optionで設定可能 |
| s | style/theme<br>`cellStyles`optionで設定可能 |
| c | Comments associated with the cell |
| l | Cell hyperlink object |

# Excel上のデータを読み込んで何かする

合計金額でも出してみましょうか

```index.ts
const sums: number[] = [];
for (const [key, value] of Object.entries(ws)) {
  if (key.match(/E\d+/) && typeof value.v === 'number') sums.push(value.v);
}

const subtotal = sums.reduce((p, c) => p + c);
console.log({ subtotal }); // -> 44920
```

# さいごに

この辺を参照すれば書き込みも問題なくできそう

https://docs.sheetjs.com/docs/api/write-options/

https://docs.sheetjs.com/docs/getting-started/example#run-the-demo-locally
