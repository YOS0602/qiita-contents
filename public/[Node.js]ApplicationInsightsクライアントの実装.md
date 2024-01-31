---
title: '[Node.js]ApplicationInsightsクライアントの実装'
tags:
  - Node.js
  - Azure
  - AzureFunctions
  - ApplicationInsights
private: false
updated_at: '2022-12-06T12:19:28+09:00'
id: 2a7493b3daf1b6912d18
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Azureでは、ログ蓄積サービスとして[ApplicationInsights](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/app-insights-overview)がある。(AWSのCloudWatch的なヤツ)
自分が今思う最適な実装を書き残しておく。

# 環境

開発環境: macOS 12.4
ランタイム: Node16
作成するサービス: Azure Functions

# 実装

```src/utils/applicationInsights.ts
import { TelemetryClient } from 'applicationinsights'

/**
 * ApplicationInsightクラス
 * ApplicationInsightクライアントをシングルトンで管理する
 */
export class ApplicationInsight {
  private static _applicationInsight: ApplicationInsight // 自クラスのインスタンスを保持するクラス変数
  private static _client: TelemetryClient // ApplicationInsightクライアント
  private static _initialized = false // ApplicationInsightがインスタンス化されているか

  /**
   * ApplicationInsightインスタンスを作成する
   * アクセス修飾子をprivateとし、インスタンスが単一を保つようシングルトンで作成
   */
  private constructor() {
    ApplicationInsight._client = new TelemetryClient(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING)
    ApplicationInsight._initialized = true
  }

  /**
   * ApplicationInsightクライアントをreturnする
   * @returns ApplicationInsightクライアント
   */
  public static getClient(): TelemetryClient {
    // _initialized済で、_clientがnullやundefinedではないなら_clientをreturn
    if (ApplicationInsight._initialized && ApplicationInsight._client) {
      return ApplicationInsight._client
    }

    // インスタンスが存在しない時はインスタンス化
    ApplicationInsight._applicationInsight = new ApplicationInsight()
    return ApplicationInsight._client
  }
}
```

# 処理からの参照

```src/utils/logger/logger.ts
import { TelemetryClient } from 'applicationinsights'
import { ApplicationInsight } from '../applicationInsights'

export class Logger {
  // applicationInsightへの接続クライアント
  private static _client: TelemetryClient = ApplicationInsight.getClient()

  /**
   * 深刻度: インフォメーションのログを発行
   * @param msgId メッセージID
   * @param methodName 呼び出し元のメソッド名
   * @param stack スタックトレース
   * @return void
   */
  public static logInfo(msgId: string, methodName: string, stack = ''): void {
    // applicationInsightへのログ出力を行う。
    Logger._client.trackEvent({
      name: 'applicationLog',
      properties: {
        eventTimeJST: getJst().format('YYYY-MM-DD HH:mm:ss.SSS'), // getJst()は現在日時(JST)を取得する共通処理
        logLevel: 'INFO',
        methodName: methodName,
        messageId: msgId,
        message: msg,
        stack: stack,
      },
    })
  }
}
```

# 課題

テスト実行後、`client`が完全に破棄されきっていないらしく、接続をまだ保持し続けてしまっているっぽく、Jestが正常に終了しない

```terminal
Test Suites: 5 passed, 5 total
Tests:       27 skipped, 109 passed, 136 total
Snapshots:   0 total
Time:        10.896 s, estimated 11 s
Ran all test suites.
Jest did not exit one second after the test run has completed.

This usually means that there are asynchronous operations that weren't stopped in your tests. Consider running Jest with `--detectOpenHandles` to troubleshoot this issue.
```

DBのコネクションクローズとか、
puppeteerの`await browser.close()`とかみたいに
applicationinsightsのクライアントを破棄する良い方法はないものか……
