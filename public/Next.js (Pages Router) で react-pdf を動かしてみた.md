---
title: Next.js (Pages Router) で react-pdf を動かしてみた
tags:
  - PDF
  - 検証
  - React
  - Next.js
  - react-pdf
private: false
updated_at: '2024-09-27T23:40:56+09:00'
id: 20ccf9473157da50f9b0
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

今回試したソースコードはこちらで確認いただけます。

https://github.com/YOS0602/react-pdf-with-next-pages-router

## やってみたこと

正しい表現か自信はありませんが、ニュアンスを感じ取っていただければ幸いです。

- クライアントサイドでPDFをレンダリングする(CSR)
- サーバーサイドでPDFを生成して、クライアントサイドからは埋め込みで閲覧する(SSR)
- API RouteのレスポンスとしてPDFファイルを表示する

今回検証に使用したツール類のバージョンはこちらです。

| Name | Version |
| --- | --- |
| M2 MacBook Air 2022 | 14.6.1 |
| Google Chrome | 129.0.6668.70 |
| Node.js | 20.11.0 |
| npm | 10.2.4 |
| @react-pdf/renderer | 4.0.0 |
| next | 14.2.13 |
| react | 18.3.1 |
| typescript | 5.6.2 |

# 環境構築

まずは Next.js のテンプレートからリポジトリを作成します。

- Biomeを使ってみたかったのでESLintはNo
- 簡単にお試しするだけなので、UtilityFirstなTailwindを選択
- AppRouterは今回使いませんのでNo
- その他は適当です

```bash
$ npx create-next-app@latest
Need to install the following packages:
create-next-app@14.2.13
Ok to proceed? (y) y
✔ What is your project named? … react-pdf-with-next-pages-router
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … No
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … No
✔ Would you like to customize the default import alias (@/*)? … Yes
✔ What import alias would you like configured? … @/*
Creating a new Next.js app in /Users/user/workspace/react-pdf-with-next-pages-router.
```

<details><summary>依存ライブラリを最新化しておく</summary>

```bash
npm i react@latest react-dom@latest next@latest typescript@latest \
  @types/node@latest @types/react@latest @types/react-dom@latest \
  postcss@latest tailwindcss@latest
```

`package.json` ↓

```json
  "dependencies": {
    "next": "^14.2.13",
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@biomejs/biome": "1.9.2",
    "@types/node": "^22.6.1",
    "@types/react": "^18.3.9",
    "@types/react-dom": "^18.3.0",
    "postcss": "^8.4.47",
    "tailwindcss": "^3.4.13",
    "typescript": "^5.6.2"
  }
```

</details>

## Biome導入

検証ついでに以前から気になっていたBiomeを使ってみます。ESLint、Prettier と高い互換性を持つツールチェーンです。

https://biomejs.dev/ja/guides/getting-started/

https://biomejs.dev/ja/guides/integrate-in-editor/

https://biomejs.dev/ja/reference/vscode/

ライブラリを導入してinitします。

```bash
npm install --save-dev --save-exact @biomejs/biome
npx @biomejs/biome init
```

`package.json` 内のscriptsを編集します。

```diff
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
-   "lint": "next lint",
+   "lint": "biome lint",
+   "format": "biome format",
+   "check": "biome check --write"
  },
```

Cursor(VSCode)拡張機能のインストール

```bash
# VSCode
code --install-extension biomejs.biome
# Cursor
cursor --install-extension biomejs.biome
```

`.vscode/settings.json`

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  }
}
```

まだコード量が少ないので、速度に関してはESLint+Prettierとの差を体感することはできませんでした。ただ、使い勝手は全然変わらないですね。乗り換えのハードルはかなり低そうです。

## react-pdf

https://react-pdf.org/

```bash
npm install @react-pdf/renderer --save
```

# PDFを出力してみる

## PDFDocumentコンポーネントを作っておく

`src/components/PDFDocument.tsx`

```tsx
import { Document, Page, Text, View } from "@react-pdf/renderer";
import { StyleSheet } from "@react-pdf/renderer";

// Create styles
export const styles = StyleSheet.create({
  page: {
    fontFamily: "HackGen35",
    flexDirection: "row",
    backgroundColor: "#E4E4E4",
  },
  section: {
    margin: 10,
    padding: 10,
    flexGrow: 1,
  },
  boldText: {
    fontWeight: "bold",
  },
});

/**
 * Create Document Component
 */
const PDFDocument = () => (
  <Document>
    <Page size="A4" style={styles.page}>
      <View style={styles.section}>
        <Text>Section #1</Text>
        <Text>こんにちは</Text>
      </View>
      <View style={styles.section}>
        <Text>Section #2</Text>
        <Text style={styles.boldText}>太字です</Text>
      </View>
    </Page>
  </Document>
);

export default PDFDocument;
```

## 日本語が文字化けすることへの対処

Fontを登録してあげないと、日本語文字が文字化けする可能性があります。
[公式ドキュメントでは](https://react-pdf.org/fonts) TTFフォントファイルのみサポートしている記述がありますが、[Issue Comment](https://github.com/diegomura/react-pdf/issues/334#issuecomment-2164985108)を見る限りWOFFファイルも対応しているようです。
今回はプログラミング向けフォントとして公開されている [白源 (はくげん／HackGen)](https://github.com/yuru7/HackGen?tab=readme-ov-file) を使ってみました。

なお、 `public` フォルダ内の静的Assetにアクセスするパスが処理が動く環境によって異なるようで、PDFレンダリング処理ごとに設定を呼び分けています。（詳細は各コードを参照）

## リンクボタンを作っておく（任意）

`src/pages/index.tsx` に以下を追加しておきます。これでTopページから各コンポーネントにアクセスしやすくなります

```tsx
<div className="gap-2 flex">
  <Link href={"/client"}>
    <Button>Client</Button>
  </Link>
  <Link href={"/server"}>
    <Button>Server</Button>
  </Link>
  <Link href={"/api/pdf"}>
    <Button>/api/pdf</Button>
  </Link>
</div>
```

## Client Side でPDFをレンダリング

`src/pages/client/index.tsx` 

```tsx
"use client";

import PDFDocument from "@/components/PDFDocument";
import { Font } from "@react-pdf/renderer";
import dynamic from "next/dynamic";

Font.register({
  family: "HackGen35",
  fonts: [
    {
      src: "/fonts/HackGen35-Regular.ttf",
      fontStyle: "normal",
      fontWeight: "normal",
    },
    {
      src: "/fonts/HackGen35-Bold.ttf",
      fontStyle: "normal",
      fontWeight: "bold",
    },
  ],
});

const DynamicPDFViewer = dynamic(
  () => import("@react-pdf/renderer").then((mod) => mod.PDFViewer),
  {
    loading: () => <p>Loading...</p>,
    ssr: false,
  }
);

/**
 * Client Side でPDFをレンダリングする
 */
const Page = () => {
  return (
    <>
      <p>クライアントサイドで作成したPDFを表示しています。</p>
      <DynamicPDFViewer className="mx-auto" width={1200} height={1000}>
        <PDFDocument />
      </DynamicPDFViewer>
    </>
  );
};

export default Page;
```

[http://localhost:3000/client](http://localhost:3000/client) にアクセスするとPDFがレンダリングされて表示されます。
![ClientSide.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/92031cd5-c9df-7d38-d99b-4143f8cfe9e0.png)

`Lazy Loading` ([Doc](https://nextjs.org/docs/pages/building-your-application/optimizing/lazy-loading))の`ssr` optionをfalseとすることで、サーバーでのレンダリングを抑制しフロントで処理させています。

:::note warn
`@react-pdf/renderer` からexportされている `PDFViewer` コンポーネントは内部的にWebAPIを使用しているようで、Node.js環境で実行するとエラーが発生します。
:::

## Server Side でPDFを生成

`src/server/pdf-font.ts`

```ts
import path from "node:path";
import { Font } from "@react-pdf/renderer";

/**
 * サーバーサイドからpublicフォルダ内の静的ファイルにアクセスするための絶対パスを取得する
 */
export const getPublicAssetPath = (fileName: string): string =>
  path.resolve("./public", fileName);

Font.register({
  family: "HackGen35",
  fonts: [
    {
      src: getPublicAssetPath("fonts/HackGen35-Regular.ttf"),
      fontStyle: "normal",
      fontWeight: "normal",
    },
    {
      src: getPublicAssetPath("fonts/HackGen35-Bold.ttf"),
      fontStyle: "normal",
      fontWeight: "bold",
    },
  ],
});
```

`src/server/makePDF.tsx`

```tsx
import "@/server/pdf-font";
import PDFDocument from "@/components/PDFDocument";
import { getPublicAssetPath } from "@/server/pdf-font";
import * as ReactPDF from "@react-pdf/renderer";

/**
 * サーバーサイドでPDFを生成し、publicフォルダにファイルとして出力する
 */
export const makePDF = async () => {
  const pdfFileName = "generated-in-server.pdf";
  await ReactPDF.renderToFile(<PDFDocument />, getPublicAssetPath(pdfFileName));
  return { fileName: pdfFileName, url: `/${pdfFileName}` };
};
```

`src/pages/server/index.tsx`

```tsx
import { makePDF } from "@/server/makePDF";
import type { GetServerSideProps, InferGetServerSidePropsType } from "next";

const Page = ({
  fileName,
  fileURL,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  return (
    <>
      <p>サーバーサイドで作成したPDFを表示しています。</p>
      <object
        className="mx-auto"
        title={fileName}
        data={fileURL}
        type="application/pdf"
        width={1000}
        height={1200}
      />
    </>
  );
};

export const getServerSideProps = (async () => {
  const { fileName, url } = await makePDF();
  return { props: { fileName, fileURL: url } };
}) satisfies GetServerSideProps<{ fileName: string; fileURL: string }>;

export default Page;
```

[http://localhost:3000/server](http://localhost:3000/server) にアクセスするとPDFがレンダリングされて表示されます。clientの時と見た目はほぼ一緒ですが、PDFファイルの生成がサーバーサイドで行われている点が異なります。
![ServerSide.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/992a4fc7-bebd-01e7-bb0c-35b11cca9c45.png)

:::note info
`getServerSideProps` からは `JSON serializable data types` しかreturnできないっぽいので、ファイルのStreamやBufferをコンポーネントpropsに渡すことは出来なさそうでした。
:::

## API RouteのレスポンスとしてPDFファイルを表示する

`src/pages/api/pdf.tsx`

```tsx
import PDFDocument from "@/components/PDFDocument";
import ReactPDF from "@react-pdf/renderer";
import type { NextApiRequest, NextApiResponse } from "next";
import "@/server/pdf-font";

export default async function handler(
  _req: NextApiRequest,
  res: NextApiResponse<never>
) {
  const stream = await ReactPDF.renderToStream(<PDFDocument />);
  res.setHeader("Content-Type", "application/pdf");
  res.setHeader(
    "Content-Disposition",
    'inline; filename="react-pdf-sample.pdf"'
  );
  stream
    .pipe(res)
    .on("end", () => console.log("Done streaming, response sent."));
}
```

Streamを使用してレスポンスします。
[http://localhost:3000/api/pdf](http://localhost:3000/api/pdf) にアクセスすると、サーバーで生成されたPDFがフルスクリーンで表示されます。
![APIRoute.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/e7e991cd-3ce2-6e7a-bc04-3e6b60b704fa.png)

# おわりに

クライアントサイド、サーバーサイドを選ばずPDF生成できるのでかなり便利だなと思いました。
React-pdf が公開するPDFのComponentsやStylingに関しては検証できていないので、また時間がある時に触ってみようと思います。

# 参考

https://medium.com/@boris.poehland.business/next-js-api-routes-how-to-read-files-from-directory-compatible-with-vercel-5fb5837694b9

https://github.com/yuru7/HackGen?tab=readme-ov-file

https://zenn.dev/ksk2/articles/c0cf38b8ba61ec
