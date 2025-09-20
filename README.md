# MCPサーバー構築ガイド for Claude Code
最低限構築するMCPサーバーリスト

## MCPサーバー構築の考え方
| **Scope** | **範囲** | **設定ファイル** |
|-------|------|--------------|
| local | 今のプロジェクトで使える | ~/.claude.json |
| project | 今のプロジェクトで使える | .mcp.json(Ropoのルート直下) |
| user | どこでも使える | ~/.claude.json |

スコープは原則project、localとuserはあまり推奨されない。

## 対象MCPサーバー
1. context7 - ファイルコンテキスト管理
2. github - GitHub連携
3. notion - Notion連携
4. linear - Linear連携
5. serena - AI アシスタント

https://docs.claude.com/ja/docs/claude-code/mcp


## 基本設定方法

### 前提条件
- npmがインストール済み
- Claude Codeがグローバルにインストール済み: `npm install -g @anthropic-ai/claude-code`
- github,notion,linearのアカウント作成完了・ログインができる状態

### MCP設定スコープ
- `local`: プロジェクト固有の設定
- `project`: チーム共有可能な設定（推奨）
- `user`: ユニバーサル設定

### 基本コマンド
```bash
# MCPサーバーの追加
claude mcp add <server-name> -s project -- <installation-command>

# インストール済みサーバーの確認
claude mcp list

# サーバーの削除
claude mcp remove <server-name>
```

### 設定ファイル
- 手動での`.mcp.json`編集も可能
- 設定ファイルはMCPホスト間で互換性あり

## MCPサーバーインストール手順

### 1. context7 - ファイルコンテキスト管理
```bash
claude mcp add -s user context7 -- npx --yes @upstash/context7-mcp
```

**機能**: 最新のライブラリドキュメントとコード例を提供
**使用方法**: プロンプトに "use context7" を含める

### 2. github - GitHub連携
```bash
claude mcp add -s user github -- npx -y @modelcontextprotocol/server-github@latest 
```

**機能**: GitHub API連携、リポジトリ操作、PR管理
**使用方法**: GitHub Personal Access Tokenが必要（環境変数で設定）

設定確認
```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "env": {}
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github@latest"],
      "env": {}
    },
    "notion": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {}
    }
  }
}
```

### 3. notion - Notion連携
```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

**機能**: Notionデータベース操作、ページ作成・更新
**使用方法**: Notion API Tokenが必要（環境変数NOTION_TOKENで設定）

### 4. linear - Linear連携
```bash
claude mcp add --transport sse linear -s project https://mcp.linear.app/sse
```

**機能**: Linearプロジェクト管理、イシュー作成・管理
**使用方法**: `/mcp`コマンドでOAuth認証フローを実行

### 5. serena - AIアシスタント
```bash
claude mcp add serena-mcp-server "uvx --from git+https://github.com/oraios/serena serena-mcp-server --context ide-assistant --project $(pwd)"
```
github: https://github.com/oraios/serena
- SerenaのWebダッシュボードページがブラウザで開き、ログが閲覧できるが、このページを毎回見たくない場合は`--enable-web-dashboard false`を設定（`.serena/serena_config.yml`）

**機能**: セマンティックコード解析、シンボルレベル編集、多言語サポート
**使用方法**: IDE統合型AIアシスタント、Language Server Protocol連携

設定確認（更新後）
```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "env": {}
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github@latest"],
      "env": {}
    },
    "notion": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {}
    },
    "linear": {
      "type": "sse",
      "url": "https://mcp.linear.app/sse"
    },
    "serena": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/oraios/serena",
        "serena",
        "start-mcp-server",
        "--context",
        "ide-assistant",
        "--project",
        "/Users/tomohiro/dev/mcp-server-management"
      ],
      "env": {}
    }
  }
}
```


## Setup
- ローカルでmcpの接続チェック
```bash
claude mcp list
```

- Claude Codeへ接続
```bash
claude mcp connect
```

認証系のタスクを実施して接続確認ができればok

- 不要なmcp serverの削除
```bash
claude mcp remove <server-name>
```

## .CLAUDE/settings.local.jsonの役割

- MCPサーバーの有効/無効制御
  .mcp.jsonで定義されたMCPサーバーのうち、どれを実際に有効化するかを制御
  - プロジェクト固有設定: 特定のプロジェクトでのみ使用したいMCPサーバーを選択的に有効化
  - ローカル環境設定: .mcp.jsonをチーム共有しつつ、個人の開発環境に合わせてサーバーを調整

  - 現在の設定
  ```json
  {
    "enabledMcpjsonServers": [
      "context7",
      "github",
      "notion",
      "linear",
      "serena"
    ]
  }
  ```

### 使用例
  - チーム開発で全MCPサーバーが.mcp.jsonに定義されているが、個人では一部のみ使用したい場合
  - 特定のプロジェクトで不要なMCPサーバーを無効化したい場合
  - パフォーマンス調整やトラブルシューティングで一時的にサーバーを無効化したい場合

### ファイル管理

  - .gitignoreに追加推奨（個人環境固有の設定のため）
  - .mcp.jsonは共有、.CLAUDE/settings.local.jsonは個人管理
