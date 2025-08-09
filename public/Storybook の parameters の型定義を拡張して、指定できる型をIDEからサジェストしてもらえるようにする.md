---
title: Storybook の parameters の型定義を拡張して、指定できる型をIDEからサジェストしてもらえるようにする
tags:
  - tips
  - TypeScript
  - React
  - storybook
private: false
updated_at: '2025-08-09T15:46:38+09:00'
id: 93281cd555b1e76ea4b0
organization_url_name: null
slide: false
ignorePublish: false
---
## Storybook で使用できる parameters で感じる課題

Storybook の stories から指定できる `parameters` は、多くの addon で使われたり、decorator に値を渡す用途で使われたりと、とても便利です。一方で、storybook ライブラリが提供している型定義は次のとおりで、IDE の補完が効きません。結果として typo が混入しやすく、どんなプロパティが指定できるのかも利用側からは分かりづらい、という課題がありました。

```ts
interface Parameters {
    [name: string]: any;
}
```

## ライブラリから提供されている型定義を拡張する

そこで `Parameters` 型をモジュール拡張でマージし、プロジェクトで扱いたいプロパティをあらかじめ宣言しておきます。

```ts
import type { ReactRouterAddonStoryParameters } from "storybook-addon-remix-react-router";

// Parameters がexportされてるパッケージ名を書く。Storybook@8では"@storybook/react"となる
declare module "@storybook/react-vite" {
  export interface Parameters {
    // addonに渡すパラメータ
    reactRouter?: ReactRouterAddonStoryParameters;
    // decoratorで定義しているContextProviderなどに渡すパラメータ
    mode?: "light" | "dark";
    // Storybookにデフォルトで指定できるlayoutパラメータの型
    layout?: 'padded' | 'centered' | 'fullscreen';
  }
}
```

`declare module` 宣言は任意の .ts or .d.ts ファイル（例： `src/types/storybook.d.ts` ）に書くことができますが、 `tsconfig.json` の `include` に含まれないとtsコンパイラが読み取ってくれないようなので注意しましょう。

### IDEで補完が効くようになる

Storybook のデコレータで `parameters` を参照すると、型補完により `mode` の候補がサジェストされます。ちょっとしたユニオン型でも補完が効くので、記述ミスの予防や読みやすさの向上に役立ちます。

```tsx
export const sampleStorybookDecorator: Decorator = (Story, ctx) => {
  const mode = ctx.parameters.mode ?? "light";

  return (
    <SomeProvider value={mode}>
      <Story />
    </SomeProvider>
  );
};
```

![型補完.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/8f6b0b2f-c513-4055-afee-3120091a0932.png)

### 使用されているテクニック

TypeScript の Declaration Merging（宣言のマージ）と Module Augmentation（モジュール拡張）を使っています。Declaration Merging は、同名のインターフェース宣言があった場合にプロパティをマージして 1 つの型として扱う仕組みです。Module Augmentation は、`declare module "..."` の形で既存モジュールの型を後から拡張できる仕組みです。

上の例では `@storybook/react-vite` モジュール内の `Parameters` インターフェースに対して、`reactRouter` と `mode` を追加しています。これにより Storybook 側の定義は壊さず、プロジェクト固有の `parameters` だけを IDE 補完の対象にできます。

## 参考にしたページ

- [Storybook の parameters の型をいい感じにして便利に使う](https://zenn.dev/layerx/articles/e94d5cf1a3caba)
- [TypeScript: Documentation - Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)
- [TypeScriptのdeclareやinterface Windowを勘で書くのをやめる2022](https://zenn.dev/qnighy/articles/9c4ce0f1b68350#%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E6%8B%A1%E5%BC%B5)
