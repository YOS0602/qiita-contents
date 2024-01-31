---
title: Excel for Macで全シートA1セルに戻し、倍率を100%にする
tags:
  - Mac
  - Excel
  - VBA
  - ExcelVBA
private: false
updated_at: '2022-10-06T21:11:26+09:00'
id: c049c15d9e9af6c68011
organization_url_name: null
slide: false
ignorePublish: false
---
# 環境

MacOS 12.6
Excel for Mac 16.65

# 参考

https://qiita.com/nekomoff/items/128e8580e726f2f4f4a5

# コード

```vb
Sub Cursor_A1()
' 画面更新をさせない（チカチカするため）
Application.ScreenUpdating = False

Dim ws As Worksheet
' 各シートを順番に見ていく
For Each ws In Worksheets
' カーソルをA1に移動
ws.Activate
ws.Range("A1").Select
' 表示位置を左上端に移動
ActiveWindow.ScrollColumn = 1
ActiveWindow.ScrollRow = 1
' 表示倍率を100%に戻す
ActiveWindow.Zoom = 100
Next ws

' １枚目のシートに移動
Sheets(1).Select
'画面更新を再開させておく
Application.ScreenUpdating = True

End Sub
```

# 変更点

```diff_visual_basic
' カーソルをA1に移動
- ws.Select
- Range("A1").Select
+ ws.Activate
+ ws.Range("A1").Select
```

これをやらないと、画像のエラーがちょくちょく出るんですよね

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/59ca28ff-855b-7092-899a-00f6abab71bb.png)

VBにはそんなに興味がないので原因の深掘りはしてません。動けばそれでいいや。
