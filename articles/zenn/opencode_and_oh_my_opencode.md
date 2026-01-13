---
title: "突撃！隣のOpenCode！（概要から実践編まで）"
emoji: "🤖"
type: "tech"
topics:
  - "AI"
  - "opencode"
  - "ohmyopencode"
  - "エージェント"
  - "開発ツール"
published: false
---

## 初めに

こんにちは、[かめぽん](https://twitter.com/kamepon_fe)です。

本記事では、オープンソースのAIコーディングエージェント「**OpenCode**」と、それを拡張するプラグイン「**oh-my-opencode**」について、導入から実践的な設定まで解説します。

---

## 概要編（opencode編）

### opencodeとは

[OpenCode](https://opencode.ai/)は、オープンソースのAIコーディングエージェントです。ターミナルベースのTUI（Terminal User Interface）、デスクトップアプリ、IDE拡張として利用できます。ざっくり言うとオープンソース版Claude Codeとも言えます。

https://opencode.ai/

#### 主な特徴

- **オープンソース**: 完全にオープンソースで開発されており、透明性が高い
- **マルチプロバイダー対応**: 75以上のLLMプロバイダーに対応（OpenAI、Anthropic、Google、xAI等）
- **エージェントシステム**: Build、Plan、General、Exploreなどの専門エージェントを搭載
- **AGENTS.md対応**: `AGENTS.md`（Claude Codeにおける `CLAUDE.md`のようなもの）がデフォルトで対応
- **GitHub連携**: Issue/PRで`/opencode`コマンドを使って直接エージェントを呼び出せる
- **MCP対応**: Model Context Protocol（MCP）に対応し、外部ツールとの連携が可能

### opencodeの導入方法

#### インストール

最も簡単な方法はインストールスクリプトを使う方法です。

```bash
curl -fsSL https://opencode.ai/install | bash
```

パッケージマネージャーを使う場合は以下の通りです。

```bash
# npm
npm install -g opencode-ai
# Homebrew (macOS/Linux)
brew install anomalyco/tap/opencode
```

#### 初期設定

インストール後、プロジェクトディレクトリで以下を実行します。

```bash
opencode
```

初回起動時にLLMプロバイダーの設定を求められます。`/connect`コマンドで各プロバイダーのAPIキーを設定できます。

```
/connect
```

#### 推奨モデル

[公式ドキュメント](https://opencode.ai/docs/models/#recommended-models)では、コード生成とツール呼び出しの両方に優れたモデルとして以下が推奨されています（2026年1月時点）。

- GPT 5.2
- GPT 5.1 Codex
- Claude Opus 4.5
- Claude Sonnet 4.5
- Minimax M2.1
- Gemini 3 Pro

### opencodeの基本的な使い方

#### TUI（Terminal User Interface）の起動

```bash
# プロジェクトディレクトリで起動
opencode

# 前回のセッションを継続
opencode -c

# 特定のセッションを継続
opencode -s <session_id>
```

#### 基本的なコマンド

TUI内で使用できる主なコマンドです。`/`に続けてコマンド名を入力します。

| コマンド | キーバインド | 説明 |
|----------|-------------|------|
| `/connect` | - | プロバイダーのAPIキーを設定 |
| `/models` | `ctrl+x m` | 使用可能なモデル一覧を表示 |
| `/init` | `ctrl+x i` | `AGENTS.md`ファイルを作成・更新（プロジェクトルールの設定） |
| `/new` | `ctrl+x n` | 新しいセッションを開始 |
| `/session` | `ctrl+x l` | セッション一覧を表示・切り替え |
| `/compact` | `ctrl+x c` | 会話を要約してコンテキストを圧縮 |
| `/undo` | `ctrl+x u` | 直前のメッセージを取り消し（ファイル変更も元に戻る） |
| `/redo` | `ctrl+x r` | undoを取り消し |
| `/share` | `ctrl+x s` | セッションを共有 |
| `/help` | `ctrl+x h` | ヘルプを表示 |

#### ファイル参照とシェルコマンド

TUI内ではファイル参照やシェルコマンドの実行も可能です。

```bash
# @でファイルをファジー検索して参照
@src/components/Button.tsx のコードをレビューして

# !でシェルコマンドを実行（結果が会話に追加される）
!npm test
```

#### CLIでの直接実行

TUIを起動せずに、コマンドラインから直接実行することもできます。

```bash
# 直接プロンプトを実行
opencode run "このプロジェクトの構造を説明して"

# 特定のモデルを使用
opencode run -m openai/gpt-5.2 "バグを修正して"
```

#### エージェントの切り替え

OpenCodeには複数のエージェントが用意されています。TUI内で**Tab**キーを押すことでエージェントを切り替えられます。

| エージェント | 種類 | 説明 |
|-------------|------|------|
| **Build** | プライマリ | 全ツールへのアクセス権を持つデフォルトエージェント |
| **Plan** | プライマリ | コードの分析・レビュー専用（変更を加えない） |
| **General** | サブ | 汎用的なタスク用 |
| **Explore** | サブ | コードベースの探索・検索用 |

サブエージェントは`@`メンションで呼び出すこともできます。

```
@explore このプロジェクトで認証がどのように実装されているか調べて
```

### opencode.jsonの設定について

プロジェクトルートに`opencode.json`を配置することで、プロジェクト固有の設定ができます。

#### 設定ファイルの場所

| スコープ | パス |
|---------|------|
| グローバル | `~/.config/opencode/opencode.json` |
| プロジェクト | `<project>/.opencode/opencode.json` または `<project>/opencode.json` |

プロジェクト設定はグローバル設定を上書きします。

#### 実践的な設定例

以下は実際に使用している設定例です。

```json
{
  "$schema": "https://opencode.ai/config.json",
  "theme": "opencode",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "opencode/grok-code-fast-1",
  "instructions": ["AGENTS.md"],
  "provider": {
    "anthropic": {
      "name": "Anthropic"
    },
    "grok": {
      "name": "Grok"
    }
  },
  "tui": {
    "scroll_speed": 1,
    "scroll_acceleration": {
      "enabled": true
    },
    "diff_style": "auto"
  },
  "permission": {
    "bash": {
      "*": "allow",
      "rm -rf *": "deny",
      "git push --force*": "deny",
      "sudo *": "deny",
      "dd *": "deny",
      "mkfs*": "deny",
      "format*": "deny"
    }
  },
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

#### 各設定項目の解説

##### 基本設定

| 項目 | 説明 | 例 |
|------|------|-----|
| `$schema` | JSON Schemaを指定 | `"https://opencode.ai/config.json"` |
| `theme` | TUIのテーマ | `"opencode"`, `"catppuccin"` 等 |
| `model` | デフォルトで使用するモデル | `"anthropic/claude-sonnet-4-5"` |
| `small_model` | 軽量タスク用のモデル（要約等） | `"opencode/grok-code-fast-1"` |
| `instructions` | エージェントに読み込ませるルールファイル | `["AGENTS.md"]` |

##### プロバイダー設定（provider）

使用するLLMプロバイダーを設定します。APIキーは`/connect`コマンドまたは環境変数で設定します。

```json
{
  "provider": {
    "anthropic": {
      "name": "Anthropic"
    },
    "openai": {
      "name": "OpenAI"
    },
    "grok": {
      "name": "Grok"
    }
  }
}
```

##### TUI設定（tui）

ターミナルUIの挙動をカスタマイズします。

```json
{
  "tui": {
    "scroll_speed": 1,
    "scroll_acceleration": {
      "enabled": true
    },
    "diff_style": "auto"
  }
}
```

| 項目 | 説明 |
|------|------|
| `scroll_speed` | スクロール速度（1〜、デフォルトはUnix: 1, Windows: 3） |
| `scroll_acceleration.enabled` | macOS風のスクロール加速を有効化（有効時は`scroll_speed`は無視） |
| `diff_style` | 差分表示スタイル（`"auto"`, `"unified"`, `"split"`） |

##### パーミッション設定（permission）

危険なコマンドの実行を制御します。セキュリティ上重要な設定です。

```json
{
  "permission": {
    "bash": {
      "*": "allow",
      "rm -rf *": "deny",
      "git push --force*": "deny",
      "sudo *": "deny",
      "dd *": "deny",
      "mkfs*": "deny",
      "format*": "deny"
    }
  }
}
```

| 値 | 説明 |
|----|------|
| `"allow"` | 確認なしで実行を許可 |
| `"deny"` | 実行を拒否 |
| `"ask"` | 実行前に確認を求める |

ワイルドカード（`*`）を使ってパターンマッチングができます。上記の例では、基本的にすべてのコマンドを許可しつつ、危険なコマンド（`rm -rf`、`git push --force`、`sudo`等）は拒否しています。

##### MCP設定（mcp）

Model Context Protocol（MCP）サーバーとの連携を設定します。MCPを使うと外部ツールやAPIとシームレスに連携できます。

```json
{
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp"
    },
    "serena": {
      "type": "local",
      "command": [
        "uvx",
        "--from",
        "git+https://github.com/oraios/serena",
        "serena-mcp-server"
      ]
    },
    "chrome-devtools": {
      "type": "local",
      "command": ["npx", "-y", "chrome-devtools-mcp"],
      "environment": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    }
  }
}
```

| タイプ | 説明 | 設定項目 |
|--------|------|----------|
| `remote` | リモートのMCPサーバーに接続 | `url`: エンドポイントURL |
| `local` | ローカルでMCPサーバーを起動 | `command`: 起動コマンド、`environment`: 環境変数 |

よく使われるMCPサーバー：
- **context7**: ライブラリのドキュメント検索
- **serena**: コード解析・リファクタリング支援
- **chrome-devtools**: ブラウザのDevToolsと連携

##### プラグイン設定（plugin）

OpenCodeはプラグインシステムを持っており、oh-my-opencodeなどの拡張機能を追加できます。

```json
{
  "plugin": [
    "oh-my-opencode"
  ]
}
```

---

## 実践編（oh-my-opencodeと接続編）

### oh-my-opencodeとは

[oh-my-opencode](https://ohmyopencode.com/)は、OpenCodeを拡張する**エージェントオーケストレーションプラグイン**です。「The Best Agent Harness（最高のエージェントハーネス）」を謳っており、複数の専門エージェントを連携させて複雑なタスクを効率的に処理します。

https://ohmyopencode.com/

https://github.com/code-yeongyu/oh-my-opencode

#### 主な特徴

- **Sisyphusオーケストレーター**: 複数の専門エージェントを自動的に連携させる司令塔
- **バッテリー内蔵**: エージェント、フック、MCP、LSPサポートが事前設定済み
- **本番運用実績**: Google、Microsoft、Indentなどで実際に使用されている
- **高度なカスタマイズ**: エージェント、フック、MCP、LSPサーバーを自由に設定可能

#### 名前の由来

「Oh My Zsh」のように、設定済みの便利な機能が最初から使える状態で提供されることから名付けられています。エージェントハーネスの選択に悩む必要がなく、インストールすればすぐに強力なエージェントシステムが使えます。

### oh-my-opencodeの導入方法

#### インストール

```bash
# npm
npm install -g oh-my-opencode

# bun
bun install -g oh-my-opencode
```

#### 初期設定

OpenCodeの設定ファイル（`~/.config/opencode/opencode.json`）にプラグインを追加します。

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "oh-my-opencode"
  ]
}
```

### Sisyphusオーケストレーターと専門エージェントシステム

oh-my-opencodeの中核は**Sisyphus**というオーケストレーターエージェントです。ギリシャ神話のシーシュポスから名前を取っており、「毎日岩を転がす」ように、開発者と共に日々のタスクをこなすパートナーとして設計されています。

#### 専門エージェント一覧

Sisyphusが連携する専門エージェントは以下の通りです。

| エージェント | 役割 | 使いどころ |
|-------------|------|-----------|
| **explore** | コードベース内の探索・検索 | 「この機能どこで実装されてる？」 |
| **librarian** | 外部ドキュメント・OSS検索 | 「ReactのuseEffectの使い方調べて」 |
| **oracle** | アーキテクチャ設計・トラブルシューティング | 複雑な設計判断、2回以上失敗した問題 |
| **frontend-ui-ux-engineer** | フロントエンドのUI/UX実装 | 視覚的な変更、スタイリング |
| **document-writer** | ドキュメント作成 | README、APIドキュメントの作成 |
| **multimodal-looker** | メディアファイルの分析 | PDF、画像、図の解析 |

### oh-my-opencode.jsonの設定について

oh-my-opencodeの設定ファイルは以下の場所に配置します。

| スコープ | パス |
|---------|------|
| グローバル | `~/.config/opencode/oh-my-opencode.json` |
| プロジェクト | `<project>/.opencode/oh-my-opencode.json` |

プロジェクト設定はグローバル設定を上書きします。

#### 基本的な設定例

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "Sisyphus": {
      "model": "anthropic/claude-opus-4-5-20251101"
    },
    "oracle": {
      "model": "openai/gpt-5.2"
    },
    "frontend-ui-ux-engineer": {
      "model": "anthropic/claude-sonnet-4-5-20241022"
    },
    "librarian": {
      "model": "opencode/gemini-3-pro"
    },
    "explore": {
      "model": "opencode/grok-code-fast-1"
    }
  }
}
```

#### 各エージェントのモデル設定

エージェントごとに異なるモデルを設定できます。用途に応じて最適なモデルを選びましょう。

| エージェント | 推奨モデル | 理由 |
|-------------|-----------|------|
| **Sisyphus** | Claude Opus 4.5 | 高度な推論と長いコンテキストが必要 |
| **oracle** | GPT 5.2 | 複雑なアーキテクチャ判断に強い |
| **frontend-ui-ux-engineer** | Claude Sonnet 4.5 | UIコード生成のバランスが良い |
| **librarian** | Gemini 3 Pro | 大量のドキュメント検索に適している |
| **explore** | Grok Code Fast 1 | 高速なコード検索に特化 |

#### プロバイダーの指定方法

モデルIDは`プロバイダー/モデル名`の形式で指定します。

```
anthropic/claude-opus-4-5-20251101  # Anthropic本家
openai/gpt-5.2                       # OpenAI本家
opencode/gemini-3-pro                # OpenCode Zen経由
opencode/grok-code-fast-1            # OpenCode Zen経由
```

サブスクリプション契約がある場合は本家プロバイダーを直接指定し、それ以外は**OpenCode Zen**を経由するのがコスト効率が良いです。

#### 設定可能なオプション

エージェントごとに以下のオプションを設定できます。

```json
{
  "agents": {
    "oracle": {
      "model": "openai/gpt-5.2",
      "temperature": 0.7,
      "disable": false
    }
  }
}
```

| オプション | 説明 | デフォルト |
|-----------|------|-----------|
| `model` | 使用するモデルID | エージェントにより異なる |
| `temperature` | 出力のランダム性（0.0〜2.0） | モデルにより異なる |
| `disable` | エージェントを無効化 | `false` |

#### フックとMCPの設定

oh-my-opencodeには20以上の組み込みフックがあり、ワークフローを自動化できます。また、MCPサーバーとの連携も設定可能です。

```json
{
  "disabled_hooks": ["some-hook-name"],
  "disabled_mcps": ["some-mcp-name"]
}
```

詳細な設定については[公式ドキュメント](https://ohmyopencode.com/documentation/)を参照してください。

---

## 使ってみての感想

### 既存プロジェクトに導入しやすかった

opencodeを起動してから、`/init`コマンドを打つと、既存プロジェクトのディレクトリ構成やpackage.json、README、プロダクトコードなどを読み取って、**プロジェクトの概要を自動でまとめた`AGENTS.md`を生成してくれる**。それを元にopencode.jsonに生成した`AGENTS.md`を設定するだけで、細かいチューニングなどは必要なもののあらかたプロジェクトのルールに沿って実装してくれて便利だった

### モデルの選択肢が豊富で柔軟に使える

OpenCodeのモデル選択は公式お手製のOpenCodeZen経由、API key経由、Ollama、ローカルLLMなど多様な指定方法ができるので、例えば「ちょっと試してみたい・軽い作業をさせたい」くらいならOllamaやOpenCodeZenの無料モデルで事足り、本格的に使うならClaude Opus 4.5やGPT 5.2でオーケストレーションさせて複雑なタスクをこなすこともできる。ユースケースに応じてコストと性能のバランスを調整できるのがありがたい。

個人的には、Sisyphus（メインオーケストレーター）にはClaude Opus 4.5、探索系のexplore/librarianにはGrok Code Fast 1やGemini 3 Proみたいな安めのモデルを割り当てて、コストを抑えつつ複雑なタスクにも対応できるようにしてる。

### 注意点

- **トークン消費は多め**: 複数エージェントが並列で動くので、explore/librarianには安いモデルを割り当てるのがおすすめ
- **設定ファイルの理解は必要**: 最初はちょっととっつきにくいけど、公式ドキュメントを見れば大丈夫

---

## 参考

- [OpenCode公式ドキュメント](https://opencode.ai/docs/)
- [OpenCode GitHub](https://github.com/anomalyco/opencode)
- [Oh My OpenCode公式サイト](https://ohmyopencode.com/)
- [Oh My OpenCode GitHub](https://github.com/code-yeongyu/oh-my-opencode)
- [Oh My OpenCode DeepWiki](https://deepwiki.com/code-yeongyu/oh-my-opencode)
- [OpenCode Zen](https://opencode.ai/zen)
