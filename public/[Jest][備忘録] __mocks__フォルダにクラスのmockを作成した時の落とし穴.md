---
title: '[Jest][備忘録] __mocks__フォルダにクラスのmockを作成した時の落とし穴'
tags:
  - ユニットテスト
  - Jest
private: false
updated_at: '2023-02-02T16:19:05+09:00'
id: b24719547cd0e0d6305e
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

- `__mocks__`フォルダに定義するクラスは、必ず実体と同じ名前で定義
    - `export`も同じ名前にする
- テストで使う時に実体と同じ名前だと混乱するので、下記のように`import`時に別名をつけてあげる
    - `import { ExampleClass as MockExampleClass } from '../infrastructures/repositories/__mocks__/ExampleClass'`

# 環境

```txt
MacOS 13.1
Node.js 18.12.1
npm 8.19.2
typescript 4.9.3
jest 28.1.3
ts-jest 28.0.8
```

# つまづいたこと

jestの[マニュアルモック](https://jestjs.io/ja/docs/manual-mocks)ドキュメントにあるように、`__mocks__`フォルダ配下にモックモジュールを定義できる。
テストファイル側で`jest.mock('./moduleName')`としてやれば、そのモジュールの実体が実行されてしまうことはなく、全てjestのmock関数に置き換えられる便利な代物だ。

しかし、クラスのmockモジュールを`__mocks__`フォルダに作成、テストを実行したところ、下記のエラーが出力されてテストがFAILした。

>TypeError: "x" is not a constructor

## その時の実装

実体側

```ExampleClass.ts
class ExampleClass {
  // 省略
}

export { ExampleClass }
```

mock実装

```__mocks__/ExampleClass.ts
class MockExampleClass {
  // 省略
}

export { MockExampleClass }
```

こうしたのは、クラス名として`Mock`をつけておいた方が、テストファイルで利用する時に実体なのかmockクラスなのか分かりやすいだろう、という思いから。
しかしこれが良くなかった。

# テストが落ちる理由

もし実装側で`ExampleClass`をインスタンス化する処理があったとする。

```otherModule.ts
import { ExampleClass } from './ExampleClass'

const exampleClass = new ExampleClass()
```

テスト側で`ExampleClass`をmockした場合、`otherModule.ts`でimportが参照するのは`MockExampleClass`になってしまい、コンストラクタ名が一致しないため`new`できずに落ちていた。

```otherModule.spec.ts
jest.mock('./ExampleClass')
// ->import { ExampleClass } from './ExampleClass' している箇所は
// 全てMockExampleClassを参照するようになってしまう
```

# 解決策

- `__mocks__`フォルダに定義するクラスは、必ず実体と同じ名前で定義する
- exportも実体と同じ名前にする
- テスト側で実体とmockクラスが同じ名前で良いのならそのまま使えば良いが、テストの可読性が悪いため、下記のように別名をつけてimportする方が良いと思われる。

実体側

```ExampleClass.ts
class ExampleClass {
  // 省略
}

export { ExampleClass }
```

mock実装

```__mocks__/ExampleClass.ts
/**
 * ExampleClassをmockしたクラス
 *
 * 実体と同じクラス名で定義・exportしないと、テスト実行時、実装ファイルでインスタンス化できなくなってしまう。
 * テストファイルでimportする時は可読性向上のため、`import { ExampleClass as MockExampleClass }`のように別名をつけること。
 */
class ExampleClass {
  // 省略
}

export { ExampleClass }
```

テストファイル

```otherModule.spec.ts
import { ExampleClass as MockExampleClass } from '__mocks__/ExampleClass'
```

# 参考

https://jestjs.io/ja/docs/manual-mocks
