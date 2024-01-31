---
title: '[Node.js] Firestore のバッチ書き込みとBulkWriterの速度比較をしてみた'
tags:
  - Firebase
  - Firestore
private: false
updated_at: '2023-11-09T18:14:39+09:00'
id: 522e7c9cd299dc5910c1
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Firebase (Google Cloud Platform) には Firestore と呼ばれる、フルマネージドでスケーラブルなサーバーレスのドキュメント データベース[^1] があります。
大量のドキュメント書き込みをする要件があり、良い方法がないか調べたところ、[バッチ(一括)書き込み](https://firebase.google.com/docs/firestore/manage-data/transactions?hl=ja#batched-writes) と [BulkWriter(並列書き込み)](https://googleapis.dev/nodejs/firestore/latest/BulkWriter.html) という2つの方法を見つけました。

>バッチ書き込みは、シリアル化された書き込みより優れたパフォーマンスを発揮しますが、並列書き込みほど優れてはいません。

上記のように、公式曰くBulkWriterを使った方が速いそうですが、どれほど差が出るのか検証してみることにしました。
なお、ランタイムはNode.jsです。

## 各種バージョン&スペック

```txt
Node 16.17.0
firebase-admin 11.11.0 (一部 10.3.0でも計測)

MacOS 14.1
M2 プロセッサ
8GB メモリ
```

# TL;DR

ver `11.11.0`
※3回計測し平均時間を算出

|  | 500 docs (ms) | 1,000 docs (ms) |
|:-:|:-:|:-:|
| バッチ  | 771 | 1602 |
| BulkWriter | (未計測) | 1241 |

# バッチ書き込みの速度計測

```ts
describe('大量ドキュメント作成の速度検証', () => {
  const db = initializeApp().firestore();

  test.only('batch 1,000 docs', async () => {
    async function executeBatch() {
      const batch = db.batch();

      // 500件までの制限があるため、500件ずつコミットする。
      // See https://firebase.google.com/docs/firestore/manage-data/transactions?hl=ja#batched-writes
      for (let i = 1; i <= 500; i += 1) {
        batch.create(db.collection('batch-test').doc(), {
          id: i,
          name: `${i}-batch-test`,
          created: new Date(),
          data: {
            nested: true,
          },
          list: [i],
        });
      }
      await batch.commit();
    }

    console.time('batch');

    await executeBatch();
    console.timeLog('batch', '500/1,000 created.');
    await executeBatch();

    console.timeEnd('batch');
  });
});
```

ver `10.3.0`

|  | 500 docs (ms) | 1,000 docs (ms) |
| --- | --- | --- |
| 1st | 1523 | 2468 |
| 2nd | 1322 | 2010 |
| 3rd | 1247 | 2124 |

ver `11.11.0`

|  | 500 docs (ms) | 1,000 docs (ms) |
| --- | --- | --- |
| 1st | 979 | 1740 |
| 2nd | 596 | 1518 |
| 3rd | 737 | 1547 |

# BulkWriterの速度検証

```ts
describe('大量ドキュメント作成の速度検証', () => {
  const db = initializeApp().firestore();

  test.only('BulkWriter 1,000 docs', async () => {
    console.time('BulkWriter');

    const bulkWriter = db.bulkWriter();

    for (let i = 1; i <= 1000; i += 1) {
      bulkWriter.create(db.collection('BulkWriter-test').doc(), {
        id: i,
        name: `${i}-BulkWriter-test`,
        created: new Date(),
        data: {
          nested: true,
        },
        list: [i],
      });
    }
    await bulkWriter.close();

    console.timeEnd('BulkWriter');
  });
});
```

ver `10.3.0`

|  | 1,000 docs (ms) |
| --- | --- |
| 1st | 1297 |
| 2md | 1625 |
| 3rd | 1275 |

ver `11.11.0`

|  | 1,000 docs (ms) |
| --- | --- |
| 1st | 1230 |
| 2md | 1232 |
| 3rd | 1260 |

# 感想

やはり公式通り、BulkWriterの方が速いですね。数百件程度なら気にならない差かもしれませんが、何千、何万単位になってくるとBulkWriterを採用することになってきそうです。バッチ書き込みは500件ずつしかcommitできない制約もありますし。
ただし、書き込みのAPI リクエスト最大サイズは`10MiB`という制約があるそうなので、BulkWriterを使ってるからといって、膨大な量のドキュメントを一気に登録しようとすると、制限に引っかかってしまう恐れがあるかもしれません。

https://firebase.google.com/docs/firestore/quotas?hl=ja

バッチ書き込みを使うなら、500件ずつcommitするのはもちろんですが、BulkWriterを使用する際も、ある程度エンキューされたドキュメントが溜まってきたら、`BulkWriter.flush()`([Doc](https://googleapis.dev/nodejs/firestore/latest/BulkWriter.html#flush))するのが良さそうです。

[^1]: [Firestore: NoSQL ドキュメント データベース | Google Cloud](https://cloud.google.com/firestore?hl=ja)より引用
