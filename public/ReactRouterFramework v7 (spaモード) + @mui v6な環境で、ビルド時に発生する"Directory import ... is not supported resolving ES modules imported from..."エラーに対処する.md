---
title: >-
  ReactRouterFramework v7 (spaモード) + @mui v6な環境で、ビルド時に発生する"Directory import ... is not
  supported resolving ES modules imported from..."エラーに対処する
tags:
  - 'react-router'
  - 'MUI'
  - 'vite'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

`vite.config.ts` の `ssr.noExternal` に `["@mui/*"]` を追加する。
詳しくは [こちら](#対処法) へ。

# はじめに

[SSR機能を無効化](https://reactrouter.com/how-to/spa)した [React Router Framework v7](https://reactrouter.com/7.1.1/home) と [Material UI v6](https://mui.com/material-ui/getting-started/) を組み合わせたアプリケーションを開発しています。
`react-router dev` で開発サーバーは問題なく起動できるのに、 `react-router build` で下記エラーが発生する状況でした。解決するために実施したことと、エラー原因について調査したことをメモしておきます。

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

## SSRを有効化している場合のエラー内容

ちなみにSSRを有効化している場合は、ビルドが成功する代わりに開発サーバーの起動に失敗します。

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

`vite.config.ts` の `ssr.noExternal` に `["@mui/*"]` を追加する。

```diff_typescript
// vite.config.ts
export default defineConfig({
  plugins: [reactRouter(), tsconfigPaths()],
  server: {
    open: true,
  },
  ssr: {
+   noExternal: [
+     "@mui/*",
+   ],
  },
});
```

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

エラーは解決しましたが、せっかくですので対処法についてもう少し詳しく見ていきます。
まずはエラー原因について推察し、 `ssr.noExternal` の指定による挙動の変化を見てみようと思います。

## エラーが起きていた原因

:::note warn
筆者はESMやCJSといった[JavaScript モジュール](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules)に精通しているわけではありません。
誤解を含む場合がありますのでご注意ください。
:::

エラーメッセージをもう一度見てみましょう。
`not supported resolving ES modules imported`
この部分を読んだ感じ、どうやらESM形式のimport文の仕様を満たせていなさそうです。

エラーが発生している `node_modules/@mui/material/styles/index.js` ( `@mui/material@6.3.1` )ファイルを見てみると......

```javascript
// node_modules/@mui/material/styles/index.js を一部抜粋
import _formatMuiErrorMessage from "@mui/utils/formatMuiErrorMessage";
export { default as THEME_ID } from "./identifier.js";
...
```

`"@mui/utils/formatMuiErrorMessage"` の部分で拡張子を省略したimport文が書かれていますね。
しかし、ESMの仕様ではimport文のmodule名は拡張子を省略せず記載することになっています。
（TypeScriptでimport文を書く時は拡張子を省略するのが一般的ですが、TypeScriptコンパイラやバンドラがtsconfigなどの設定に沿ってよしなにやってくれてる認識です。これ以上は難しくて分かりません。）

>モジュールを含む .js ファイルへの相対または絶対 URL となっています。Node では、拡張子なしのインポートは node_modules におけるパッケージへの参照であることが多いです。バンドラーによっては、拡張子を省略してもよいことにしています。環境を確認してください。
>
>https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import#構文 より引用

`Did you mean to import "@mui/utils/formatMuiErrorMessage/index.js"?`
エラーメッセージ内で上記のように示唆してくれている通り、拡張子付きのファイル名まで記述すればこの問題は起きなさそうだと分かります。
しかし、エラーが発生しているのはmuiパッケージ内のコードであるため、自分ではコントロールできません。
そこで、 `ssr.noExternal` オプションを利用しビルド時のバンドル設定を変更することで対処したのだと認識しています。

## `ssr.noExternal` の指定で何が行われているのか

以下はVite公式ドキュメントからの引用です。

>（補足: `ssr.noExternal` を使用することで）指定した依存関係が SSR のために外部化されるのを防ぎます。
>
>https://ja.vite.dev/config/ssr-options#ssr-noexternal より引用

>SSR を実行する場合、依存関係はデフォルトで Vite の SSR トランスフォームモジュールシステムから「外部化」されます。これにより、開発とビルドの両方を高速化します。
>
>https://ja.vite.dev/guide/ssr#ssr-externals より引用

「外部化」というのは、Vite の SSR トランスフォームモジュールシステムを使わないことを指すようです。
`ssr.noExternal` にmuiモジュールを指定すると、**Viteによってモジュールがトランスパイル/バンドルされる** と読み替えることができそうです。

`ssr.noExternal` に `["@mui/*"]` を指定した場合のビルドassetを見てみましょう。
エラーメッセージを読むと、どうやらサーバー向けassetをビルドする際にエラーが発生してそうです。なので `build/server` 配下を調べれば良さそうです。
spaモードのままだとビルド完了後にサーバー向けassetは削除されてしまうので、確認のために一旦ssrを有効化します。

```diff_typescript
// react-router.config.ts
import type { Config } from "@react-router/dev/config";

export default {
- ssr: false,
+ ssr: true,
} satisfies Config;
```

ビルドコマンドを実行した後の `build/server/index.js` ファイルを見てみましょう。今回エラーが発生していた `formatMuiErrorMessage` 関数を探してみます。
すると `index.js` からexportされていないローカル関数（表現が正しいか分かりませんが）として存在することが分かります。
`@mui/utils/formatMuiErrorMessage/index.js` からimportするのではなく、Viteによって1つのJSファイルにバンドルされたということみたいです。

>`webworker` のランタイムなどの場合、SSR のビルドを 1 つの JavaScript ファイルにバンドルしたい場合があります。`ssr.noExternal` を true に設定することで、この動作を有効にできます。
>
>https://ja.vite.dev/guide/ssr#ssr-バンドル より引用

```build/server/index.js
...
function formatMuiErrorMessage(code, ...args) {
  const url = new URL(`https://mui.com/production-error/?code=${code}`);
  args.forEach((arg2) => url.searchParams.append("args[]", arg2));
  return `Minified MUI error #${code}; visit ${url} for the full message.`;
}
...
```

muiのGitHubリポジトリを参照してみましたが、やはり `formatMuiErrorMessage` の実装がそのまま `index.js` に移されたようですね。

```packages/mui-utils/src/formatMuiErrorMessage/formatMuiErrorMessage.ts
export default function formatMuiErrorMessage(code: number, ...args: string[]): string {
  const url = new URL(`https://mui.com/production-error/?code=${code}`);
  args.forEach((arg) => url.searchParams.append('args[]', arg));
  return `Minified MUI error #${code}; visit ${url} for the full message.`;
}
```

（https://github.com/mui/material-ui/blob/60106b31d0b4e5e304fdb4d612cc62c1c21a20f6/packages/mui-utils/src/formatMuiErrorMessage/formatMuiErrorMessage.ts#L11-L15 より引用）

# 最後に

ESM/CJSといったモジュールの仕組みは本当に難しいですね。
ところで、MUIの次のメジャーバージョンであるv7ではESMサポートが予定されているようです。この問題も解決すると良いですね。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/07f73dbf-776b-ab7d-7f6b-8bc2f47c3b14.png" alt="Material UI v7 (draft) Milestone" width="600" />

https://github.com/remix-run/remix/issues/7314

https://github.com/mui/material-ui/issues/43980

# 参照資料

[@mui/icons-materialを使うとCannot use import statement outside a moduleと言われる件](https://www.chikko11.net/blog/2024/material-ui-icons-esm/)

https://ja.vite.dev/config/ssr-options#ssr-noexternal

https://zenn.dev/uhyo/articles/typescript-module-option
