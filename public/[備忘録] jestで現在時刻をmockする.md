---
title: '[備忘録] jestで現在時刻をmockする'
tags:
  - TypeScript
  - Jest
private: false
updated_at: '2023-02-01T17:32:49+09:00'
id: db802e6ad920af20563a
organization_url_name: null
slide: false
ignorePublish: false
---
# TL;DR

```ts
describe('timer mock', () => {
  beforeEach(() => {
    jest.useFakeTimers()
  })

  it('test', () => {
    jest.setSystemTime(new Date('2023-02-01T12:05:13.354+09:00'))
    // TODO assert
  })

  afterEach(() => {
    jest.useRealTimers()
  })
})
```

# `useFakeTimers`しないと...

下記がターミナルに表示され、現在時刻はmockできない。

```terminal
A function to advance timers was called but the timers APIs are not replaced with fake timers. Call `jest.useFakeTimers()` in this test file or enable fake timers for all tests by setting 'fakeTimers': {'enableGlobally': true} in Jest configuration file.
    Stack Trace:
```

# Reference

https://jestjs.io/ja/docs/jest-object#jestsetsystemtimenow-number--date
