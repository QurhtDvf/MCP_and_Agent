# 🤖 A2A Demo — + HuggingFace Transformers

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![HuggingFace](https://img.shields.io/badge/🤗%20Transformers-4.x-FFD21E?style=flat-square)](https://huggingface.co/docs/transformers)
[![Model](https://img.shields.io/badge/Model-Qwen2.5--0.5B--Instruct-blueviolet?style=flat-square)](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)
[![A2A](https://img.shields.io/badge/Protocol-A2A%20v0.3-4285F4?style=flat-square&logo=google&logoColor=white)](https://github.com/a2aproject/A2A)
[![License](https://img.shields.io/badge/License-Apache%202.0-green?style=flat-square)](LICENSE)
[![Open in Colab](https://img.shields.io/badge/Open%20in-Colab-F9AB00?style=flat-square&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Linux Foundation](https://img.shields.io/badge/Governed%20by-Linux%20Foundation-003366?style=flat-square&logo=linux&logoColor=white)](https://www.linuxfoundation.org/)

Google の **Agent2Agent (A2A) プロトコル**を、HuggingFace Transformers によるローカル LLM を用いて Google Colab 上で動かすデモノートブックです。エージェントカードの発行・取得から、`tasks/send` / `tasks/get` によるエージェント間委譲、チェーン実行まで、A2A 仕様の主要要素を一通り体験できます。

---

## 📋 目次

- [A2A とは何か — 歴史と設計思想](#a2a-とは何か--歴史と設計思想)
- [デモのアーキテクチャ](#デモのアーキテクチャ)
- [クイックスタート](#クイックスタート)
- [実装した A2A 仕様要素](#実装した-a2a-仕様要素)
- [デモシナリオ](#デモシナリオ)
- [ファイル構成](#ファイル構成)
- [発展的な試み](#発展的な試み)
- [参考リンク](#参考リンク)

---

## A2A とは何か — 歴史と設計思想

### 背景：エージェント時代の「方言問題」

2024 年後半から、LLM を核とする自律エージェントが企業環境に急速に普及し始めた。LangChain、CrewAI、AutoGen といったフレームワークはそれぞれ独自のマルチエージェント機構を持つが、それらは「自分のエコシステムの中だけで完結する」設計であった。たとえば Salesforce のエージェントと SAP のエージェントが協調して受発注を自動化しようとしても、共通の通信言語がなく、ベンダーごとの独自実装による「点と点をつなぐ」脆弱な統合が繰り返されていた。Google はこの問題を「大規模マルチエージェントシステムを顧客向けに展開するなかで何度も繰り返し直面した課題」と表現している。

### 2025年4月9日：Google Cloud Next での発表

**2025年4月9日**、Google は Google Cloud Next カンファレンスにおいて **Agent2Agent (A2A) プロトコル**を発表した。発表時点ですでに Atlassian、Box、Cohere、Intuit、LangChain、MongoDB、PayPal、Salesforce、SAP、ServiceNow、Workday といった 50 社以上の技術パートナー、および Accenture、Deloitte、McKinsey などの大手コンサルティングファームの支持を取り付けており、単なる技術仕様の公開にとどまらない業界横断的な取り組みとして登場した。

A2A は Anthropic が 2024 年に発表した **Model Context Protocol (MCP)** と相互補完的な関係として設計された。MCP が「エージェントと外部ツール・データソースの接続」を担うのに対し、A2A は「エージェントとエージェントの対話」を担う。Google 自身も「A2A is an open protocol that complements MCP」と位置づけており、両者は競合ではなく役割分担の関係にある。

### 設計の5原則

Google がパートナー企業とともに策定した A2A の設計原則は以下の5点である。

| 原則 | 内容 |
|---|---|
| **エージェント的能力の尊重** | エージェントをツールに縛らず、記憶・ツール・コンテキストを共有しない状態でも自律的に協調できるよう設計する |
| **既存標準への準拠** | HTTP、SSE（Server-Sent Events）、JSON-RPC という実績ある標準の上に構築し、既存 IT スタックへの統合コストを下げる |
| **セキュアバイデフォルト** | OpenAPI の認証スキームと同等の企業グレード認証・認可を初期仕様から組み込む |
| **長時間タスクへの対応** | 数秒で完了するタスクから、人間のレビューを挟む数日単位のタスクまで同一プロトコルで扱える |
| **モダリティに依存しない** | テキスト・ファイル・フォーム・ストリームなど複数のコンテンツ形式を `parts` として統一的に扱う |

### コアコンセプト：エージェントカードとタスク

A2A の中心的な概念は2つである。

**エージェントカード (`/.well-known/agent.json`)**
各エージェントが HTTP で公開する JSON ファイルで、エージェントの名前・バージョン・エンドポイント URL・スキル一覧・対応モダリティ・認証方式を宣言する。IBM はこれを「AI エージェントの LinkedIn プロフィール」と表現している。クライアントエージェントはこのカードを取得することでリモートエージェントの能力を発見し、委譲するタスクの適合性を判断できる。

**タスク (`tasks/send` / `tasks/get`)**
エージェント間の通信は「タスク」オブジェクトを中心に設計されている。タスクには `submitted → working → completed / failed` というライフサイクルがあり、`tasks/send` でタスクを委譲し、`tasks/get` で非同期に状態を確認する。長時間タスクでは SSE によるストリーミング更新も可能である。タスクの出力は「アーティファクト」と呼ばれ、構造化された形で返される。

### 2025年6月23日：Linux Foundation への寄贈

発表からわずか2ヶ月半後の **2025年6月23日**、Google は A2A プロトコルを **Linux Foundation** に寄贈し、`Agent2Agent (A2A) Project` として独立したオープンソースプロジェクトを発足させた。これは単一ベンダーによる囲い込みを防ぎ、コミュニティ主導の標準化を加速するための判断であった。Linux Foundation への寄贈時点でサポート企業は 100 社以上に拡大しており、AWS（Swami Sivasubramanian 副社長）、Microsoft、SAP なども参加コメントを発表した。

### 2025年7月31日：v0.3 リリース

**2025年7月31日**、A2A v0.3 がリリースされた。主な追加要素は以下のとおりである。

- **gRPC サポート** — JSON-RPC over HTTP に加え、低レイテンシな gRPC 通信が可能に
- **署名付きセキュリティカード** — エージェントカードへの署名機能によりなりすましを防止
- **Python SDK の拡張** — クライアントサイドサポートの充実
- **OpenTelemetry 対応** — 各リクエスト/レスポンスにトレース ID が付与され、OTLP 形式でメトリクスを出力

この時点でサポート企業は **150 社以上** に拡大しており、Adobe、S&P Global、ServiceNow、Twilio といった企業による実際のユースケースも公開された。

### MCPとの住み分けと現状（2026年現在）

A2A は発表後に広く注目を集めた一方で、MCP の方が開発者コミュニティへの浸透が速かった。これは MCP が「既存の AI アシスタントと初日から連携できる」という即効性を持っていたのに対し、A2A が「新しいインフラを構築する必要がある」という参入コストの差が主因とされる。2026年現在、A2A は Linux Foundation の下で仕様として存続しており、企業グレードのマルチエージェントシステムを構築するための基盤として参照され続けている。

```
エージェントのレイヤー構造（現在の理解）

┌─────────────────────────────────────────┐
│           AI エージェント (Claude, Gemini 等)  │
├─────────────────────────────────────────┤
│     A2A — エージェント同士の通信・委譲           │
├─────────────────────────────────────────┤
│     MCP — エージェントとツール・データの接続       │
└─────────────────────────────────────────┘
```

---

## デモのアーキテクチャ

```
ユーザー
  └─→ OrchestratorAgent  (port 8100)
            │
            │  1. 起動時に /.well-known/agent.json を収集
            │  2. LLM でルーティング判定
            │  3. A2A tasks/send で委譲
            │  4. チェーン実行（前の出力を次の入力に）
            │
            ├─→ SummarizerAgent  (port 8101)  ← 3行箇条書き要約
            ├─→ TranslatorAgent  (port 8102)  ← 英→日翻訳
            └─→ KeywordAgent     (port 8103)  ← キーワード5語抽出
```

| エージェント | ポート | スキル ID | 説明 |
|---|---|---|---|
| OrchestratorAgent | 8100 | `orchestrate` | LLM によるルーティング・チェーン委譲 |
| SummarizerAgent | 8101 | `summarize` | テキストを3行の箇条書きに要約 |
| TranslatorAgent | 8102 | `translate_ja` | 英語テキストを日本語に翻訳 |
| KeywordAgent | 8103 | `extract_keywords` | 重要キーワードを5語抽出 |

---

## クイックスタート

### 1. ランタイムの設定

Google Colab を開き、**GPU ランタイムを選択**してください。

```
Runtime > Change runtime type > T4 GPU
```

### 2. ノートブックを開く

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

### 3. セルを順番に実行

```
Step 1: ライブラリインストール       (~1分)
Step 2: Qwen2.5-0.5B-Instruct ロード (~3分、初回のみダウンロード)
Step 3: A2A 型定義
Step 4: ワーカーエージェント起動
Step 5: エージェントカード取得・表示
Step 6: オーケストレーター起動
Step 7: 全エージェントカード一覧
Step 8: A2A デモ実行（3シナリオ）
Step 9: tasks/get による状態確認
```

### 動作要件

| 項目 | 要件 |
|---|---|
| ランタイム | Google Colab (T4 GPU 推奨、CPU でも動作) |
| Python | 3.10 以上 |
| 主要ライブラリ | `transformers`, `accelerate`, `flask`, `flask-cors` |
| モデル | `Qwen/Qwen2.5-0.5B-Instruct` (~1 GB) |

---

## 実装した A2A 仕様要素

| A2A 仕様要素 | 実装内容 |
|---|---|
| **AgentCard** | `/.well-known/agent.json` を各エージェントが HTTP で公開 |
| **エージェント発見** | オーケストレーターが起動時にカードを収集・レジストリ化 |
| **tasks/send** | タスクの委譲エンドポイント（POST） |
| **tasks/get** | タスク状態の非同期確認（POST） |
| **stateTransitionHistory** | user/agent/timestamp 付きのメッセージ履歴を記録 |
| **タスクチェーン** | 前エージェントの出力を次の入力に渡す連鎖実行 |
| **ローカル LLM** | Transformers (`Qwen2.5-0.5B-Instruct`) による推論 |

---

## デモシナリオ

### シナリオ 1: 要約（単体委譲）
```
OrchestratorAgent → SummarizerAgent
```
A2Aプロトコルについての英文テキストを3行の箇条書きに要約します。

### シナリオ 2: キーワード抽出 → 翻訳チェーン
```
OrchestratorAgent → KeywordAgent → TranslatorAgent
```
機械学習に関するテキストからキーワードを抽出し、そのキーワードリストを日本語に翻訳します。前エージェントの出力が次エージェントへの入力としてチェーンされます。

### シナリオ 3: 要約 → 翻訳チェーン
```
OrchestratorAgent → SummarizerAgent → TranslatorAgent
```
コンテキストウィンドウ効率化に関するテキストを要約し、その要約を日本語に翻訳します。

### シナリオ 4: ワーカーへの直接呼び出し + tasks/get
```
KeywordAgent (直接) → tasks/get で状態確認
```
オーケストレーターを介さず KeywordAgent を直接呼び出し、`stateTransitionHistory` とタスク状態確認の動作を確認します。

---

## ファイル構成

```
.
├── A2A_Demo_Transformers.ipynb   # メインノートブック
├── README.md                     # このファイル
└── LICENSE                       # Apache 2.0
```

---

## 発展的な試み

**モデルの変更**
```python
MODEL_ID = "Qwen/Qwen2.5-1.5B-Instruct"   # より高精度
MODEL_ID = "google/gemma-3-1b-it"          # Google 製モデルで比較
```

**新しいワーカーを追加する**
```python
sentiment_card = AgentCard(
    name="SentimentAgent",
    description="Analyzes sentiment of text.",
    url="http://localhost:8104",
    ...
)
```
`discover_workers()` がカードを自動収集するため、ワーカーを追加するだけでオーケストレーターが認識します。

**ストリーミング対応**

エージェントカードの `capabilities.streaming` を `true` に設定し、SSE (`text/event-stream`) でレスポンスをストリームするよう `tasks/send` エンドポイントを拡張することで、A2A のストリーミング仕様を試せます。

---

## 参考リンク

- [A2A プロトコル公式仕様 (GitHub)](https://github.com/a2aproject/A2A)
- [Google Developers Blog — A2A 発表記事 (2025年4月9日)](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [Google Cloud Blog — v0.3 リリース (2025年7月31日)](https://cloud.google.com/blog/products/ai-machine-learning/agent2agent-protocol-is-getting-an-upgrade)
- [Linux Foundation — A2A Project 発足 (2025年6月23日)](https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents)
- [IBM — What Is Agent2Agent (A2A) Protocol?](https://www.ibm.com/think/topics/agent2agent-protocol)
- [Qwen2.5-0.5B-Instruct (HuggingFace)](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)

---

## ライセンス

[Apache License 2.0](LICENSE)

A2A プロトコル仕様自体も Apache 2.0 ライセンスで公開されています。
