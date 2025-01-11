---
title: MUIのuseTheme()の戻り値を、カスタマイズした型に推論させる
tags:
  - interface
  - material-ui
  - MUI
private: false
updated_at: '2025-01-11T14:15:02+09:00'
id: 53f60f313120c91f5235
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

MUIでは [Theme provider](https://mui.com/material-ui/customization/theming/#theme-provider) を利用して、MUIから提供されている[デフォルトのtheme object](https://mui.com/material-ui/customization/default-theme/)をカスタマイズすることができます。
しかし、型推論では、カラーコードなど具体的な設定値が取得されません。この記事では、`useTheme()` の戻り値をカスタマイズした型に推論させる方法を紹介します。

:::note info
theme object にオリジナルのプロパティを追加したい場合、[モジュール拡張 (module augmentation)](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation)を行う必要があります。
詳細は以下の公式ドキュメントをご確認ください。  
https://mui.com/material-ui/customization/theming/#typescript
:::

# コード例

以下は、`useTheme()` の型をカスタマイズするコード例です。

```typescript
import { createTheme, type Theme } from "@mui/material/styles";

type CreateThemeOptions = Parameters<typeof createTheme>[0];

// theme variables をカスタマイズする
const themeOptions = {
  palette: {
    text: {
      primary: "#1A1A1C",
      secondary: "#626264",
      disabled: "#949497",
    },
  },
  typography: {
    fontFamily: [
      '"Noto Sans JP Variable"',
      "-apple-system",
    ].join(","),
  },
} as const satisfies CreateThemeOptions;

export const theme = createTheme(themeOptions);

/**
 * カスタマイズされた theme のリテラルを含む型
 */
type CustomizedTheme = typeof themeOptions & Theme;

// モジュール拡張
declare module "@mui/material/styles" {
  /**
   * `useTheme` のジェネリクス型を上書きする。
   * @example
   * ```ts
   * import { useTheme } from "@mui/material/styles";
   * const theme = useTheme();
   * // `theme` は `CustomizedTheme` に推論される
   * ```
   */
  export function useTheme<T = CustomizedTheme>(): T;
}
```

## 補足

- `themeOptions` はカスタマイズされたオプションを保持するオブジェクトであり、その型情報がリテラル型として保持されます。
- `CustomizedTheme` により、MUIの `Theme` 型と `themeOptions` のリテラル型を結合します。
- `useTheme` を上書きすることで、`themeOptions` の型が推論されるようになります。

## 型推論の動作確認

`const theme = useTheme();` を利用すると、`themeOptions` で定義したカラーコードなどが型推論されることを確認できます。

![カラーコードが型推論されている画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/9f95d4d0-8f9d-876a-5e2e-551eb3c197db.png)

また、[ThemeProviderコンポーネント](https://mui.com/material-ui/customization/theming/#themeprovider)で正しく設定したthemeを渡すことで、UI描画にも正しく反映されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/03d9feba-c789-555b-d64c-5fb7826f78d6.png)

# 参照資料

https://mui.com/material-ui/customization/default-theme/

https://mui.com/material-ui/customization/theming/

https://zenn.dev/qnighy/articles/9c4ce0f1b68350#%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E6%8B%A1%E5%BC%B5
