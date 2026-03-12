# 🤝 A2A マルチエージェント with Phi-3.5

Google Colab上で3つのAIエージェントが **A2A (Agent-to-Agent) プロトコル** でHTTP通信しながら協調するノートブックです。
登録不要・ビルド不要で動作します。

---

## A2Aプロトコルとは

### 概要

A2A（Agent-to-Agent）プロトコルは、異なるフレームワーク・ベンダー・プラットフォームで作られたAIエージェント同士が、**標準化されたHTTP通信で相互に連携するためのオープン規格**です。

HTTP、JSON-RPC 2.0、Server-Sent Events（SSE）という既存の標準技術の上に構築されており、既存のITインフラと親和性が高いのが特徴です。

### 歴史

| 時期 | 出来事 |
|------|--------|
| **2025年4月9日** | GoogleがGoogle Cloud Nextで A2A v0.1 を発表。Atlassian、Salesforce、SAP、LangChain、PayPalなど50社以上がローンチパートナーとして参加 |
| **2025年6月23日** | GoogleがA2Aの仕様・SDK・開発ツール一式をLinux Foundationに寄贈。特定ベンダーによるロックインを防ぐためのベンダーニュートラルなガバナンス体制へ移行 |
| **2025年7月31日** | A2A v0.3 リリース。gRPCサポート・署名付きAgentCard・Python SDKのクライアント拡張が追加。対応組織数が150以上に拡大。Tyson FoodsやGordon Food Serviceなど実企業での採用事例も公開 |
| **2025年8月** | IBMが推進していた類似規格「ACP（Agent Communication Protocol）」がA2Aに統合。IBMがTechnical Steering Committeeに参加 |
| **2025年12月** | AnthropicがMCP（Model Context Protocol）をLinux Foundation傘下の新組織「Agentic AI Foundation（AAIF）」に寄贈。A2AとMCPが同一の中立的なガバナンス体制のもとに置かれる方向へ |

### MCPとの関係

A2AはMCPとよく比較されますが、解決する問題のレイヤーが異なり、**補完関係**にあります。

```
MCPの役割（垂直方向）:
  エージェント ← → ツール・DB・API・ファイル
  「エージェントが外部ツールを使う」ための規格

A2Aの役割（水平方向）:
  エージェント ← → エージェント
  「エージェント同士が協調する」ための規格
```

実際の活用例として、在庫管理エージェントがMCPでデータベースにアクセスし、在庫不足を検知したら発注エージェントにA2Aで通知、発注エージェントが外部サプライヤーのエージェントとA2Aで交渉する、といった多層的な構成が想定されています。

### 技術的な特徴

**AgentCard（エージェント名刺）**
各エージェントは `/.well-known/agent.json` にJSONファイルを公開し、自分の名前・説明・スキル・認証方式を宣伝します。クライアントはこれを見てエージェントを動的に発見・選択できます。

**タスクライフサイクル管理**
タスクには固有のIDと状態（submitted / working / completed / failed）が定義されており、長時間タスクや人間の承認を挟むワークフローにも対応しています。

**セキュリティ**
エージェントは内部の実装・メモリ・モデルを相手に公開せず、メッセージのみをやりとりします。OpenAPI互換の認証スキームをサポートし、エンタープライズグレードのセキュリティを想定した設計です。

**マルチモーダル対応**
テキストだけでなく、ファイル・画像・音声・動画のストリーミングも規格上サポートしています。

### 現状と課題

A2Aは2025年を通じてエンタープライズ向けの機能を充実させてきた一方、開発者コミュニティではより学習コストが低いMCPが先行して普及しました。現時点ではMCPの方がエコシステムが大きく、A2Aはエンタープライズ・マルチエージェント用途での採用が中心です。ただし両者は競合ではなく補完関係であり、Linux Foundationのもとで共存する形が整いつつあります。

---

## このノートブックの構成

```
ユーザーのテーマ
       ↓
┌──────────────────────────────────────────┐
│  🔍 調査エージェント (localhost:5001)     │
│  A2AServer として起動                    │
└────────────────┬─────────────────────────┘
                 ↓ HTTP POST (A2Aプロトコル)
┌──────────────────────────────────────────┐
│  📊 分析エージェント (localhost:5002)     │
│  調査結果を受け取り分析                   │
│  → 追加調査依頼もA2Aで送信              │ ← エージェント間ディスカッション
└────────────────┬─────────────────────────┘
                 ↓ HTTP POST (A2Aプロトコル)
┌──────────────────────────────────────────┐
│  ✍️  執筆エージェント (localhost:5003)    │
│  全結果を統合して最終回答を執筆           │
└──────────────────────────────────────────┘
       ↓
    最終回答
```

---

## Pythonオブジェクト直接呼び出し版との違い

前バージョン（[multi_agent_discussion.ipynb](./multi_agent_discussion.ipynb)）と本バージョンの本質的な違いを示します。

### 通信方式

```python
# 前バージョン: Pythonオブジェクトを直接呼び出す（同一プロセス内）
result = analysis_agent.analyze(topic, research_result)

# 本バージョン: A2AプロトコルでHTTP POSTを送る（ネットワーク通信）
response = analysis_client.send_message(
    Message(content=TextContent(text=f'{topic}\n{research_result}'),
            role=MessageRole.USER)
)
```

### 比較表

| 項目 | Pythonオブジェクト版 | A2Aプロトコル版（本ノートブック） |
|------|---------------------|--------------------------------|
| 通信方式 | Pythonの関数呼び出し | HTTP POST（JSON-RPC 2.0） |
| エージェントの独立性 | 同一プロセス内 | 各エージェントが独立したHTTPサーバー |
| メッセージ形式 | 文字列・辞書 | A2A標準 Message / TextContent |
| AgentCard | なし | あり（名前・説明・URLを公開） |
| 異なる言語・FWとの連携 | 不可 | 可能（A2A互換なら何でもOK） |
| ネットワーク越しの連携 | 不可 | 可能（URLを変えるだけ） |
| スケールアウト | 困難 | エージェントを別サーバーに移すだけ |

### エージェント間通信のフロー

```
オーケストレーター
    │
    ├─ HTTP POST → 🔍 調査エージェント (localhost:5001)
    │                    │
    │              A2Aレスポンス返却
    │
    ├─ HTTP POST → 📊 分析エージェント (localhost:5002)
    │                    │
    │              内部でHTTP POST → 🔍 調査エージェント（追加調査依頼）
    │                    │
    │              A2Aレスポンス返却（追加調査結果を含む）
    │
    └─ HTTP POST → ✍️  執筆エージェント (localhost:5003)
                         │
                   A2Aレスポンス返却（最終回答）
```

---

## 動作環境

| 項目 | 内容 |
|------|------|
| 実行環境 | Google Colab |
| GPU | T4 (15GB VRAM) 推奨 |
| モデル | Phi-3.5-mini-instruct (Microsoft) |
| モデルサイズ | 約2.2GB (Q4_K_M量子化) |
| A2Aライブラリ | python-a2a |
| ライセンス | MIT |

---

## セットアップ

### 1. Colabでノートブックを開く

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

### 2. ランタイムをGPUに変更

`ランタイム` → `ランタイムのタイプを変更` → `T4 GPU` を選択

### 3. セルを順番に実行

| Step | 内容 | 備考 |
|------|------|------|
| Step 1 | llama-cpp-python & python-a2a インストール | 完了後 **ランタイム再起動** |
| Step 2 | GPU確認 & モデルダウンロード | 約3〜5分 |
| Step 3 | モデルロード & 共通関数定義 | 約1〜2分 |
| Step 4 | A2Aエージェントクラス定義 | 即時 |
| Step 5 | A2Aサーバー起動（3エージェント） | 即時 |
| Step 6 | AgentCard確認 | 即時 |
| Step 7 | オーケストレーター定義 | 即時 |
| Step 8 | 実行 | テーマによる |
| Step 9 | A2A通信ログ確認 | — |
| Step 10 | インタラクティブUI | — |

> ⚠️ Step 1のインストール後は必ずランタイムを再起動してから Step 2 以降を実行してください。

---

## 実行例

```
📋 テーマ: 機械学習

┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
📤 [オーケストレーター → 🔍 調査エージェント] A2Aメッセージ送信
  [🔍 調査] 受信: 機械学習
  [🔍 調査] 送信: 機械学習はデータからパターンを...
📥 [🔍 調査エージェント → オーケストレーター] A2Aレスポンス受信

┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
📤 [オーケストレーター → 📊 分析エージェント] A2Aメッセージ送信
  [📊 分析] 受信: テーマ: 機械学習 / 調査結果: ...
  [📊 分析] → 調査エージェントに追加調査をA2Aで送信: 深層学習との関係性について
  [🔍 調査] 受信: 深層学習との関係性について  ← エージェント間A2A通信
  [🔍 調査] 送信: 深層学習は機械学習の一分野...
📥 [📊 分析エージェント → オーケストレーター] A2Aレスポンス受信

┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
📤 [オーケストレーター → ✍️  執筆エージェント] A2Aメッセージ送信
📥 [✍️  執筆エージェント → オーケストレーター] A2Aレスポンス受信

✅ 最終回答
機械学習とは、データからパターンを自動的に学習するAI技術です...
```

---

## カスタマイズ

### エージェントを追加する

```python
class ReviewAgent(A2AServer):
    def __init__(self):
        agent_card = AgentCard(
            name='ReviewAgent',
            description='最終回答をレビューする',
            url='http://localhost:5004/a2a',
            version='1.0.0',
        )
        super().__init__(agent_card=agent_card)

    def handle_message(self, message: Message) -> Message:
        result = llm_chat([...], max_tokens=300)
        return Message(
            content=TextContent(text=result),
            role=MessageRole.AGENT,
            parent_message_id=message.message_id,
            conversation_id=message.conversation_id,
        )

start_agent(ReviewAgent(), port=5004)
review_client = A2AClient('http://localhost:5004/a2a')
```

### 外部のA2A互換エージェントと連携する

```python
# URLを変えるだけで外部エージェントと接続できる
external_client = A2AClient('https://external-agent.example.com/a2a')
response = external_client.send_message(
    Message(content=TextContent(text='質問内容'), role=MessageRole.USER)
)
```

---

## ライセンス

MIT License

---

## 参考

- [A2A Protocol 公式GitHub](https://github.com/a2aproject/a2a-samples)
- [Google Developers Blog: A2A発表（2025年4月）](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
- [Google Cloud Blog: A2A v0.3リリース（2025年7月）](https://cloud.google.com/blog/products/ai-machine-learning/agent2agent-protocol-is-getting-an-upgrade)
- [Linux Foundation: A2Aプロジェクト寄贈（2025年6月）](https://www.linuxfoundation.org/)
- [python-a2a](https://pypi.org/project/python-a2a/)
- [Phi-3.5-mini-instruct (Microsoft)](https://huggingface.co/microsoft/Phi-3.5-mini-instruct)
- [bartowski/Phi-3.5-mini-instruct-GGUF](https://huggingface.co/bartowski/Phi-3.5-mini-instruct-GGUF)
- [llama-cpp-python](https://github.com/abetlen/llama-cpp-python)
