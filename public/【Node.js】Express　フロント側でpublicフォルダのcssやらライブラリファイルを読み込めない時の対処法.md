---
title: 【Node.js】Express　フロント側でpublicフォルダのcssやらライブラリファイルを読み込めない時の対処法
tags:
  - Node.js
  - jQuery
  - Express
private: false
updated_at: '2020-06-26T18:15:47+09:00'
id: a323e8459b4b5f4c455e
organization_url_name: null
slide: false
ignorePublish: false
---
#jqueryが読み込めない・・・
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/0a859323-4aa3-dd0b-f334-e04ef745a3d5.png)
index.ejsから
・public/lib/jquery-3.5.1.min.jsw
・public/css/index.css
を読み込めない！

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/6e9a52d0-0752-cae7-359f-24ff01a5bf2a.png)
404 Not Foundと言われていますね


##なんで・・・？
```index.ejs
<!-- css -->
<link rel="stylesheet" type="text/css" href="public\css\index.css">
<!-- jQuery -->
<script type="text/javascript" src="public\lib\jquery-3.5.1.min.js"></script>
```
ちゃんと相対パス書いてあげてるのになー
絶対パスに変えてみてもダメだった

#解決策

```app.js
// 静的ファイルは無条件に公開
app.use('/public', express.static('public'))
```
app.jsでpublicフォルダをstaticにしてあげるだけでよかった！


#おわり
みなさんも同じ手詰まりをしないように
