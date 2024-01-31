---
title: '[備忘録][Azure] Logic Appsの繰り返しスケジュールトリガーで分指定ができない時の解決策'
tags:
  - Azure
  - logicapps
private: false
updated_at: '2023-03-31T09:12:43+09:00'
id: 44a5bd8c34c9c408de90
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

2023/03/30時点での情報です。

# TL;DR

- 対象はLogic Appsの「Recurrence」トリガー
    - Preview Designer のみが対象です
- 分指定する時は`[0,15,30,45]`のように配列カッコをつけてあげないといけない

# やりたいこと

Logic Appsを使って、一定時間の間、15分おきにトリガさせたい。

# 参考にしたドキュメント

https://learn.microsoft.com/ja-jp/azure/logic-apps/tutorial-build-schedule-recurring-logic-app-workflow#add-the-recurrence-trigger

# ドキュメントに従ってセットアップ

Logic AppsのPreview Designer画面より、「Recurrence」トリガーを選択。
`Frequency`, `Interval`, `At these hours`を設定し、「分」の情報を入力するところまできました。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/5c5421c7-b93d-0e29-4cd5-80053dec5495.png" width="600">

ふむ、0-59までの有効な分値をカンマ区切りで入力せよ、と。
hourや曜日はチェックボックスでしたが、分は自分で書き込む方式なのか、まあ60個も選択肢があるチェックボックス嫌だしね。

# 分を入力して一旦保存っと

じゃ、こんな感じで15分おきにして。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/1b2933e7-ec74-2a6c-a050-51b327425ded.png" width="500">

一旦保存しとくか。保存ボタンポチッ。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/b89c825a-6c5c-23b1-b147-06cff3f025f3.png)

ええ......？
`0,15,30,45`を`System.Int32[]`型に変換できませんでしたって言われても......

# 何が悪いんだろう

何度見返しても、[公式ドキュメント](https://learn.microsoft.com/ja-jp/azure/logic-apps/tutorial-build-schedule-recurring-logic-app-workflow#add-the-recurrence-trigger)通りの記述をしてるし、プレースホルダとも同じ書き方です。

いったん生成されたCodeを見てみます。

```json
{
  "type": "Recurrence",
  "recurrence": {
    "frequency": "Week",
    "interval": 1,
    "timeZone": "Tokyo Standard Time",
    "schedule": {
      "hours": [
        "14",
        "15",
        "16"
      ],
      "minutes": "0,15,30,45",
      "weekDays": [
        "Sunday",
        "Saturday",
        "Friday",
        "Thursday",
        "Wednesday",
        "Tuesday",
        "Monday"
      ]
    }
  }
}
```

`minutes`キーに文字列が入ってるのが気になりますね。
`hours`と`weekDays`は配列なのに。

ま、まさか...

こうやって`[0,15,30,45]`配列のカッコをつけてあげたら......

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/c15d2cfb-0643-5333-6f04-a322b5a31abc.png" width="500">

# いけた

保存も成功しました。

```diff_json
{
  "type": "Recurrence",
  "recurrence": {
    "frequency": "Week",
    "interval": 1,
    "timeZone": "Tokyo Standard Time",
    "schedule": {
      "hours": [
        "14",
        "15",
        "16"
      ],
-      "minutes": "0,15,30,45",
+      "minutes": [
+        0,
+        15,
+        30,
+        45
+      ],
      "weekDays": [
        "Sunday",
        "Saturday",
        "Friday",
        "Thursday",
        "Wednesday",
        "Tuesday",
        "Monday"
      ]
    }
  }
}
```

# 公式のissue

https://github.com/Azure/LogicAppsUX/issues

調べてはみましたが、多分報告はあがってないと思います。
もうちょっと調査してからissue発行しようかな。

## 2023/03/31追記 issue発行しました

https://github.com/Azure/LogicAppsUX/issues/1859

MicrosoftのContributorの方から、[コメント](https://github.com/Azure/LogicAppsUX/issues/1859#issuecomment-1490754537)がつきまして、繰り返しトリガーの描写について検討を進めていて、こちらのバグについても取り込む予定らしいです。
OSSにissue立てるのは初めてだったんですが、なんだかすごく達成感がありますね。
