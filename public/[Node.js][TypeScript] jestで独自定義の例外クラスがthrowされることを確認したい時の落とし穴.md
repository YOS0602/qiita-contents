---
title: '[Node.js][TypeScript] jestで独自定義の例外クラスがthrowされることを確認したい時の落とし穴'
tags:
  - TypeScript
  - Jest
private: false
updated_at: '2022-12-14T00:36:29+09:00'
id: 1e2a6b4980fb3270ca76
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

jestで何かしらがthrowされる時は、公式通りに`toThrow()`マッチャを使えば良いと思っていた時期が僕にもありました。

- [.rejects](https://jestjs.io/ja/docs/expect#rejects)
- [.toThrow(error?)](https://jestjs.io/ja/docs/expect#tothrowerror)

## バージョンなど

```
macOS 12.6
node v18.12.1
npm 8.19.2
"@types/jest": "^28.1.8",
"jest": "^28.1.3",
"ts-jest": "^28.0.8",
"typescript": "^4.9.3"
```

# 事象

デバッグしても実行ログを見ても`throw`されているのに、
`Received function did not throw`が表示されてテストがこける

```terminal
 FAIL   sample/__tests__/index.spec.ts
  ✕ should thrown in doSomething (1 ms)

  ● should thrown in doSomething

    expect(received).rejects.toThrow(expected)
    Expected constructor: HandmadeException
    Received function did not throw

       8 |
       9 |   // HandmadeExceptionクラスがthrowされることを期待
    > 10 |   await expect(app.doSomething()).rejects.toThrow(HandmadeException)
         |                                           ^
      11 | })
      12 |

      at Object.toThrow (node_modules/expect/build/index.js:241:22)
      at Object.<anonymous> (sample/__tests__/index.spec.ts:10:43)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        0.881 s, estimated 1 s
Ran all test suites matching /index.spec.ts/i.
```

## テストしようとしていたもの

こんな感じでテストしたい処理がある。
`HandmadeException`はお手製のExceptionクラス。

```sample/index.ts
class App {
  async doSomething(): Promise<void> {
    try {
      await someModule.sleep()
    } catch (error) {
      throw new HandmadeException('failed to make dir', error as string)
    }
  }
}

const someModule = {
  sleep: () => {
    return new Promise((resolve) => setTimeout(resolve, 1000))
  },
}

class HandmadeException {
  msg: string
  stack?: string | undefined
  constructor(msg: string, stack?: string) {
    this.msg = msg
    this.stack = stack
  }
}

export { App, someModule, HandmadeException }
```

```sample/__tests__/index.spec.ts
import { App, someModule, HandmadeException } from '..'

const app = new App()

it('should thrown in doSomething', async () => {
  // someModule.sleep()でrejectされるようmock
  jest.spyOn(someModule, 'sleep').mockRejectedValueOnce('error!')

  // HandmadeExceptionクラスがthrowされることを期待
  await expect(app.doSomething()).rejects.toThrow(HandmadeException)
})
```

# 原因

[jestの`toThrow(error?)`マッチャの実装コード](https://github.com/facebook/jest/blob/2f6931e91d5ab126de70caf150c68709752e7f6c/packages/expect/src/toThrowMatchers.ts#L223
)を見てみると、
`throwされたobjectのmessage === 期待するobjectのmessage`を検証していました。

```typescript
const pass = thrown !== null && thrown.message === expected.message;
```

なので、プロパティとして`message`を持つ`Error`コンストラクタがthrowされている場合や、
Errorを`extends`して作成された独自定義の例外クラスならば、`toThrow(error?)`マッチャが使えた、ということですね。
※`implements`でもいいかもと思ったけど、テストが通らなかった

# 対策

## その1 独自定義例外クラスは必ず`Error`を継承する

```diff_typescript
- class HandmadeException {
+ class HandmadeException extends Error {
    msg: string
    stack?: string | undefined
    constructor(msg: string, stack?: string) {
+       super()
        this.msg = msg
        this.stack = stack
    }
}
```

## その2 テスト時に`toThrow(error?)`マッチャ以外を使用する

自分は`.toBeInstanceOf(Class)`を使いました。

https://jestjs.io/ja/docs/expect#tobeinstanceofclass

```diff_typescript
// sample/__tests__/index.spec.ts
it('should thrown in doSomething', async () => {
  // someModule.sleep()でrejectされるようmock
  jest.spyOn(someModule, 'sleep').mockRejectedValueOnce('error!')

  // HandmadeExceptionクラスがthrowされることを期待
-  await expect(app.doSomething()).rejects.toThrow(HandmadeException)
+  await expect(app.doSomething()).rejects.toBeInstanceOf(HandmadeException)
})
```

## 結果

```terminal
 PASS   sample/__tests__/index.spec.ts
  ✓ should thrown in doSomething (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.946 s, estimated 1 s
Ran all test suites matching /index.spec.ts/i.
```

# 参考

https://tech.bitbank.cc/20201201/

https://github.com/facebook/jest/blob/2f6931e91d5ab126de70caf150c68709752e7f6c/packages/expect/src/toThrowMatchers.ts#L223

