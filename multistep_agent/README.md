# 🤖 AI Agent with llama-cpp-python + Phi-3.5

Google Colab上でオープンソースLLMを使ったマルチステップAIエージェントを動かすノートブックです。
登録不要・ビルド不要で、すぐに試せます。

## マルチステップAIエージェントとは

通常のLLMは「質問を受けて回答を返す」だけですが、**AIエージェント**はタスクを自律的に分解・実行できます。

```
通常のLLM:
  ユーザー → 質問 → LLM → 回答（1回で終わり）

マルチステップAIエージェント:
  ユーザー → タスク
                ↓
          1つのエージェントが以下を順番に処理
                ↓
          [Step 1: プランニング]  タスクを複数ステップに分解
                ↓
          [Step 2: ツール実行]   各ステップをツールで実行
            - 検索する
            - 計算する
            - データを分析する ...
                ↓
          [Step 3: サマリー]     結果をまとめて最終回答を生成
                ↓
             ユーザー ← 回答
```

### 通常のLLMとの違い

| | 通常のLLM | マルチステップエージェント |
|---|---|---|
| 動作 | 1回の推論で回答 | 1つのエージェントが複数ステップを順番に実行 |
| ツール利用 | なし | 計算・検索・分析などを呼び出せる |
| 複雑なタスク | 苦手 | ステップに分解して対応できる |
| 精度 | ハルシネーションが起きやすい | ツールで事実確認できる |

### 具体例

```
タスク:「テストの点数を分析して、平均点を教えて」

Step 1: data_analyzer で統計を計算  ← ツール実行
Step 2: 結果をまとめて日本語で回答  ← LLM生成
```

LLMは「プランの立案」と「最終回答の生成」にのみ使用します。
計算や統計分析はPython関数として実装されたツールが直接実行するため、**LLMのハルシネーションに左右されない正確な結果**が得られます。

| フェーズ | 処理 | LLM呼び出し |
|---------|------|------------|
| Phase 1: プランナー | タスクをステップに分解 | ✅ あり |
| Phase 2: エグゼキューター | 各ツール（計算・検索・分析）を実行 | ❌ なし (※text_summarizerを除く) |
| Phase 3: サマライザー | 結果をまとめて最終回答を生成 | ✅ あり |

---

## 構成

```
Pythonコード
    ↓ 直接呼び出し
Llama クラス (llama-cpp-python)
    ↓
Phi-3.5-mini-instruct Q4_K_M (GPU)
```

## 特徴

- **登録不要** — Hugging Faceアカウント不要でモデルをダウンロード
- **ビルド不要** — `pip install` だけで動作
- **マルチステップ実行** — タスクを自動分解して順番に処理
- **OpenAI非依存** — 完全ローカルで動作
- **GPU加速** — CUDA対応 (Google Colab T4推奨)

## 動作環境

| 項目 | 内容 |
|------|------|
| 実行環境 | Google Colab |
| GPU | T4 (15GB VRAM) 推奨 |
| モデル | Phi-3.5-mini-instruct (Microsoft) |
| モデルサイズ | 約2.2GB (Q4_K_M量子化) |
| ライセンス | MIT |

## セットアップ

### 1. Colabでノートブックを開く

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

### 2. ランタイムをGPUに変更

`ランタイム` → `ランタイムのタイプを変更` → `T4 GPU` を選択

### 3. セルを順番に実行

| Step | 内容 | 時間 |
|------|------|------|
| Step 1 | llama-cpp-pythonインストール → **ランタイム再起動** | 約1分 |
| Step 2 | GPU確認 | 即時 |
| Step 3 | Phi-3.5モデルのダウンロード | 約3〜5分 |
| Step 4 | モデルのロード | 約1〜2分 |
| Step 5〜7 | 関数・エージェント定義 | 即時 |
| Step 8 | エージェント実行 | タスクによる |
| Step 9 | インタラクティブUI | — |

> ⚠️ Step 1のインストール後は必ずランタイムを再起動してからStep 2以降を実行してください。

## 利用可能なツール

| ツール名 | 機能 |
|---------|------|
| `calculator` | 数式の計算 |
| `search_knowledge` | ナレッジベース検索 |
| `data_analyzer` | 数値データの統計分析 |
| `text_summarizer` | テキスト要約 (LLM使用) |
| `web_search` | Web検索 (モック) |

## 実行例

```
📋 タスク: 次のテストスコアを分析して平均点と最高点を教えて: 72, 85, 91, 68, 78, 95, 83, 77, 88, 64

📌 Phase 1: タスク分解中...
  Step 1: データ分析 [🔧 data_analyzer]
  Step 2: 結果まとめ

⚙️  Phase 2: ステップ実行中...
  ▶ Step 1: データ分析
  ✓ 件数:10  最小:64.0  最大:95.0
    平均:80.10  中央値:80.5  標準偏差:9.60

✅ 最終回答
テストスコアの平均点は80.1点、最高点は95点です。
```

## カスタマイズ

### 別モデルへの切り替え（すべて登録不要）

```bash
# Qwen2.5-3B（日本語が強い）
wget https://huggingface.co/bartowski/Qwen2.5-3B-Instruct-GGUF/resolve/main/Qwen2.5-3B-Instruct-Q4_K_M.gguf

# Gemma-3-1B（超軽量）
wget https://huggingface.co/bartowski/gemma-3-1b-it-GGUF/resolve/main/gemma-3-1b-it-Q4_K_M.gguf
```

```python
llm = Llama(model_path='/content/Qwen2.5-3B-Instruct-Q4_K_M.gguf',
            chat_format='chatml', n_gpu_layers=35, n_ctx=4096, verbose=False)
```

### 実Web検索への差し替え

```python
!pip install duckduckgo-search
from duckduckgo_search import DDGS

def web_search(query: str) -> str:
    with DDGS() as ddgs:
        results = list(ddgs.text(query, max_results=3))
        return '\n'.join(r['body'] for r in results)

TOOLS['web_search']['fn'] = web_search
```

### 独自ツールの追加

```python
def my_tool(input: str) -> str:
    # 処理
    return result

TOOLS['my_tool'] = {'fn': my_tool, 'desc': 'ツールの説明'}
```

## ライセンス

MIT License

## 参考

- [llama-cpp-python](https://github.com/abetlen/llama-cpp-python)
- [Phi-3.5-mini-instruct (Microsoft)](https://huggingface.co/microsoft/Phi-3.5-mini-instruct)
- [bartowski/Phi-3.5-mini-instruct-GGUF](https://huggingface.co/bartowski/Phi-3.5-mini-instruct-GGUF)
