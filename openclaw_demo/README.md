# 🦞 OpenClaw on Google Colab（ローカルLLM版）

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/your-username/openclaw-colab/blob/main/openclaw_colab_demo.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-24%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Ollama](https://img.shields.io/badge/Ollama-0.19%2B-black?logo=ollama)](https://ollama.com/)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.3.x-ff6b35?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyeiIvPjwvc3ZnPg==)](https://openclaw.ai/)
[![GPU](https://img.shields.io/badge/GPU-T4%20(Colab%20Free)-76b900?logo=nvidia&logoColor=white)](https://colab.research.google.com/)
[![Stars](https://img.shields.io/github/stars/openclaw/openclaw?style=social)](https://github.com/openclaw/openclaw)

**APIキー不要・ngrok不要**でOpenClawをGoogle Colab（無料版）上でローカルLLM（Ollama）を使って動かすためのノートブックです。

---

## 📖 OpenClawとは

OpenClawはオーストリアの開発者Peter Steinbergerが作ったオープンソースのAIエージェントフレームワークです。LLMをメッセージングアプリ（WhatsApp、Telegram、Discordなど）に接続し、コンピュータ上のタスクを自律実行できる「個人用AIアシスタント」として設計されています。

### 🕰️ 開発の歴史

OpenClawは2025年11月、Steinbergerが個人的な実験プロジェクトとして「ClawdBot」という名前で公開したのが始まりです。その後、急成長とともに名称変更を繰り返しました：

| 時期 | 名称 | 出来事 |
|------|------|--------|
| 2025年11月 | **ClawdBot** | WhatsAppリレーボットとして初公開 |
| 2026年1月27日 | **MoltBot** | Anthropicの商標問題で改名 |
| 2026年1月30日 | **OpenClaw** | 現在の名称に変更・オープンソース化を明確化 |
| 2026年2月14日 | — | SteinbergerがOpenAIに参加、プロジェクトを独立財団に移管 |
| 2026年2月 | — | GitHubスターが247,000を突破、最速で成長したOSSプロジェクトの一つに |
| 2026年3月5日 | — | NvidiaのJensen Huang CEOが「おそらく史上最も重要なソフトウェアリリース」と言及 |
| 2026年3月16日 | — | NvidiaがエンタープライズセキュリティアドオンNemoClawをリリース |

### ✨ 主な機能

OpenClawはローカルファーストのゲートウェイとして、セッション・チャンネル・ツール・イベントを一元管理します。

**マルチチャンネル対応**
WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage（BlueBubbles）、IRC、Microsoft Teams、Matrix、LINE、Twitch、WeChatなど20以上のメッセージングプラットフォームに対応しています。

**マルチエージェント**
インバウンドチャンネル・アカウント・ピアを独立したエージェント（ワークスペース＋エージェント別セッション）にルーティングするマルチエージェント機能を備えています。

**スキルシステム**
スキルはSKILL.mdファイルを含むディレクトリとして保存され、バンドル・グローバル・ワークスペース単位でインストールでき、ワークスペーススキルが最優先されます。

**ローカルLLM対応**
Claude、GPT、GeminiなどのクラウドAPIに加え、Ollamaを経由したローカルモデルにも対応しており、APIコストをゼロにすることも可能です。

**音声・ビジュアル**
macOS/iOSでのウェイクワードとAndroidでの継続音声（ElevenLabs＋システムTTSフォールバック）、およびエージェント主導のビジュアルワークスペース（A2UI）をサポートします。

### ⚠️ セキュリティ上の注意

OpenClawはシェルコマンドの実行やファイルシステムへのアクセス、ネットワークリクエストをエージェントが行えるため、設定ミスや不正なプロンプトインジェクションによるリスクがあります。特にColabのような共有環境ではAPIキーの管理に注意してください。

---

## 🚀 このノートブックの特徴

- ✅ **ngrok不要** — Colab組み込みの`serve_kernel_port_as_iframe`を使用
- ✅ **APIキー不要** — Ollamaによるローカル推論
- ✅ **無料** — Google Colab無料枠（T4 GPU）で動作
- ✅ **日本語対応** — Qwen2.5モデルによる日本語処理

---

## 📋 動作環境

| 項目 | 要件 |
|------|------|
| Google Colab | 無料版（T4 GPU推奨） |
| Node.js | 24以上（自動インストール） |
| Ollama | 0.19以上（自動インストール） |
| モデル | qwen2.5:7b-instruct-q4_K_M |
| VRAM | 16GB（T4 GPU） |

---

## 📝 使い方

1. バッジの **Open in Colab** をクリック
2. 「ランタイム → ランタイムのタイプを変更 → **T4 GPU**」に設定
3. セルを上から順番に実行

---

## 🔧 トラブルシューティング

| 症状 | 対処法 |
|------|--------|
| `Unable to detect NVIDIA/AMD GPU` | T4 GPUを選択してセッションをリセット |
| `could not connect to a running Ollama instance` | Step 3を再実行（pkillしてから再起動） |
| `No API key found for provider "anthropic"` | Step 4の設定セルを再実行 |
| `unknown option '-agent main'` | `--agent` と `main` を別引数にする |
| iframeが空白 | Step 5（ゲートウェイ起動）を再実行し10秒待つ |
| セッション切れ後 | Step 3〜5のみ再実行（1・2は不要） |

---

## 📚 参考リンク

- [OpenClaw 公式サイト](https://openclaw.ai/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw ドキュメント](https://docs.openclaw.ai/)
- [Ollama 公式サイト](https://ollama.com/)
- [ClawHub スキルマーケット](https://clawhub.ai/)

---

## 📄 ライセンス

このノートブックはMITライセンスで公開しています。OpenClaw本体のライセンスは[openclaw/openclaw](https://github.com/openclaw/openclaw)を参照してください。
