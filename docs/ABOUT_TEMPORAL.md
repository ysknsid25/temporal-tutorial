# Temporalとは

**Temporal**は、分散システムにおける複雑で長時間実行されるワークフローを管理するためのプラットフォームです。

## 主な用途

1. **Sagaパターンの実装** - 分散トランザクションの管理
2. **長時間実行ワークフロー** - 数分から数日、数ヶ月かかる処理
3. **信頼性の高いタスク処理** - 失敗時の自動リトライ、状態管理
4. **複雑なビジネスロジック** - 条件分岐、ループ、並列処理を含むワークフロー

## Sagaパターンとの関係

Temporalは確かにSagaパターンを実現する優れたツールですが、それ以上の機能を提供します。

Sagaパターンについては、[この記事](https://zenn.dev/farstep/articles/saga-pattern)がほどよくまとまっている。

### Sagaパターンでできること
- 分散トランザクションの管理
- 補償トランザクション（Compensation）の自動実行
- 失敗時のロールバック

### Temporalでさらにできること
- **タイムアウト管理**: 長時間の待機やスケジューリング
- **状態の永続化**: ワーカーが停止してもワークフローの状態が保持される
- **可視性**: Web UIでワークフローの実行状況をリアルタイム監視
- **バージョニング**: ワークフローコードの安全な更新
- **テスト支援**: ワークフローのユニットテスト機能

## 具体例: 注文処理ワークフロー（Sagaパターン）

```go
// Sagaパターンの例: 注文処理ワークフロー
func OrderWorkflow(ctx workflow.Context, order Order) error {
    // 1. 在庫予約
    var inventoryResult InventoryResult
    err := workflow.ExecuteActivity(ctx, ReserveInventory, order).Get(ctx, &inventoryResult)
    if err != nil {
        return err
    }

    // 2. 決済処理
    var paymentResult PaymentResult
    err = workflow.ExecuteActivity(ctx, ProcessPayment, order).Get(ctx, &paymentResult)
    if err != nil {
        // 失敗時: 在庫予約をキャンセル（補償トランザクション）
        workflow.ExecuteActivity(ctx, CancelInventoryReservation, inventoryResult)
        return err
    }

    // 3. 配送手配
    err = workflow.ExecuteActivity(ctx, ArrangeShipping, order).Get(ctx, nil)
    if err != nil {
        // 失敗時: 決済と在庫予約をキャンセル
        workflow.ExecuteActivity(ctx, RefundPayment, paymentResult)
        workflow.ExecuteActivity(ctx, CancelInventoryReservation, inventoryResult)
        return err
    }

    return nil
}
```

## まとめ

**TemporalはSagaパターンを含む、より包括的な分散システム管理プラットフォーム**です。

単純な分散トランザクションだけでなく、複雑で長時間実行されるビジネスプロセス全体を管理することができる強力なツールです。

## 主要な概念

- **Workflow**: ビジネスロジックを定義する関数
- **Activity**: 実際の作業を行う関数（外部API呼び出し、データベース操作など）
- **Worker**: ワークフローとアクティビティを実行するプロセス
- **Task Queue**: ワーカーがタスクを受け取るためのキュー

これらの概念を組み合わせることで、信頼性が高く、スケーラブルな分散アプリケーションを構築できます。

## Orchestration方式 vs Choreography方式

分散システムにおけるサービス間の協調には、主に2つのアプローチがあります。

### Choreography方式（分散型）
- 各サービスが自分の役割を知っていて、イベントに反応して次のサービスを呼び出す
- 中央管理者がいない
- サービス間でイベントをやり取りして協調する
- 例：注文サービスが注文完了イベントを発行 → 在庫サービスがそれを受信して在庫更新イベントを発行 → 決済サービスがそれを受信して決済処理

### Orchestration方式（中央集権型）
- **中央のオーケストレーター**がワークフロー全体を管理
- オーケストレーターが各サービス（Activity）を順次呼び出す
- ワークフローの実行順序や条件分岐を一箇所で管理

## TemporalはOrchestration方式

**TemporalはOrchestration方式**を採用しています。

### 理由1: Workflowが中央管理者の役割

```go
func OrderWorkflow(ctx workflow.Context, order Order) error {
    // Workflowが全体の流れを制御
    workflow.ExecuteActivity(ctx, ReserveInventory, order)
    workflow.ExecuteActivity(ctx, ProcessPayment, order)
    workflow.ExecuteActivity(ctx, ArrangeShipping, order)
}
```

### 理由2: 各Activityは自分の仕事だけを知っている

```go
func ReserveInventory(ctx context.Context, order Order) error {
    // 在庫予約の処理のみ。次に何をするかは知らない
    return reserveItems(order.Items)
}
```

### 理由3: ワークフローエンジンが実行を管理

- Temporal Serverが各ステップの実行状況を追跡
- 失敗時の再試行やエラーハンドリングを制御
- 状態の永続化と復旧を管理

### Orchestration方式のメリット

- **可視性が高い**: ワークフロー全体が一箇所で定義される
- **デバッグしやすい**: 実行状況をTemporal Web UIで確認可能
- **変更が容易**: ワークフローの変更が一箇所で済む
- **テストしやすい**: ワークフロー全体のテストが書きやすい

### Orchestration方式のデメリット

- **中央集権的**: オーケストレーターが単一障害点になる可能性
- **結合度が高い**: ワークフローが各サービスの詳細を知る必要がある

ただし、Temporalの場合は分散アーキテクチャと高可用性設計により、これらのデメリットは大幅に軽減されています。