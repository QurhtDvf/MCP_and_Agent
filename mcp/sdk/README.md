# MCP Server サンプル (FastMCP) on Google Colab

Google Colab上でMCP (Model Context Protocol) サーバーを構築・テストできるJupyter Notebookのサンプルです。公式Python SDKに含まれる **FastMCP** を使い、最小限のコードでMCPサーバーを実装します。

---

## MCP (Model Context Protocol) とは

**MCP**はAnthropicが2024年11月に公開したオープンプロトコルで、AIモデルと外部ツール・データソースを**標準化された方法で接続する**ための仕様です。

従来、AIアプリケーションに外部ツールを組み込む場合、開発者はサービスごとに独自の連携コードを書く必要がありました。MCPはこの問題を解決するために、**USB-Cポートのような統一インターフェース**を提供します。一度MCPサーバーを実装すれば、MCPに対応したあらゆるクライアント（Claude Desktop、IDE、カスタムアプリなど）から同じサーバーを利用できます。

```
┌─────────────────────────────────────────────────────┐
│                   MCPクライアント                     │
│  (Claude Desktop / IDE / カスタムアプリ など)         │
└───────────────────────┬─────────────────────────────┘
                        │ MCP Protocol (JSON-RPC 2.0)
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
    MCPサーバーA   MCPサーバーB   MCPサーバーC
    (ファイル操作)  (データベース)  (外部API)
```

### MCPの主要コンセプト

| 概念 | 説明 | 例 |
|---|---|---|
| **Tools** | AIが呼び出せる関数 | 計算、API呼び出し、DB検索 |
| **Resources** | AIが読み取れるデータソース | ファイル、DB、メモ |
| **Prompts** | 再利用可能なプロンプトテンプレート | 要約、翻訳、コードレビュー |

### トランスポート方式

| 方式 | 用途 |
|---|---|
| **stdio** | ローカルプロセス間通信。Claude Desktopなどのデスクトップアプリ向け |
| **SSE** | HTTP経由のリアルタイム通信。リモートサーバーやWebアプリ向け |
| **Streamable HTTP** | 次世代のHTTPトランスポート。SSEの後継として策定中 |

---

## FastMCP とは

### 誕生の背景

公式MCP Python SDKの低レベルAPIは、ツール1つを登録するだけでも多くのボイラープレートが必要でした。

```python
# 低レベルAPI（公式SDKの旧来の書き方）
@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="add",
            description="2つの整数を足し算して返す",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"},
                },
                "required": ["a", "b"],
            },
        )
    ]

@app.call_tool()
async def call_tool(name, arguments):
    if name == "add":
        return [types.TextContent(type="text", text=str(arguments["a"] + arguments["b"]))]
```

**FastMCP**はこの問題を解決するために作られたフレームワークです。Pythonの型ヒントとdocstringから、スキーマ定義・バリデーション・ドキュメントを**自動生成**します。

```python
# FastMCP（同じツールを1/5のコードで書ける）
@mcp.tool()
def add(a: int, b: int) -> int:
    """2つの整数を足し算して返す"""
    return a + b
```

### FastMCPの歴史

FastMCPはもともと**Jeremiah Lowin氏**が「life's too short for boilerplate（ボイラープレートを書く時間は無駄）」というコンセプトで開発したOSSです。MCP登場からわずか1週間後にリリースされ、その設計思想が高く評価されました。

- **2024年**: FastMCP 1.0 が公式MCP Python SDKに取り込まれ、`mcp.server.fastmcp.FastMCP` としてインポート可能に
- **2025年4月**: FastMCP 2.0 をスタンドアロンパッケージとしてリリース（サーバープロキシ、OpenAPI統合、クライアント機能などを追加）

現在、**全言語のMCPサーバーの約70%**が何らかの形でFastMCPを利用しており、スタンドアロン版は1日あたり100万回以上ダウンロードされています。

### 2つのFastMCP

| | 公式SDK内蔵版 | スタンドアロン版 |
|---|---|---|
| **インポート** | `from mcp.server.fastmcp import FastMCP` | `from fastmcp import FastMCP` |
| **インストール** | `pip install mcp` | `pip install fastmcp` |
| **バージョン** | FastMCP 1.0ベース | FastMCP 2.0以降（機能が多い） |
| **用途** | 基本的なサーバー実装 | プロキシ・OpenAPI統合・高度な構成 |

このNotebookでは**公式SDK内蔵版**（`mcp.server.fastmcp`）を使用しています。

### FastMCPの主な機能

```python
from mcp.server.fastmcp import FastMCP
mcp = FastMCP("my-server")

# ツール：型ヒントからJSONスキーマを自動生成
@mcp.tool()
def search(query: str, limit: int = 10) -> list[str]:
    """指定したクエリで検索する"""
    ...

# リソース：URIテンプレートでデータを公開
@mcp.resource("users://{user_id}/profile")
def get_profile(user_id: str) -> str:
    """ユーザープロフィールを返す"""
    ...

# プロンプト：再利用可能なテンプレート
@mcp.prompt()
def summarize(text: str, language: str = "日本語") -> str:
    """テキストを要約するプロンプト"""
    return f"{language}で3行以内に要約してください:\n\n{text}"
```

---

## このリポジトリについて

### ファイル構成

```
.
├── README.md
└── mcp_fastmcp.ipynb   # メインのNotebook
```

### Notebookの内容

| Step | 内容 |
|---|---|
| 1 | パッケージインストール (`mcp[cli]`, `nest_asyncio`) |
| 2 | `server.py` の書き出し（FastMCPサーバー本体） |
| 3 | 書き出したファイルの存在確認・内容表示 |
| 4 | ツール関数を直接呼び出してテスト（asyncio不要） |
| 5 | `ClientSession` でインプロセスMCPプロトコル通信テスト |

### 実装されているツール・リソース・プロンプト

```python
# Tools
add(a: int, b: int) -> int           # 足し算
greet(name: str) -> str              # 挨拶
get_weather(city: str) -> str        # ダミー天気情報

# Resource
@mcp.resource("memo://sample")       # サンプルテキストリソース

# Prompt
summarize(text: str) -> str          # 要約プロンプトテンプレート
```

---

## 使い方

### Google Colabで実行する

1. `mcp_fastmcp.ipynb` をGoogle Colabで開く
2. セルを**上から順番に**実行する

> **注意事項**
> - セルは必ず上から順に実行してください。`server.py` の書き出しセルをスキップすると `ModuleNotFoundError` が発生します。
> - Step 5ではColabのJupyterカーネルの制約（`fileno()` 非対応）のため、stdioではなくインプロセス接続を使用しています。JSON-RPCのシリアライズ・デシリアライズは実際に行われます。

### ローカル環境で実行する

```bash
pip install "mcp[cli]" nest_asyncio jupyter
jupyter notebook mcp_fastmcp.ipynb
```

---

## MCPサーバーの拡張方法

### 新しいツールを追加する

`server.py` に関数を追加するだけです。

```python
@mcp.tool()
def translate(text: str, target_lang: str = "English") -> str:
    """テキストを指定言語に翻訳する"""
    # 実際にはDeepL APIなどを呼び出す
    return f"[{target_lang}] {text}"
```

### 外部APIと連携する

非同期関数にするだけで `httpx` などが使えます。

```python
import httpx

@mcp.tool()
async def fetch_url(url: str) -> str:
    """URLのコンテンツを取得する"""
    async with httpx.AsyncClient() as client:
        resp = await client.get(url)
        return resp.text
```

### Claude Desktopと接続する

`server.py` を単体で起動し、Claude Desktopの設定ファイル (`claude_desktop_config.json`) に登録します。

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

---

## 技術スタック

- [mcp](https://github.com/modelcontextprotocol/python-sdk) — 公式MCP Python SDK（FastMCP 1.0を内蔵）
- [fastmcp](https://github.com/jlowin/fastmcp) — FastMCP 2.0スタンドアロン版（より高機能）
- [nest_asyncio](https://github.com/erdewit/nest_asyncio) — Jupyter環境でのasyncio対応

## 参考リンク

- [MCP公式ドキュメント](https://modelcontextprotocol.io)
- [MCP Python SDK (GitHub)](https://github.com/modelcontextprotocol/python-sdk)
- [FastMCP 2.0ドキュメント](https://gofastmcp.com)
- [FastMCP 2.0リリースノート](https://www.jlowin.dev/blog/fastmcp-2)
- [Anthropic MCP発表ブログ](https://www.anthropic.com/news/model-context-protocol)

## ライセンス

MIT
