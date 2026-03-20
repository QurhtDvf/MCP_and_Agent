# 🤖 A2A Demo — Google Colab + HuggingFace Transformers

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![HuggingFace](https://img.shields.io/badge/🤗%20Transformers-4.x-FFD21E?style=flat-square)](https://huggingface.co/docs/transformers)
[![Model](https://img.shields.io/badge/Model-Qwen2.5--0.5B--Instruct-blueviolet?style=flat-square)](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)
[![A2A](https://img.shields.io/badge/Protocol-A2A%20v1.0--rc-4285F4?style=flat-square&logo=google&logoColor=white)](https://github.com/a2aproject/A2A)
[![Linux Foundation](https://img.shields.io/badge/Governed%20by-Linux%20Foundation-003366?style=flat-square&logo=linux&logoColor=white)](https://www.linuxfoundation.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green?style=flat-square)](LICENSE)
[![Open in Colab](https://img.shields.io/badge/Open%20in-Colab-F9AB00?style=flat-square&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Stars](https://img.shields.io/github/stars/a2aproject/A2A?style=flat-square&logo=github&label=A2A%20Stars)](https://github.com/a2aproject/A2A)

Google の **Agent2Agent (A2A) プロトコル** を、HuggingFace Transformers によるローカル LLM を用いて Google Colab 上で動かすデモノートブックです。エージェントカードの発行・取得から、`tasks/send` / `tasks/get` によるエージェント間委譲、チェーン実行まで、A2A 仕様の主要要素を一通り体験できます。

---

## 📋 目次

- [A2A とは何か — 歴史と設計思想](#a2a-とは何か--歴史と設計思想)
- [エージェントカードの詳細解説](#エージェントカードの詳細解説)
  - [AgentCard とは何か](#agentcard-とは何か)
  - [AgentCard の全フィールド](#agentcard-の全フィールド)
  - [AgentSkill とは何か](#agentskill-とは何か)
  - [AgentCard と AgentSkill の関係](#agentcard-と-agentskill-の関係)
  - [公開・取得・セキュリティ](#公開取得セキュリティ)
  - [拡張エージェントカード](#拡張エージェントカード)
- [SKILLS.md / AGENTS.md との関係整理](#skillsmd--agentsmd-との関係整理)
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

2024 年後半から、LLM を核とする自律エージェントが企業環境に急速に普及し始めた。LangChain・CrewAI・AutoGen といったフレームワークはそれぞれ独自のマルチエージェント機構を持つが、それらは「自分のエコシステムの中だけで完結する」設計であった。Salesforce のエージェントと SAP のエージェントが協調しようとしても共通の通信言語がなく、ベンダーごとの独自実装による脆弱な統合が繰り返されていた。A2A はこの「エージェント間の方言問題」に対する業界横断的な解として登場した。

### 2025年4月9日：Google Cloud Next での発表

**2025年4月9日**、Google は Google Cloud Next カンファレンスにおいて **Agent2Agent (A2A) プロトコル** を発表した。発表時点ですでに Atlassian・Box・Cohere・LangChain・PayPal・Salesforce・SAP・ServiceNow・Workday など **50社以上** の技術パートナー、および Accenture・Deloitte・McKinsey などの支持を取り付けており、業界横断的な取り組みとして登場した。

A2A は Anthropic の **Model Context Protocol (MCP)** と相互補完的な関係として設計された。

```
┌───────────────────────────────────────────────┐
│        AI エージェント (Claude, Gemini 等)         │
├───────────────────────────────────────────────┤
│  A2A ── エージェント同士の通信・委譲（本プロトコル）   │
├───────────────────────────────────────────────┤
│  MCP ── エージェントとツール・データソースの接続      │
└───────────────────────────────────────────────┘
```

### 設計の5原則

| 原則 | 内容 |
|---|---|
| **エージェント的能力の尊重** | 内部メモリ・ツール・実装を公開せず、不透明なサービスとして協調できる |
| **既存標準への準拠** | HTTP・SSE・JSON-RPC 2.0 という実績ある標準の上に構築 |
| **セキュアバイデフォルト** | OpenAPI 相当の企業グレード認証・認可を初期仕様から組み込む |
| **長時間タスクへの対応** | 数秒〜人間レビューを挟む数日単位のタスクを同一プロトコルで扱える |
| **モダリティに依存しない** | テキスト・ファイル・フォーム・ストリームを `parts` として統一的に扱う |

### 2025年6月23日：Linux Foundation への寄贈

発表からわずか2ヶ月半後、Google は A2A プロトコルを **Linux Foundation** に寄贈し、**Agent2Agent (A2A) Project** として独立したオープンソースプロジェクトを発足させた。寄贈時点でサポート企業は **100社以上** に拡大し、AWS・Microsoft・SAP なども参加コメントを発表した。

### 2025年7月31日：v0.3 → 現在：v1.0-rc

主な追加要素:

- **gRPC サポート** — JSON-RPC over HTTP に加え、低レイテンシな gRPC 通信が可能に
- **署名付きセキュリティカード** — エージェントカードへの JWS 署名によるなりすまし防止
- **OpenTelemetry 対応** — OTLP 形式でメトリクスを出力
- **仕様の規範的ソースを `a2a.proto` に統一** — Protocol Buffers を唯一の真実の源泉とし、JSON Schema・各言語 SDK を自動生成する設計に

---

## エージェントカードの詳細解説

### AgentCard とは何か

AgentCard は A2A サーバーが `/.well-known/agent-card.json` で公開する JSON メタデータドキュメントであり、エージェントのアイデンティティ・能力・スキル・エンドポイント・認証要件を記述する。エージェントの「デジタル名刺」として機能し、以下の4つを実現する。

- **自動発見** — 他のエージェントや人間がエージェントを発見・理解できる
- **認証ネゴシエーション** — クライアントが使用すべき認証スキームを判断できる
- **能力検証** — 接続前にエージェントが何をできるかを把握できる
- **プロトコル互換性確認** — 互換性のあるプロトコルで通信できることを保証する

### AgentCard の全フィールド

```jsonc
// /.well-known/agent-card.json の完全な例
{
  // ── 基本情報（必須） ──────────────────────────────
  "name": "Enterprise Data Analysis Agent",
  "description": "企業データの分析とレポート生成を行うエージェント",
  "url": "https://api.example.com/a2a/",
  "version": "2.1.3",
  "provider": {
    "name": "Example Corp",
    "url": "https://example.com",
    "support_contact": "support@example.com"
  },

  // ── 能力（Capabilities） ──────────────────────────
  "capabilities": {
    "streaming": true,               // SSE によるストリーミング対応
    "pushNotifications": true,       // Webhook によるプッシュ通知対応
    "stateTransitionHistory": true,  // タスク履歴の保持
    "extendedAgentCard": true        // 認証済み拡張カードの提供
  },

  // ── デフォルトの入出力モダリティ ──────────────────
  "defaultInputModes":  ["text/plain", "application/json"],
  "defaultOutputModes": ["application/json", "text/html"],

  // ── 認証スキーム ──────────────────────────────────
  "securitySchemes": {
    "google": {
      "openIdConnectSecurityScheme": {
        "openIdConnectUrl": "https://accounts.google.com/.well-known/openid-configuration"
      }
    }
  },

  // ── スキル一覧 ────────────────────────────────────
  "skills": [
    {
      "id": "data_analysis",
      "name": "Data Analysis",
      "description": "CSV/JSON データを統計解析し、インサイトを生成する",
      "tags": ["analytics", "statistics"],
      "examples": ["売上データを四半期ごとに集計して", "異常値を検出して"],
      "inputModes":  ["application/json", "text/csv"],
      "outputModes": ["application/json", "text/html"]
    },
    {
      "id": "report_generation",
      "name": "Report Generation",
      "description": "分析結果を構造化されたレポートとして出力する",
      "tags": ["report", "document"],
      "inputModes":  ["application/json"],
      "outputModes": ["text/html", "application/pdf"]
    }
  ],

  // ── 対応プロトコルインターフェース ────────────────
  "supportedInterfaces": [
    { "transport": "jsonrpc", "url": "https://api.example.com/a2a/" },
    { "transport": "grpc",    "url": "https://api.example.com/a2a/grpc" }
  ]
}
```

各フィールドの役割:

| フィールド | 必須 | 役割 |
|---|:---:|---|
| `name` | ✅ | エージェントの表示名 |
| `description` | ✅ | エージェントの概要説明 |
| `url` | ✅ | A2A エンドポイントの URL |
| `version` | ✅ | エージェントソフトウェアのバージョン |
| `provider.name` | ✅ | 提供者名 |
| `capabilities` | ✅ | 対応機能のフラグ群 |
| `defaultInputModes` | ✅ | デフォルト入力 MIME タイプ |
| `defaultOutputModes` | ✅ | デフォルト出力 MIME タイプ |
| `skills` | ✅ | スキル定義の配列（最低1つ） |
| `securitySchemes` | 任意 | 認証スキームの定義 |
| `supportedInterfaces` | 任意 | 対応トランスポートの宣言 |

### AgentSkill とは何か

AgentSkill はエージェントが実行できる特定の能力を記述する。エージェントがどのような種類のタスクに適しているかをクライアントに伝えるための構成要素であり、`id` / `name` / `description` / `tags` / `examples` / モダリティから成る。

```python
skill = AgentSkill(
    id="summarize",
    name="Text Summarization",
    description="テキストを3行の箇条書きに要約する",  # LLM がシステムプロンプトで使う
    tags=["nlp", "summarization"],
    examples=["この記事を要約して", "重要なポイントだけ教えて"],
    inputModes=["text/plain"],
    outputModes=["text/plain"],
)
```

スキル設計のベストプラクティス:

- **単一責任** — 1スキルは1機能に集中させる
- **明確な description** — オーケストレーター LLM がシステムプロンプトに埋め込むため、自然言語で正確に記述する
- **具体的な examples** — 「このスキルで解けるか」の判断材料になる
- **適切な tags** — レジストリ検索性を高める

> ⚠️ **セキュリティ注意**: `name`・`description`・`skills.description` フィールドに細工されたデータを埋め込むプロンプトインジェクション攻撃に注意。外部エージェントのカード情報をサニタイズせずに LLM プロンプトへ渡すことは避けなければならない。

### AgentCard と AgentSkill の関係

AgentCard と AgentSkill は **「封筒と手紙」** の関係にある。AgentCard がエージェント全体を表す外側の宣言であり、AgentSkill がその中に含まれる個別能力の宣言である。

```
AgentCard（エージェント全体の宣言）
│
├── name / description / url / version     ← エージェントの基本情報
├── capabilities                            ← プロトコルレベルの対応機能
├── defaultInputModes / defaultOutputModes ← エージェント全体のデフォルト
├── securitySchemes                         ← 認証情報
│
└── skills[]                                ← スキルの配列
     ├── AgentSkill: summarize              ← 個別スキル①
     ├── AgentSkill: translate_ja           ← 個別スキル②
     └── AgentSkill: extract_keywords       ← 個別スキル③
          ├── id / name / description       ← スキルの識別・説明
          ├── tags / examples               ← 発見性と LLM ヒント
          └── inputModes / outputModes      ← スキル固有のモダリティ
               （省略時は AgentCard のデフォルト値を継承）
```

### 公開・取得・セキュリティ

```
GET /.well-known/agent-card.json   （現行仕様 v1.0-rc）
GET /.well-known/agent.json        （旧称・互換のため残存）
```

AgentCard は RFC 7515 に定義された JSON Web Signature (JWS) を用いてデジタル署名でき、クライアントはカードが改ざんされておらず、主張された提供者から発行されたことを検証できる。

### 拡張エージェントカード

公開カードの `capabilities.extendedAgentCard: true` が宣言されている場合、認証済みクライアントのみが追加スキル・詳細設定を含む拡張カードを取得できる。

```
# 公開カード（誰でも取得可能）
GET /.well-known/agent-card.json
→ 基本情報 + 公開スキルのみ

# 拡張カード（認証済みクライアントのみ）
GET /extendedAgentCard
Authorization: Bearer <token>
→ 公開情報 + 内部スキル + 詳細な設定情報
```

---

## SKILLS.md / AGENTS.md との関係整理

A2A の AgentCard と、`SKILLS.md`・`AGENTS.md` はそれぞれ**別の問題を解くための別のレイヤー**にある。いずれも「エージェントに指示・能力を与える」という問題意識を共有しているが、対象・スコープ・エコシステムが異なる。

### SKILLS.md（Anthropic / Claude Code 専用）

`SKILLS.md` は **Anthropic が Claude Code 専用に策定したエージェントスキルフレームワーク**である。Claude が繰り返し実行する複雑なタスク（PDFフォームの記入、コード生成のパターンなど）を「スキル」としてパッケージ化し、再利用可能にする仕組みである。

構造上の特徴:

```
skills/
└── my_skill/
     ├── SKILL.md       ← YAMLフロントマター（name / description）+ 指示文
     ├── script.py      ← バンドルされた実行コード
     └── reference.md   ← 参照ドキュメント
```

最大の特徴は**プログレッシブ・ディスクロージャー**（段階的情報開示）の設計にある。

```
レベル1: スキル一覧の name / description だけをスキャン
          ↓ タスクに一致した場合
レベル2: SKILL.md の完全な指示を読み込む
          ↓ 追加リソースが必要な場合
レベル3: バンドルされたコード・ドキュメントを取り込む
```

これにより、コンテキストウィンドウを圧迫せずに多数のスキルを管理できる。Claude Code の Tool Search Tool（遅延ロード）と同じ思想に基づいている。

**制約**: Claude / Anthropic エコシステム専用であり、他ベンダーのモデルへの移行は困難なベンダーロックインがある。

### AGENTS.md（オープン規約 / コーディングエージェント向け）

`AGENTS.md` は **Sourcegraph・OpenAI・Google などが共同で策定したオープン規約**であり、特定のコードベースで AI コーディングエージェントがどのように作業すべきかを記述するための**標準的なマークダウンファイル**である。どのエージェントでも読める「プロジェクトのチートシート」として機能する。

典型的な内容:

```markdown
# AGENTS.md

## Setup
npm install && npm run build

## Testing
npm test -- --coverage

## Code Style
- Use TypeScript strict mode
- Format with Prettier before commit

## Commit Messages
feat: / fix: / docs: プレフィックスを使用
```

**特徴**: 特定の AI ベンダーに依存しない。Claude Code・Cursor・Copilot・Gemini CLI など、どのツールでも同一ファイルを参照できる。ただし、エージェントに**新しい能力を与えることはできない**。既存能力の使い方を指示するルールブックに過ぎない。

### 3者の比較

| 観点 | A2A AgentCard | SKILLS.md | AGENTS.md |
|---|---|---|---|
| **策定主体** | Google → Linux Foundation（オープン） | Anthropic（プロプライエタリ） | Sourcegraph ほか複数社（オープン） |
| **対象** | エージェント同士の通信・発見 | Claude の再利用可能スキル | コードベース作業ルール |
| **スコープ** | ネットワーク越しのエージェント間 | 単一エージェントの能力拡張 | 単一リポジトリ内の作業標準 |
| **記述形式** | JSON（`/.well-known/agent-card.json`） | Markdown + YAML フロントマター（フォルダ構造） | Markdown（単一ファイル） |
| **動的/静的** | 動的（実行時に HTTP で取得） | 動的（必要に応じて段階的にロード） | 静的（エージェント起動時に読み込み） |
| **エコシステム依存** | なし（任意のA2A対応エージェント） | あり（Claude 専用） | なし（任意のコーディングエージェント） |
| **主なユースケース** | マルチエージェントシステムの相互運用 | 複雑な多段階タスクの専門化 | 開発ワークフローの標準化 |

### レイヤーとして見た整理

```
┌──────────────────────────────────────────────────────┐
│          A2A AgentCard                                │
│  エージェント間の発見・通信・能力宣言（ネットワーク層）    │
│  → 「このエージェントは何ができるか」を外部に伝える      │
├──────────────────────────────────────────────────────┤
│          SKILLS.md  (Claude Code)                    │
│  エージェント単体の能力パッケージング（実行層）           │
│  → 「このエージェントをどう強化するか」を内部で定義する   │
├──────────────────────────────────────────────────────┤
│          AGENTS.md                                   │
│  コードベース固有の作業ルール（コンテキスト層）           │
│  → 「このリポジトリでどう作業すべきか」を指示する         │
└──────────────────────────────────────────────────────┘
```

3者は競合ではなく**スタックとして組み合わせて使うことができる**。たとえば、`AGENTS.md` でリポジトリの作業ルールを定め、`SKILLS.md` でその作業を実行するスキルを定義し、A2A AgentCard でそのエージェントの能力を外部に公開するという構成が自然である。

---

## デモのアーキテクチャ

```
ユーザー
  └─→ OrchestratorAgent  (port 8100)
            │
            │  1. 起動時に /.well-known/agent.json を収集してレジストリ化
            │  2. ローカル LLM でルーティング判定
            │  3. tasks/send で委譲
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

### 1. ランタイムの設定（GPU 推奨）

```
Runtime > Change runtime type > T4 GPU
```

### 2. ノートブックを開く

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

### 3. セルを順番に実行

```
Step 1: ライブラリインストール            (~1分)
Step 2: Qwen2.5-0.5B-Instruct ロード  (~3分、初回のみダウンロード)
Step 3: A2A 型定義
Step 4: ワーカーエージェント起動（3体）
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
| **AgentSkill** | 各カードに `id` / `name` / `description` / モダリティを定義 |
| **エージェント発見** | オーケストレーターが起動時にカードを収集・レジストリ化 |
| **tasks/send** | タスクの委譲エンドポイント（POST） |
| **tasks/get** | タスク状態の非同期確認（POST） |
| **stateTransitionHistory** | user/agent/timestamp 付きのメッセージ履歴を記録 |
| **タスクチェーン** | 前エージェントの出力を次の入力に渡す連鎖実行 |
| **ローカル LLM** | Transformers (`Qwen2.5-0.5B-Instruct`) による推論 |

---

## デモシナリオ

### シナリオ 1: 要約（単体委譲）
`OrchestratorAgent → SummarizerAgent`

### シナリオ 2: キーワード抽出 → 翻訳チェーン
`OrchestratorAgent → KeywordAgent → TranslatorAgent`

### シナリオ 3: 要約 → 翻訳チェーン
`OrchestratorAgent → SummarizerAgent → TranslatorAgent`

### シナリオ 4: 直接呼び出し + tasks/get
`KeywordAgent（直接）→ tasks/get で状態確認`

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
MODEL_ID = "Qwen/Qwen2.5-1.5B-Instruct"
MODEL_ID = "google/gemma-3-1b-it"
```

**新しいワーカーを追加する**
```python
sentiment_card = AgentCard(
    name="SentimentAgent",
    description="Analyzes the sentiment of text as positive/negative/neutral.",
    url="http://localhost:8104",
    version="1.0.0",
    capabilities=AgentCapabilities(),
    skills=[AgentSkill(
        id="sentiment_analysis",
        name="Sentiment Analysis",
        description="Classifies text sentiment as positive, negative, or neutral",
        tags=["nlp", "sentiment"],
        examples=["このレビューは肯定的?", "感情を分析して"]
    )]
)
```

`discover_workers()` がカードを自動収集するため、ワーカーを追加するだけでオーケストレーターが認識する。

**現行仕様への対応（エンドポイントパスの修正）**
```python
# 旧称（このデモで使用）
@app.route("/.well-known/agent.json")

# 現行仕様 v1.0-rc
@app.route("/.well-known/agent-card.json")
```

---

## 参考リンク

**A2A プロトコル**
- [A2A 公式リポジトリ (GitHub)](https://github.com/a2aproject/A2A)
- [A2A 公式ドキュメント](https://a2a-protocol.org/)
- [Google Developers Blog — A2A 発表記事 (2025年4月9日)](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [Linux Foundation — A2A Project 発足 (2025年6月23日)](https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents)
- [A2A Protocol Specification (a2a.proto)](https://github.com/a2aproject/A2A/blob/main/specification/a2a.proto)

**SKILLS.md / AGENTS.md**
- [Anthropic — Agent Skills の概要](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Claude Code Skills ドキュメント](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Skills.md vs. Agents.md — eesel AI による比較解説](https://www.eesel.ai/ja/blog/skills-md-vs-agents-md)

**モデル・ライブラリ**
- [Qwen2.5-0.5B-Instruct (HuggingFace)](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)
- [awesome-a2a — A2A ツール・SDK・サンプル集](https://github.com/ai-boost/awesome-a2a)

---

## ライセンス

[Apache License 2.0](LICENSE)

A2A プロトコル仕様自体も Apache 2.0 ライセンスで公開されている。
