---
title: >-
  ReactRouterFramework v7 + MUI v6な環境で、ビルド時に発生する"Directory import ... is not
  supported resolving ES modules imported from..."エラーに対処する
tags:
  - react-router
  - MUI
  - vite
private: false
updated_at: '2025-05-08T18:22:42+09:00'
id: ec90322ae89d3a4c488b
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

`vite.config.ts` の `ssr.noExternal` に `["@mui/*"]` を追加する。
詳しくは [こちら](#対処法) へ。

:::note info
2025/05/08追記
MUI v7でESMサポートが改善されました。 `ssr.noExternal` に `["@mui/*"]` を追加しなくてもエラーが発生しないようになっていると思います。
https://mui.com/blog/material-ui-v7-is-here/#improved-esm-support
:::

# はじめに

[SSR機能を無効化](https://reactrouter.com/how-to/spa)した [React Router Framework v7](https://reactrouter.com/7.1.1/home) プロジェクトで [Material UI v6 (MUI)](https://mui.com/material-ui/getting-started/) を導入した際に発生したエラーについて、その原因と解決方法をメモしています。

## 問題の概要

`react-router dev` で開発サーバーは問題なく起動できるのに、 `react-router build` で下記エラーが発生する状況でした。

```txt
...
vite v5.4.11 building SSR bundle for production...
✓ 195 modules transformed.
build/server/.vite/manifest.json    0.17 kB
build/server/index.js             177.66 kB
x Build failed in 124ms
[react-router] Directory import '/path/to/project/sample-repo/node_modules/@mui/utils/formatMuiErrorMessage' is not supported resolving ES modules imported from /path/to/project/sample-repo/node_modules/@mui/material/styles/index.js
Did you mean to import "@mui/utils/formatMuiErrorMessage/index.js"?
    at finalizeResolution (node:internal/modules/esm/resolve:263:11)
    at moduleResolve (node:internal/modules/esm/resolve:932:10)
    at defaultResolve (node:internal/modules/esm/resolve:1056:11)
    at ModuleLoader.defaultResolve (node:internal/modules/esm/loader:654:12)
    at ModuleLoader.#cachedDefaultResolve (node:internal/modules/esm/loader:603:25)
    at ModuleLoader.resolve (node:internal/modules/esm/loader:586:38)
    at ModuleLoader.getModuleJobForImport (node:internal/modules/esm/loader:242:38)
    at ModuleJob._link (node:internal/modules/esm/module_job:135:49) {
  code: 'PLUGIN_ERROR',
  url: 'file:///path/to/project/sample-repo/node_modules/@mui/utils/formatMuiErrorMessage',
  pluginCode: 'ERR_UNSUPPORTED_DIR_IMPORT',
  plugin: 'react-router',
  hook: 'writeBundle'
}
```

### SSRを有効化している場合のエラー内容

ちなみにSSRを有効化している場合は、ビルドが成功する代わりに開発サーバーの起動に失敗する状態でした。

`Element type is invalid: expected a string (for built-in components) or a class/function (for composite components) but got: object.`

# 環境

```terminal
$ npx envinfo --system --binaries \
  --npmPackages '{vite,react*,@react-router/*,@mui/*,@emotion/*,typescript}'

  System:
    OS: macOS 15.1.1
    CPU: (8) arm64 Apple M2
    Memory: 104.55 MB / 8.00 GB
    Shell: 5.9 - /bin/zsh
  Binaries:
    Node: 22.12.0 - ~/.volta/tools/image/node/22.12.0/bin/node
    npm: 10.9.0 - ~/.volta/tools/image/node/22.12.0/bin/npm
  npmPackages:
    @emotion/react: ^11.14.0 => 11.14.0 
    @emotion/styled: ^11.14.0 => 11.14.0 
    @mui/material: ^6.3.1 => 6.3.1 
    @react-router/dev: ^7.1.1 => 7.1.1 
    @react-router/node: ^7.1.1 => 7.1.1 
    @react-router/serve: ^7.1.1 => 7.1.1 
    react: ^19.0.0 => 19.0.0 
    react-dom: ^19.0.0 => 19.0.0 
    react-router: ^7.1.1 => 7.1.1 
    typescript: ^5.7.2 => 5.7.2 
    vite: ^5.4.11 => 5.4.11
```

# 対処法

`vite.config.ts` の `ssr.noExternal` に `["@mui/*"]` を追加します。

```diff_typescript
 // vite.config.ts
 export default defineConfig({
   plugins: [reactRouter(), tsconfigPaths()],
   server: {
     open: true,
   },
+  ssr: {
+    noExternal: ["@mui/*"],
+  },
 });
```

この設定により、ViteはMUI関連のモジュールを外部モジュールとして扱わず、プロジェクト内でバンドルするようになります。

:::note info
少しでもHMRを高速化させたいなど、wildcardを使いたくない場合は、以下のように `@mui` 配下のpackage-nameを個別指定できます。

```jsonc
  ssr: {
    noExternal: [
      "@mui/material",
      "@mui/utils",
      "@mui/styled-engine",
      "@mui/system",
    ],
  }
```
:::

エラーは解決しましたが、せっかくなので対処法について少し深掘りしてみましょう。
まずはエラー原因について推察し、 `ssr.noExternal` の指定による挙動の変化を見てみます。

## エラーが起きていた原因

:::note warn
筆者はESMやCJSといった[JavaScript モジュール](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules)に詳しいわけではありません。  
内容に誤解が含まれる可能性がある点をご承知おきください。
:::

エラーメッセージをもう一度確認してみましょう。
`not supported resolving ES modules imported`
このメッセージから、どうやらESM形式の`import`文の仕様を満たせていない可能性が高いと考えられます。

エラーが発生している `node_modules/@mui/material/styles/index.js` (`@mui/material@6.3.1`) ファイルを確認すると以下のようなコードが含まれています。

```javascript
// node_modules/@mui/material/styles/index.js を一部抜粋
import _formatMuiErrorMessage from "@mui/utils/formatMuiErrorMessage";
export { default as THEME_ID } from "./identifier.js";
...
```

`"@mui/utils/formatMuiErrorMessage"` の部分で拡張子が省略されています。しかし、ESMの仕様では `import` 文で拡張子を省略しないことが求められています。
（TypeScriptでimport文を書く時は拡張子を省略するのが一般的ですが、TypeScriptコンパイラやバンドラがtsconfigなどの設定に沿ってよしなにやってくれてる認識です。これ以上は難しくて分かりません。）

>モジュールを含む .js ファイルへの相対または絶対 URL となっています。Node では、拡張子なしのインポートは node_modules におけるパッケージへの参照であることが多いです。バンドラーによっては、拡張子を省略してもよいことにしています。環境を確認してください。
>
>https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import#構文 より引用

エラーメッセージには次のような修正案が記載されています。
`Did you mean to import "@mui/utils/formatMuiErrorMessage/index.js"?`

つまり、拡張子付きで指定すれば問題は発生しないようです。しかし、このコードはMUIライブラリ内のものであり、開発者が直接修正することはできません。そのため、`ssr.noExternal` オプションを使用し、Viteによるバンドル設定を変更することで対処しました。

## `ssr.noExternal` の指定で何が行われているのか

以下はVite公式ドキュメントからの引用です。

>（補足: `ssr.noExternal` を使用することで）指定した依存関係が SSR のために外部化されるのを防ぎます。
>
>https://ja.vite.dev/config/ssr-options#ssr-noexternal より引用

>SSR を実行する場合、依存関係はデフォルトで Vite の SSR トランスフォームモジュールシステムから「外部化」されます。これにより、開発とビルドの両方を高速化します。
>
>https://ja.vite.dev/guide/ssr#ssr-externals より引用

ここで「外部化」とは、依存関係をViteのSSRモジュールトランスフォームシステム外で処理することを指しています。  
`ssr.noExternal` にMUIモジュールを指定すると、**Viteがそのモジュールをトランスパイルおよびバンドルする**ようになるということだと認識しています。

例として、 `ssr.noExternal` に `["@mui/*"]` を指定した場合のビルドアセットを確認してみます。

<details><summary>spaモードのままだとビルド完了後にサーバー向けassetは削除されてしまうので、確認のために一旦ssrを有効化します。</summary>

```diff_typescript
// react-router.config.ts
import type { Config } from "@react-router/dev/config";

export default {
- ssr: false,
+ ssr: true,
} satisfies Config;
```

</details>

ビルド時にエラーが発生していたサーバー用アセット ( `build/server/index.js` ) を確認すると、 `@mui/utils/formatMuiErrorMessage` は外部モジュールからimportするのではなくローカル関数として記述されていることがわかります。  
これは、ViteがMUIモジュールを1つのJavaScriptファイルにバンドルした結果です。

```build/server/index.js
...
function formatMuiErrorMessage(code, ...args) {
  const url = new URL(`https://mui.com/production-error/?code=${code}`);
  args.forEach((arg2) => url.searchParams.append("args[]", arg2));
  return `Minified MUI error #${code}; visit ${url} for the full message.`;
}
...
```

この動作は、以下の公式ドキュメントで説明されています。

>`webworker` のランタイムなどの場合、SSR のビルドを 1 つの JavaScript ファイルにバンドルしたい場合があります。`ssr.noExternal` を true に設定することで、この動作を有効にできます。
>
>[https://ja.vite.dev/guide/ssr#ssr-バンドル](https://ja.vite.dev/guide/ssr#ssr-%E3%83%8F%E3%82%99%E3%83%B3%E3%83%88%E3%82%99%E3%83%AB) より引用

# 最後に

JavaScriptモジュール（ESM/CJS）の仕組みは非常に奥が深く、簡単に理解できるものではありませんね。
なお、MUIの次のメジャーバージョン（v7）ではESMの完全サポートが予定されています。この変更により、今回のような問題が解決されることが期待されます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/07f73dbf-776b-ab7d-7f6b-8bc2f47c3b14.png" alt="Material UI v7 (draft) Milestone" width="600" />

https://github.com/remix-run/remix/issues/7314

https://github.com/mui/material-ui/issues/43980

# 参照資料

[@mui/icons-materialを使うとCannot use import statement outside a moduleと言われる件](https://www.chikko11.net/blog/2024/material-ui-icons-esm/)

https://ja.vite.dev/config/ssr-options#ssr-noexternal

https://zenn.dev/uhyo/articles/typescript-module-option

https://github.com/mui/material-ui/blob/60106b31d0b4e5e304fdb4d612cc62c1c21a20f6/packages/mui-utils/src/formatMuiErrorMessage/formatMuiErrorMessage.ts#L11-L15
