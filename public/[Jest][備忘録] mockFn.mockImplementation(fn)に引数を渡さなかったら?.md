---
title: '[Jest][備忘録] mockFn.mockImplementation(fn)に引数を渡さなかったら?'
tags:
  - Jest
private: false
updated_at: '2023-02-09T21:25:51+09:00'
id: 0663d9734bcc4c96b7de
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

`undefined`が返る

# 試してみたコード

環境は

```text
    "jest": "^28.1.3",
    "ts-jest": "^28.0.8",
    "typescript": "^4.9.3",
```

1~3はオマケ

```sample.spec.ts
const obj = {
  sampleAdd: function (num: number) {
    return num + 10;
  },
};

beforeEach(() => {
  jest.clearAllMocks();
  jest.resetAllMocks();
  jest.restoreAllMocks();
});

it('1', () => {
  expect(obj.sampleAdd(10)).toBe(20);
});

it('2', () => {
  const spy = jest.spyOn(obj, 'sampleAdd');
  expect(obj.sampleAdd(2)).toBe(12);
  expect(spy).toHaveBeenCalledTimes(1);
});

it('3', () => {
  const spy = jest.spyOn(obj, 'sampleAdd').mockImplementation((num: number) => {
    return num + 100;
  });
  expect(obj.sampleAdd(2)).toBe(102);
  expect(spy).toHaveBeenCalledTimes(1);
});

it('4', () => {
  const spy = jest.spyOn(obj, 'sampleAdd').mockImplementation();
  expect(obj.sampleAdd(2)).toBeUndefined(); // undefinedを期待してPASSする
  expect(spy).toHaveBeenCalledTimes(1);
});

```

# 参照

https://jestjs.io/ja/docs/mock-function-api#mockfnmockimplementationfn
