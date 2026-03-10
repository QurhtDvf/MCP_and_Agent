# MCP Server サンプル on Google Colab

Google Colab上でMCP (Model Context Protocol) サーバーを構築・テストできるJupyter Notebookのサンプルです。

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
| **Sampling** | サーバーがAIに推論を依頼する仕組み | エージェント的な処理 |

### トランスポート方式

MCPは通信方式として主に以下の2種類をサポートしています。

- **stdio**: 標準入出力を使った通信。ローカルプロセス間の連携に適しており、Claude Desktopなどのデスクトップアプリとの連携で主に使われます。
- **SSE (Server-Sent Events)**: HTTP経由のリアルタイム通信。リモートサーバーとの連携や、Webアプリケーションとの統合に適しています。

---

## このリポジトリについて

### ファイル構成

```
.
├── README.md
└── mcp_server_colab.ipynb   # メインのNotebook
```

### Notebookの内容

| セル | 内容 |
|---|---|
| 1 | パッケージインストール (`mcp`, `httpx`, `nest_asyncio`) |
| 2 | `server.py` の書き出し（MCPサーバー本体） |
| 3 | 書き出したファイルの確認 |
| 4 | `server.py` の内容表示 |
| 5 | `server.py` のインポート |
| 6〜 | 各ツール・リソース・プロンプトのテスト |
| 最後 | サブプロセスとしての起動デモ |

### 実装されているツール

```python
# 足し算
add(a: number, b: number) -> "42 + 58 = 100"

# 挨拶
greet(name: string) -> "こんにちは、Claudeさん！MCPサーバーへようこそ"

# ダミー天気情報
get_weather(city: string) -> "Tokyoの天気: 晴れ, 気温: 23度C (デモデータ)"
```

---

## 使い方

### Google Colabで実行する

1. `mcp_server_colab.ipynb` をGoogle Colabで開く
2. セルを**上から順番に**実行する

> **注意**: セルは必ず上から順に実行してください。`server.py` の書き出しセルをスキップすると `ModuleNotFoundError` が発生します。

### ローカル環境で実行する

```bash
# 依存パッケージのインストール
pip install mcp httpx nest_asyncio

# Notebookを起動
jupyter notebook mcp_server_colab.ipynb
```

---

## MCPサーバーの拡張方法

### 新しいツールを追加する

`server.py` の `list_tools()` と `call_tool()` に追加します。

```python
# list_tools() に追加
types.Tool(
    name='translate',
    description='テキストを英語に翻訳します',
    inputSchema={
        'type': 'object',
        'properties': {
            'text': {'type': 'string', 'description': '翻訳するテキスト'},
        },
        'required': ['text'],
    },
),

# call_tool() に追加
elif name == 'translate':
    # 実際にはDeepL APIなどを呼び出す
    return [types.TextContent(type='text', text=f"(翻訳結果) {arguments['text']}")]
```

### 外部APIと連携する

`httpx` を使って非同期でAPIを呼び出せます。

```python
elif name == 'fetch_stock':
    async with httpx.AsyncClient() as client:
        resp = await client.get(f"https://api.example.com/stock/{arguments['symbol']}")
        data = resp.json()
    return [types.TextContent(type='text', text=str(data['price']))]
```

### Claude Desktopと接続する

`server.py` を単体で起動し、Claude Desktopの設定ファイル (`claude_desktop_config.json`) に登録します。

```json
{
  "mcpServers": {
    "colab-demo": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

---

## 技術スタック

- [mcp](https://github.com/modelcontextprotocol/python-sdk) — MCP Python SDK
- [httpx](https://www.python-httpx.org/) — 非同期HTTPクライアント
- [nest_asyncio](https://github.com/erdewit/nest_asyncio) — Jupyter環境でのasyncio対応

## 参考リンク

- [MCP公式ドキュメント](https://modelcontextprotocol.io)
- [MCP Python SDK (GitHub)](https://github.com/modelcontextprotocol/python-sdk)
- [Anthropic MCP発表ブログ](https://www.anthropic.com/news/model-context-protocol)
- [MCP仕様書](https://spec.modelcontextprotocol.io)

## ライセンス

MIT
