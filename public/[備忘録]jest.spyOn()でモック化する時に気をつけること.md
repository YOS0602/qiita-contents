---
title: '[備忘録]jest.spyOn()でモック化する時に気をつけること'
tags:
  - Node.js
  - Jest
private: false
updated_at: '2022-09-13T09:09:54+09:00'
id: 8d8ab306275352c6c226
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

jestを使ってユニットテストをしていると、関数やメソッドのモック化は頻繁に行う。
[jest.spyOn()](https://jestjs.io/ja/docs/jest-object#jestspyonobject-methodname)は、モック関数を作るのに便利なメソッドだが、つまづいたことがあったので備忘

# 事象について

## 私の今までの認識

- `jest.spyOn()`したメソッドは、下記を使って戻り値や実装を書き換えてやることで、実行されないと思っていた（[このあたり](https://jestjs.io/ja/docs/mock-function-api)のヤツ）
    - `mockResolvedValue`
    - `mockImplementation`
    - などなど

## 実際

- 気をつけないと実装したままのコードが実行されちゃう
    - 特に`new`したインスタンスは、mockしようとしてるのが、実行されるインスタンスなのかをよく確認しないと、テスト実行時に、実装コード通りの動きをしてしまう

## コードサンプル

テストしたいファイル

- テスト対象
    - クラス：`Controller`
    - メソッド：`testTarget`
- テスト対象が依存するモジュール：`ModuleA`

```sample.ts
export class ModuleA {
  public async exec(): Promise<string> {
    // 外部APIなどテストで実行したくない処理
    console.log('これが表示されちゃダメ')
    return 'some results...'
  }
}

export class Controller {
  private _moduleA: ModuleA

  public constructor() {
    this._moduleA = new ModuleA()
  }

  /**
   * testしたいメソッド
   */
  public async testTarget(): Promise<boolean> {
    // do something...
    const r = await this._moduleA.exec()
    return r.length ? true : false
  }
}
```

# 実際に私がやっていたこと

テストコード

- まず依存している`ModuleA.exec()`の戻り値をmock
- テスト対象メソッドを実行し、expect

```sample.spec.ts
import { ModuleA, Controller } from './sample'

it('should return true', async () => {
  // given ModuleA.exec()のモック化
  const moduleA: ModuleA = new ModuleA()
  jest.spyOn(moduleA, 'exec').mockResolvedValue('hoge')
  // when Controller.testTarget()実行
  const controller: Controller = new Controller()
  // then resolveされたtrue
  await expect(controller.testTarget()).resolves.toBe(true)
})
```

## 実行結果

- テスト自体はPASS
- しかし、実行されたくない`ModuleA.exec()`が動いてしまっている

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/b58912af-480e-3101-8cfe-93396576d887.png)

## なんとなくの推察

1. `sample.ts`の`Controller`コンストラクタで実行し保持している`ModuleA`のインスタンス
2. `sample.spec.ts`の中で保持する`ModuleA`のインスタンス

もちろん両者は異なるオブジェクトであり、specファイルでmock化したのはあくまで「2」のインスタンスが保持する`ModuleA.exec()`だった
そのため、`sample.ts`側ではモック化されていない`ModuleA.exec()`がコード通りに動いてしまった

違ってたらゴメン

# 対策

## その1 `prototype`を使う

`jest.spyOn()`に渡す第1引数のオブジェクトとして`ModuleA.prototype`を用いる

```diff_typescript
import { ModuleA, Controller } from './sample'

it('should return true', async () => {
  // given ModuleA.exec()のモック化
- const moduleA: ModuleA = new ModuleA()
- jest.spyOn(moduleA, 'exec').mockResolvedValue('hoge')
+ jest.spyOn(ModuleA.prototype, 'exec').mockResolvedValue('hoge')
  // when Controller.testTarget()実行
  const controller: Controller = new Controller()
  // then resolveされたtrue
  await expect(controller.testTarget()).resolves.toBe(true)
})
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/116617f8-1245-8f8a-ec5b-535867f699eb.png)

## その2 `getter`でインスタンス化したオブジェクトをテストファイルで使用する

`sample.spec.ts`

- 実装ファイルのコンストラクタで生成されたインスタンスに対し`jest.spyOn()`する

```diff_typescript
import { ModuleA, Controller } from './sample'

it('should return true', async () => {
+ const controller: Controller = new Controller()
  // given ModuleA.exec()のモック化
- const moduleA: ModuleA = new ModuleA()
+ const moduleA: ModuleA = controller.moduleA // getterからインスタンスを取得
  jest.spyOn(moduleA, 'exec').mockResolvedValue('hoge')
  // when Controller.testTarget()実行
- const controller: Controller = new Controller()
  // then resolveされたtrue
  await expect(controller.testTarget()).resolves.toBe(true)
})
```

`sample.ts`の`Controller`クラス

- 外部から取得できるように`getter`を作る

```diff_typescript
export class Controller {
  private _moduleA: ModuleA

+  public get moduleA(): ModuleA {
+    return this._moduleA
+  }

  public constructor() {
    this._moduleA = new ModuleA()
  }

  /**
   * testしたいメソッド
   */
  public async testTarget(): Promise<boolean> {
    // do something...
    const r = await this._moduleA.exec()
    return r.length ? true : false
  }
}
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/a2ebc9db-8884-2f5e-7501-72892a4148fe.png)

# 所感

- `getter`とはいえ、テストのためだけのメソッドが実装ファイルにあるのはなんか違和感
- だから`prototype`の方が良いかもーって思ってる
- `Controller`クラスを使う上位から、`ModuleA`のインスタンスをバケツリレーする方法もあるか
    - `React`とかでもそうだけど、バケツリレーはめんどいし依存性高まったりテスト容易性が低くなったりする気がしてるから避けてる
