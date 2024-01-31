---
title: '[備忘録] [Python] azure-cliの基本的な使い方'
tags:
  - Python
  - Azure
  - AzureCLI
private: false
updated_at: '2023-01-01T03:52:00+09:00'
id: 47e15fa11b95a09f97f1
organization_url_name: null
slide: false
ignorePublish: false
---
# 環境

```
MacOS: 12.6
Python: 3.9.10
azure-cli==2.32.0
azure-cli-core==2.32.0
```

# 基本的なazコマンドの実行

例えば、`az vm show --resource-group QueryDemo --name TestVM`を実行したければこんな感じ

```az_cli.py
from azure.cli.core import get_default_cli
cli = get_default_cli()
res = cli.invoke(
    [
        "vm",
        "show",
        "--resource-group",
        "QueryDemo",
        "--name",
        "TestVM",
    ]
)
# 0: success
# 1: failure
if res != 0:
    print("ERROR!")
```

もちろん、リソースグループ名などは定数や環境変数としてセットし、参照することも可能
クエリ結果はコンソールにドバッと出力される

## azコマンドリファレンス

ここで調べたいazコマンドは網羅できるはず

https://learn.microsoft.com/ja-jp/cli/azure/reference-index?view=azure-cli-latest

# コンソール出力形式やプロパティのカスタマイズ

このページを見るのが早い

https://learn.microsoft.com/ja-jp/cli/azure/query-azure-cli?tabs=concepts%2Ccmd

https://learn.microsoft.com/ja-jp/cli/azure/format-output-azure-cli

# コンソール出力を変数に格納したい時

`cli.result.result`で取れる

`cli.invoke()`の戻り値で受け取れるって思うよね......
この場合の変数`res`に入ってるのは、先述の通り`0`: success、`1`: failure　なので注意

```py
from azure.cli.core import get_default_cli

cli = get_default_cli()
res = cli.invoke([...])
if cli.result.result:
    print(cli.result.result)
elif cli.result.error:
    print(cli.result.error)
```

参考

https://stackoverflow.com/a/65833287
