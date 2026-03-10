# OpenCode on Google Colab with Ollama (Qwen3)

Google Colab 上で **OpenCode** を無料で動かすためのセットアップノートブックです。ローカルLLMエンジンの **Ollama** と **Qwen3:14b** モデルを組み合わせ、APIキー不要でAIコーディングエージェントを体験できます。

---

## OpenCode とは？

[OpenCode](https://opencode.ai) は、ターミナル上で動作するオープンソースの **AIコーディングエージェント** です。

- **ファイルの読み書き・編集**をAIが自律的に行う
- Claude・GPT・Geminiなど多数のLLMに対応（OpenAI互換APIも可）
- `opencode run "指示"` のようにCLIから直接タスクを指示できる
- `SKILL.md` を使ってコーディングスタイルやルールをAIに学習させられる
- **完全オープンソース**（ライセンス: Apache 2.0）

> Claude Code や Cursor に近い概念ですが、CLIベースでより軽量・柔軟に動かせるのが特徴です。

---

## このノートブックでやること

| ステップ | 内容 |
|---|---|
| 1 | Ollama のインストールと起動 |
| 2 | GPU (Tesla T4) の認識確認 |
| 3 | `qwen3:14b` モデルのダウンロード |
| 4 | カスタム Modelfile で 16k コンテキストモデルを作成 |
| 5 | OpenCode のインストール（v1.2.3） |
| 6 | Ollama プロバイダーとして OpenCode を設定 |
| 7 | `opencode run` でコード生成タスクを実行 |
| 8 | `SKILL.md` によるコーディングルールの適用 |

---

## 使用技術・ツール

- **[Ollama](https://ollama.com)** — ローカルでLLMを動かすためのランタイム
- **[Qwen3:14b](https://ollama.com/library/qwen3)** — Alibaba製のオープンソースLLM（14Bパラメータ）
- **[OpenCode](https://opencode.ai)** — ターミナルで動くAIコーディングエージェント
- **Google Colab** — 無料GPU（Tesla T4）環境
- **colab-xterm** — Colab上でターミナルを使うための拡張

---

## クイックスタート

1. [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/) でノートブックを開く
2. ランタイムを **GPU（T4）** に変更する（`ランタイム` → `ランタイムのタイプを変更`）
3. セルを上から順に実行する

---

## 設定ファイル

OpenCodeはOllama経由でQwen3を使用するよう設定されます。設定内容（`~/.config/opencode/opencode.json`）：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/qwen3:14b-16k",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3:14b-16k": {
          "name": "Qwen3 14B (16k context)",
          "tools": true
        }
      }
    }
  }
}
```

---

## SKILL.md によるカスタマイズ

`SKILL.md` をプロジェクトディレクトリに置くことで、OpenCodeのコーディングルールをカスタマイズできます。

このノートブックでは以下のルールを適用しています：

```markdown
# Python コーディングスキル

## ルール
- 全ての関数には必ず日本語のdocstringをつける
- 変数名は英語、コメントは日本語で書く
- 関数の最後に使用例をprint文で示す
```

`-f` オプションでファイルを添付して使用します：

```bash
opencode run "SKILL.mdのルールに従って素数判定関数を作成してください" -f /content/SKILL.md
```

---

## 注意事項

- Google Colab の無料プランではセッション終了時にデータが消えます。生成ファイルは都度保存してください。
- Qwen3:14bのダウンロードには数分かかります（約9.3GB）。
- Ollamaは起動のたびに `subprocess.Popen` で再起動が必要です。

---

## 参考リンク

- [OpenCode 公式ドキュメント](https://opencode.ai/docs)
- [Ollama 公式サイト](https://ollama.com)
- [Qwen3 モデル一覧](https://ollama.com/library/qwen3)
