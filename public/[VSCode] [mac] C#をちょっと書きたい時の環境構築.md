---
title: '[VSCode] [mac] C#をちょっと書きたい時の環境構築'
tags:
  - Mac
  - C#
  - VSCode
private: false
updated_at: '2023-01-07T21:31:55+09:00'
id: fae06fde5b2f0bfb7a20
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

- Visual Studio Codeでやります
    - Visual StudioがC#にとって最強ツールらしいのは知ってるけど、ちょっと試したいだけでいっぱいインストールさせられたくないし
- macOS 12.6
- .NET 6

# `.NET`をインストール

https://dotnet.microsoft.com/ja-jp/download/dotnet

こちらからマシンアーキテクチャに合わせたインストーラをDLする

# コマンドが使えるか確認

```terminal
$ dotnet --list-sdks
6.0.404 [/usr/local/share/dotnet/sdk]

$ dotnet --list-runtimes
Microsoft.AspNetCore.App 6.0.12 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
Microsoft.NETCore.App 6.0.12 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
```

## エラーが出ればPATHを通す

多分これをPATHに加えればいいかな？

`/usr/local/share/dotnet`
`~/.dotnet/tools`

# VSCodeを起動してサンプルプロジェクトを作っていく

## C#拡張機能のインストール

```terminal
$ code --install-extension ms-dotnettools.csharp
```

↓これです

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/7691e714-c5e1-4d88-e3a6-87c49e7315cf.png" width="350">


## ディレクトリの作成

```terminal
$ mkdir csharp-sample
$ cd csharp-sample 
```

## サンプルプロジェクトの作成

```terminal
$ dotnet new console --use-program-main 
テンプレート "コンソール アプリ" が正常に作成されました。
```

作成されたもの↓
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/d3e27e12-0f05-a0c6-8373-cc8874a2ec46.png" width="350">

```Program.cs
namespace csharp_sample;
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
    }
}
```

## Formatterの設定

### デフォルトフォーマッタの設定

- `settings.json`(User Settingのヤツ、グローバルに設定したいならこっち)
- `[project root]/.vscode/settings.json`(リポジトリのみの設定)

に下記を記載

```settings.json
{
  "editor.formatOnSave": true,
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp"
  },
}
```

### (任意)フォーマットルールを設定

> - An omnisharp.json file located in %USERPROFILE%/.omnisharp/
> - An omnisharp.json file located in the working directory which OmniSharp has been pointed at

[OmniSharp(便利ツール的なヤツらしい)公式GitHub](https://github.com/OmniSharp/omnisharp-roslyn/wiki/Configuration-Options)で言及されてる通り、`omnisharp.json`を上記どちらかに作る
ファイル形式や設定値についてはこちらのサイトを参考されたし

https://github.com/OmniSharp/omnisharp-roslyn/wiki/Configuration-Options#formatting-options

https://tbk-memorandum.com/omnisharp-formatting-options/

#### 自動フォーマットチェック

インデントをめちゃくちゃにしても、`Cmd + S`で整形される

```Program.cs
namespace csharp_sample;
class Program {
    static void Main(string[] args) {
        Console.WriteLine("Hello, World!");
    }
}
```

JavaやTypeScriptで慣れてると、C#に特徴的な始まりのカッコ`{`を改行するのは違和感あるので、設定変えてみた

#### うまく動かないときは

`Cmd + Shift + P`でコマンドパレットを表示し、`Restart OmniSharp`を選択する

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/3c1651a7-26e1-e9f2-c4e3-0154d445185c.png" width="700">

あとはVSCodeの再起動などトライしてみて

# 動かしてみよう

```terminal
$ dotnet run
Hello, World!
```

# デバッグ

## `.vscode`配下に以下内容を記述

```jsonc:launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      // Use IntelliSense to find out which attributes exist for C# debugging
      // Use hover for the description of the existing attributes
      // For further information visit https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md
      "name": ".NET Core Launch (console)",
      "type": "coreclr",
      "request": "launch",
      "preLaunchTask": "build",
      // If you have changed target frameworks, make sure to update the program path.
      "program": "${workspaceFolder}/bin/Debug/net6.0/csharp-sample.dll",
      "args": [],
      "cwd": "${workspaceFolder}",
      // For more information about the 'console' field, see https://aka.ms/VSCode-CS-LaunchJson-Console
      "console": "internalConsole",
      "stopAtEntry": false
    },
    {
      "name": ".NET Core Attach",
      "type": "coreclr",
      "request": "attach"
    }
  ]
}
```

```tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build",
      "command": "dotnet",
      "type": "process",
      "args": [
        "build",
        "${workspaceFolder}/csharp-sample.csproj",
        "/property:GenerateFullPaths=true",
        "/consoleloggerparameters:NoSummary"
      ],
      "problemMatcher": "$msCompile"
    },
    {
      "label": "publish",
      "command": "dotnet",
      "type": "process",
      "args": [
        "publish",
        "${workspaceFolder}/csharp-sample.csproj",
        "/property:GenerateFullPaths=true",
        "/consoleloggerparameters:NoSummary"
      ],
      "problemMatcher": "$msCompile"
    },
    {
      "label": "watch",
      "command": "dotnet",
      "type": "process",
      "args": [
        "watch",
        "run",
        "--project",
        "${workspaceFolder}/csharp-sample.csproj"
      ],
      "problemMatcher": "$msCompile"
    }
  ]
}
```

## `F5`押下でデバッグスタート

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/647946/22495a86-b767-8f0b-b5ab-d0888c9fda75.png" width="900">

ちゃんとブレークポイントで止まってる

# 参考にしたサイト

https://ascii.jp/elem/000/004/038/4038170/
