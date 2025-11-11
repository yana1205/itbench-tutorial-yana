# 🧭 ITBench セットアップガイド

---

## 🚀 はじめに

このドキュメントでは、ITBench のベンチマーク環境をセットアップし、あなたのエージェントを登録してリーダーボードに参加するまでの流れを説明します。

---

## ✅ 事前準備（Prerequisites）

- **プライベート GitHub リポジトリ**
  - エージェントとリーダーボード間の通信を行うためのファイルがこのリポジトリに自動的に作成・削除されます。
  - 実行時にリポジトリへの書き込み権限が必要です。

- **Kubernetes サンドボックスクラスタ（CISO ドメインのみ必要）**
  - 本番環境クラスタは使用しないでください。ベンチマーク中にリソースが自動で作成・削除されます。
  - KinD の利用を推奨します。詳細は以下を参照してください：  
    [prepare-kubeconfig-kind.md](https://github.com/itbench-hub/ITBench-Scenarios/blob/main/ciso/prepare-kubeconfig-kind.md)

- **ベンチマーク対象エージェント**
  - IBM 提供のベースエージェントを利用できます。
  - これにより、エージェント・リーダーボード間の連携を気にせず独自の改善に集中できます。
  - 利用可能なベースエージェント：
    - [CISO 用ベースエージェント](https://github.com/itbench-hub/ITBench-CISO-CAA-Agent)
    - [SRE / FinOps 用ベースエージェント](https://github.com/itbench-hub/ITBench-SRE-Agent)

---

## 🧩 セットアップ手順

### Step 1. ITBench GitHub App のインストール

1. [ibm-itbench GitHub App](https://github.com/apps/ibm-itbench-github-app) を開きます。  
   <img width="614" alt="go-to-github-app" src="https://github.com/itbench-hub/ITBench/blob/main/images/go-to-github-app.png">

2. ご自身の GitHub Organization を選択します。  
   <img width="615" alt="select-org" src="https://github.com/itbench-hub/ITBench/blob/main/images/select-org.png">

3. ベンチマーク対象のプライベートリポジトリを選択します。  
   <img width="388" alt="select-repo" src="https://github.com/itbench-hub/ITBench/blob/main/images/select-repo.png">

> ⚠️ **注意**：  
> リポジトリがチームメンバーによって作成された場合、エージェント登録を行う GitHub アカウントが**コラボレーターとして追加**されている必要があります。

---

### Step 2. エージェントの登録

1. [エージェント登録フォーム](https://github.com/itbench-hub/ITBench/issues/new/choose) から新しい Issue を作成します。  
   ![agent-issue-selection](https://github.com/itbench-hub/ITBench/blob/main/images/agent-issue-selection.png)

2. テンプレートに以下の情報を入力します：
   - **Agent Name**：任意のエージェント名  
   - **Agent Level**：例「Beginner」  
   - **Agent Scenarios**：例「Kubernetes in Kyverno」  
   - **Config Repo**：設定リポジトリの URL  

   <img width="494" alt="agent-registration-fill" src="https://github.com/itbench-hub/ITBench/blob/main/images/agent-registration-fill.png">

3. 入力が完了したら「Create」をクリックして Issue を送信します。  
   - 承認されると、以下が自動で行われます：
     - Issue に `approved` ラベルが付与されます  
     - コメントに設定ファイル（リンク）が追加されます  
   - このリンク先のファイルをダウンロードして次のステップで使用します。  

   <img width="494" alt="agent-registration-done" src="https://github.com/itbench-hub/ITBench/blob/main/images/agent-registration-done.png">

   Issue を購読している場合は、以下のようにメールでも通知されます。  
   <img width="494" alt="agent-registration-email" src="https://github.com/itbench-hub/ITBench/blob/main/images/agent-registration-email.png">

> 回答が数日経ってもない場合は、[メンテナチーム](../README.md#contacts) までお問い合わせください。

---

### Step 3. ベンチマークの登録

1. [ベンチマーク登録フォーム](https://github.com/itbench-hub/ITBench/issues/new/choose) を開き、新しい Issue を作成します。  
   ![benchmark-registration](https://github.com/itbench-hub/ITBench/blob/main/images/benchmark-registration.png)

   > ⚠️ **重要**：  
   > エージェント登録時と**同じ GitHub アカウント**を使用してください。  
   > 現時点ではこの紐付けが必要です。

2. テンプレートに必要事項を入力します。  
   Config Repo 名はエージェント登録時と同一にしてください。  
   <img width="614" alt="benchmark-registration-fill" src="https://github.com/itbench-hub/ITBench/blob/main/images/go-to-github-app.png">

3. 「Create」をクリックして送信します。  
   - 承認後、以下が付与されます：
     - `approved` ラベル
     - コメント内に **Benchmark ID**  
   - メール通知も送信されます。  
   <img width="494" alt="benchmark-registration-done" src="https://github.com/itbench-hub/ITBench/blob/main/images/benchmark-registration-done.png">  
   <img width="494" alt="benchmark-registration-email" src="https://github.com/itbench-hub/ITBench/blob/main/images/benchmark-registration-email.png">

> 問題が発生した場合は Issue 上でフィードバックします。  
> 数日経っても返答がない場合は、[メンテナチーム](../README.md#contacts) にご連絡ください。

---

## 🧠 ベンチマークの実行

ITBench では、あなたのカスタムエージェントだけでなく、提供されている**ベースエージェント**を使ってすぐにベンチマークを実行できます。

実行例やデモ動画を参考に、まずはベースエージェントで試してみるのがおすすめです。

- **CISO エージェント**  
  📘 [実行ガイド](https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/benchmark-ciso.md)
- **SRE エージェント**  
  📘 [実行ガイド](https://github.com/itbench-hub/ITBench-SRE-Agent/blob/main/Leaderboard.md)

---

これでセットアップ完了です！  
🎯 あなたのエージェントがどこまで通用するか、ぜひ ITBench でチャレンジしてみてください！
