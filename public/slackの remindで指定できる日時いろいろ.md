---
title: slackの/remindで指定できる日時いろいろ
tags:
  - Slack
private: false
updated_at: '2024-09-08T13:42:54+09:00'
id: e52594d4faa6848800cb
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

[slackのリマインダー](https://slack.com/intl/ja-jp/help/articles/208423427-%E3%83%AA%E3%83%9E%E3%82%A4%E3%83%B3%E3%83%80%E3%83%BC%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)をご存知でしょうか？
確認しなきゃいけないメッセージが来てるけど、今は手が離せないから未読のまま放置しよう......！ みたいなことをせずに済むようになるかもしれません。

## `/remind`とは

- GUIから設定できる「後でリマインドする」を、好きなメッセージ、好きなタイミングで指定できるコマンド
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/fdd67803-b275-bb50-f1f5-a4f8ba271061.png)

- 使い方はこんな感じ
    ```text
    /remind me
    Hello
    in 3 minutes
    ```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/691e0c39-ba5c-35ac-3042-cba766962d6c.png)

## 形式

>/remind [@someone or #channel] [what] [when]

### いつリマインドするかの指定

whatはメッセージ部分になるので何でも書けるのですが、whenは今の所(2024/09/08)英語じゃないと認識してくれません。

```text
/remind me
進捗確認
毎週金曜日
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/cc09a31e-326d-e455-5ec5-982c934cfd64.png)

なので、指定できる色々をメモしておきます。新しいのに気づいたら順次追加予定です。

# 簡単なやつ

- 今日のいつ：`today 19:30` ※today省略可能
- 明日のいつ：`tomorrow 9:00`
- 次のX曜日： `next tuesday`

# X分後

`in 20 minutes`

`in 30 seconds`も`in 2 hours`もできます
`in 3 weeks`でX週間後に

# 毎週

`every friday`

時刻もつけれる
`every friday 15:00`

※時刻をつけないとデフォルト設定の時刻にリマインドされます
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/ca1b9aad-b8ed-4fa8-86b0-88ca673669e3.png)


# 毎月

https://slack.com/intl/ja-jp/help/articles/4417526840723-%E6%AF%8E%E6%9C%88%E3%81%AE%E3%83%AA%E3%83%9E%E3%82%A4%E3%83%B3%E3%83%80%E3%83%BC%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%97%E3%81%A6%E7%B5%8C%E8%B2%BB%E3%82%92%E7%94%B3%E8%AB%8B%E3%81%99%E3%82%8B

`on the 28th of every month`

時刻もつけれる
`on the 1st of every month 7:00`

# 2週間に一回

`thursday biweekly`

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/f7fe30b5-4d84-aa9d-ef00-df0b941d95a3.png)

# 毎年

`on February 16th of every year 9:00`

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/59f7e4c8-98f5-25af-f2c2-67b0d07454c6.png)
